# SASjs Agent Skills

A collection of agent skills for the SASjs ecosystem and the SAS language.

## Installing

Install the whole collection:

```bash
npx skills add sasjs/skills
```

Install (or preview) an individual skill by appending `@<skill-name>`:

```bash
npx skills add sasjs/skills@sasjs-core # install one skill
npx skills use sasjs/skills@sas        # generate a prompt for one skill w/o installing
```

Available skill names: `sas`, `sasjs-core`, `sasjs-cli`, `sasjs-adapter`, `sasjs-framework`, `sasjs-server`.

Other useful commands:

```bash
npx skills list                  # list installed skills
npx skills update                # update to latest versions
npx skills remove sasjs-core     # remove a skill
```

| Skill | Description |
|---|---|
| `sas` | Pure SAS language expertise — DATA step, PROC SQL, macro language, formats, ODS. No SASjs content. |
| `sasjs-core` | Standards and conventions for the @sasjs/core SAS macro library (doxygen headers, lint rules, macro prefixes, reuse catalog). |
| `sasjs-cli` | Using @sasjs/cli: create/compile/build/deploy/run/test workflows, targets, auth, service packs, frontend streaming. |
| `sasjs-adapter` | Frontend/Node integration with SAS backends via @sasjs/adapter (requests, auth per server type, sessions, file upload). |
| `sasjs-framework` | Building full SASjs applications — project layout, sasjsconfig.json (schema included), service contract, multi-target discipline, quality gates. |
| `sasjs-server` | Installing/configuring/running @sasjs/server — desktop vs server modes, runtimes (SAS/JS/Python/R), env vars, LDAP auth, mock server types, REST API. |

## Using with GitHub Copilot in VS Code

Skills are agent-agnostic — each is just a folder with a `SKILL.md` (name, description, instructions). VS Code Copilot auto-discovers skills in `.claude/skills/` (and `.github/skills/`) in your workspace, and loads them into whichever model is handling the turn (including Auto mode) when your prompt matches a skill's description.

To set up:

1. Run `npx skills add sasjs/skills` from your project root, or simply copy the skill folders you want into `.claude/skills/` in your repo.
2. Commit the folder so your whole team gets the skills automatically.
3. No further wiring needed. When a skill is relevant, Copilot reads its instructions before answering (e.g. DATA step questions trigger `sas`; `sasjs build` questions trigger `sasjs-cli`).

Tip for Auto mode: since the model isn't guaranteed, it can help to name the skill explicitly when you know what you want — "using the sasjs-cli skill, deploy this to my Viya target" works regardless of which underlying model is selected.

## Conventions in this repo

- Each skill is a folder containing `SKILL.md` (YAML frontmatter with `name` and `description`).
- Markdown files are not hard-wrapped: one paragraph per line.
