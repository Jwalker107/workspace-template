# AGENTS.md

This repository hosts multiple BigFix RepositorySites to create, organize, and update BigFix Content.  Relevant Skills are listed below.  Each Skill starts with a `SKILL.md` that explains when and how to use it.

## Skills

| Skill | Use it when |
| --- | --- |
| [bigfix-skill](.agents/skills/bigfix-skill/SKILL.md) | The task involves HCL BigFix in any form: Relevance (client or session), ActionScript, Fixlets, Tasks, Analyses, Baselines, `.bes` files, qna / Fixlet Debugger, client settings, or the BigFix REST API. Read its `SKILL.md` first, then the `reference/` file for the part you are working on, and validate any `.bes` output with `scripts/validate_bes.py` or `scripts/Validate-Bes.ps1` before handing it over. |

## Conventions

- `.bes` files are XML; save them UTF-8 with CRLF line endings (see `workspace.code-workspace`).
- Python tooling uses the repo-local `.venv` (Python 3.14; `pip install lxml` for the validator).
