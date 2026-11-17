---
name: sasjs-release
description: Cutting releases with @sasjs/release - conventional-commit classification, the tag as the version, asset attach and verification, the draft lifecycle (a GitHub draft creates no tag until it is published), what --prepare-publish decides, and the flow a repo whose artefacts are built AFTER the cut needs. Use when replacing semantic-release, wiring a release workflow, recovering an interrupted release, or changing the tool itself.
license: MIT
copyright: Copyright (c) SASjs
spdx-license-identifier: MIT
---

# @sasjs/release

<!-- SPDX-License-Identifier: MIT -->

`@sasjs/release` is the org's release tool: one file (`release.mjs`), zero dependencies, `bin: sasjs-release`. It classifies conventional commits, computes the version, writes the notes, creates the tag and the host release, and attaches assets. It does **not** publish to npm - that stays a separate step, in the workflow file the package's trusted publisher is registered against.

It replaced `semantic-release` for two reasons: semantic-release depends on `npm` as a package, and npm's bundled tree puts advisories in the dev dependencies that no `override` can clear; and its GitHub plugin uploads the assets it finds while saying nothing about the ones it does not, which is how a release was published with no files on it while every check stayed green.

This skill covers the tool and the pipeline around it. The cross-repo release order (library, then publish, then consumer pin) belongs to the org's release process, and the CLI's own workflows are in `sasjs-cli`.

Run `npx sasjs-release --help` for the flags available; the tool gains modes between releases.

## The model

- **The version is the tag.** Nothing writes a version into `package.json` in the repository; `--prepare-publish` stamps it into the working tree for the publish, and the tree is thrown away.
- **The notes are the commits the tag covers.** There is no `CHANGELOG.md` writer; the release page is the changelog.
- **Host-neutral.** GitHub, Gitea and GitLab over plain `https`. `--host none` does the git work only.
- **Idempotent.** `--tag`, `--push-tag` and `--release` can be re-run: an existing tag at HEAD is reused, and an existing release is found rather than duplicated.

## Modes

| Flag | What it does |
|---|---|
| `--dry-run` | classify and print; tags nothing, sends nothing |
| `--tag` | create the annotated tag (reused when it already names HEAD) |
| `--push-tag` | push the tag to `origin` |
| `--release` | create the release on the host, which also binds it to `target_commitish` - so the job needs no git credentials |
| `--draft` | create it as a draft (see below) |
| `--no-assets` | cut a release whose files are not on disk yet; requires `--draft` |
| `--asset <path>`, `--config <path>` | files to attach, or a config naming them (`release.assets` in `sasjs/sasjsconfig.json`, else a `.releaserc`) |
| `--attach <tag>` | upload the configured assets to a release that exists: no commits read, nothing tagged |
| `--verify-assets` | with `--attach`, require each asset's GitHub sha256 digest and size to match the bytes sent, and fail closed |
| `--keep-draft` | leave the release a draft after attaching, instead of publishing it |
| `--prepare-publish` | decide whether to publish, from the tag and the registry; creates nothing |
| `--prerelease <id>`, `--level <l>`, `--since <ref>`, `--paths <a,b>` | prerelease numbering, forced bump, commit range, monorepo path filter |

Every mode writes `key=value` lines on stdout and, with `--output <path>`, appends them for a workflow to consume: `released`, `version`, `tag`, `level` for a cut; `skip`, `tag`, `version`, `publish`, `exists` for `--prepare-publish`.

## The draft lifecycle, and why cutting first is safe

On **GitHub**, a draft release creates **no tag**: the backing tag is created from `target_commitish` when the draft is published. Measured, because it is easy to assume otherwise - create a draft naming a tag that does not exist, then `GET /repos/{o}/{r}/git/ref/tags/{tag}` answers 404.

That is what makes "cut first, build second" work, which is the shape a repository with platform executables needs - the artefacts have to name the release they belong to, and the assets need a release to attach to:

1. `--release --no-assets --draft` - tag bound to the commit, release invisible, nothing published.
2. build the artefacts.
3. `--attach "$TAG" --verify-assets --keep-draft` - uploaded, read back, digests checked, still invisible.
4. `--prepare-publish --tag "$TAG" --branch main` - is this version still missing from the registry?
5. `npm publish` when it says so.
6. `--attach "$TAG" --verify-assets` - verify again and publish the draft.

A failure anywhere after step 1 burns no version: no tag exists, so the next run re-classifies the same commits, computes the same version and finishes the release. Gitea and GitLab create the tag with the release, so a draft there already names one.

**Reuse of an existing release is decided by the commit it names.** A release that already exists for the tag is reused only when its `target_commitish` is the commit being released. That matters because of the draft lifecycle above: a run at a *later* commit classifies to the same version and finds the draft cut at an earlier one. So a stale draft is discarded and the release cut again at this commit, and a published release naming another commit is refused before any upload - a release whose files and tag disagree is the one outcome the tool will not produce.

The comparison is a sha comparison, not a resolution through the clone: a release job's checkout can be shallow, and the commit a stale draft names is exactly the object such a checkout lacks. A `target_commitish` that is not a sha - the branch name a release made in the web UI stores - cannot be compared with what the release held, so it is reused as it always was.

## --prepare-publish is the publish decision

Never gate a publish on `released` from the cut. A run that dies after the tag exists and before npm has the version reports the same `released` as one that finished, so the next run - which has nothing to release - would skip publishing for ever.

`--prepare-publish` answers from state: `skip=true` when no tag names this commit, `publish=true` when the registry does not serve the version, `exists=true` when it does, and `tag`/`version` either way. It also refuses with a named error when the tag is not on `--branch`, and when a shallow clone cannot see the tags at all.

Check its answer in the workflow. A step that exits 0 without writing `skip=` leaves every later gate reading an empty variable, and the run is green having released nothing - assert the decision is one the workflow understands, or the guard is a comment.

## Assets

`--attach` uploads the configured list, reads the release back, and publishes the draft only once it holds them. `--verify-assets` goes further, for GitHub: every asset must be `state: uploaded`, its size must match, and its `sha256:` digest must equal the bytes sent. It fails closed on a missing or unsupported digest, re-reads the release by id before the first upload and after the final check, and refuses an upload URL that does not belong to that repository and release.

The asset names come from `release.assets` in `sasjs/sasjsconfig.json`, or from a `.releaserc` a repository still carries - the migration from semantic-release is a string rename inside that file (`@semantic-release/github` to `@sasjs/release`), so the list moves without being retyped.

## Migrating a repo off semantic-release

- Remove `semantic-release` and its plugins from `devDependencies`; add `@sasjs/release` at an exact pin.
- `"release": "sasjs-release"` in `scripts`, so the tool comes from the lockfile and the bytes are pinned on the job that can publish.
- Never gate the publish on `released`; use `--prepare-publish` (above).
- Keep `npm publish` in the workflow file the trusted publisher is registered against - npm validates the workflow that runs the publish (the OIDC `job_workflow_ref` claim), not the one that triggered it.
- Run the test suite before the cut if a red suite should block a release; a merge-triggered release job usually runs in parallel with the PR pipeline, so nothing stops it for you.
- A `chore:` migration cuts no release, so the merge itself proves nothing: the first `fix:`/`feat:` after it is the first real run, and the run log is where `released=`, `skip=`, `publish=` and the tool's own lines get read.

## Changing the tool

`release.mjs` is one file with a real suite: `npm test` (node's test runner), a coverage gate (`test:coverage`, lines 97 / branches 69 / functions 99), `test:mutate` (mutation testing) and `test:live` (a live contract probe). Run them before pushing.

**The mutation harness targets exact source lines, and a refactor is what breaks it.** Each entry in `scripts/mutate.mjs` is a `find`/`replace` pair over the source; rename or re-indent a line an entry matches and the pattern matches nothing, so the mutation is reported as a *survivor* - the gate is telling you the entry went stale, not that a test is missing. Update the entry in the same commit, and add an entry for the behaviour you just introduced, or the next person reads a red mutation job as a coverage failure. This happened twice in one change: a variable renamed out from under an entry, then the guard's own signature changed.

The same rule applies to any harness that pins code by its text - a workflow-shape test that greps its own YAML needs multi-line `run:` bodies extracted before it asserts, because a per-line regex reads only the first line of a `run: |` block and everything else in the pipeline is written that way.
