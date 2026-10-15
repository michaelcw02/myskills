# myskills

A personal suite of [Claude](https://claude.ai) skills I use day to day.

Each skill lives in its own folder under `skills/` with a `SKILL.md` (YAML frontmatter + instructions), the standard Claude Agent Skills format.

## Skills

| Skill | Command | What it does |
|---|---|---|
| [peg](skills/peg/SKILL.md) | `/peg TICKER` | One-company deep dive: PEG 1Y/2Y/3Y with rationale, what the company does and what it has, scorecard, bull/bear, catalysts, and portfolio fit (incl. covered-call / cash-secured-put angle). |

## Using a skill

- **Claude.ai / desktop:** Settings → Capabilities → Skills → upload the skill folder (zip `skills/<name>/`).
- **Claude Code:** copy the folder to `~/.claude/skills/<name>/` (personal) or `.claude/skills/<name>/` (per project).

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with `name` and `description` frontmatter.
2. Add a row to the table above.

> Not financial advice. The investing skills reflect my own rules and preferences.
