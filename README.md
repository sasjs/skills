# SASjs Agent Skills

A collection of agent skills for the SASjs ecosystem and the SAS language.

## Installing

Install the whole collection:

```bash
npx skills add sasjs/skills
```

Install (or preview) an individual skill by appending `@<skill-name>`:

```bash
npx skills add sasjs/skills@sasjs-core      # install one skill
npx skills use sasjs/skills@sas             # generate a prompt for one skill without installing
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

## Conventions in this repo

- Each skill is a folder containing `SKILL.md` (YAML frontmatter with `name` and `description`).
- Markdown files are not hard-wrapped: one paragraph per line.
