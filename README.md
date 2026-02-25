# bolt-sprint

An [Agent Skill](https://agentskills.io) for managing sprints and stories in [Bolt](https://github.com/ndhill84/Bolt) — a collaborative software development platform built for human-AI teamwork.

## What This Skill Does

Once installed, any AI agent that supports the agentskills.io standard can:

- List and navigate projects and sprints
- Create, update, and delete stories
- Move stories through the Kanban workflow (`waiting` → `in_progress` → `completed`)
- Track blockers and inter-story dependencies
- Add notes and labels to stories
- Get sprint digests (story counts, blocked list, assignee breakdown)
- Log AI activity as agent session events (visible in the Bolt UI)
- Upload and retrieve files attached to stories
- Poll for changes efficiently with delta sync
- Run batch operations (move or patch many stories at once)

## Installation

Install to your local skills directory:

```bash
# Claude Code
git clone https://github.com/ndhill84/Bolt-skill ~/.claude/skills/bolt-sprint

# Codex CLI
git clone https://github.com/ndhill84/Bolt-skill ~/.codex/skills/bolt-sprint
```

Or via `npx skills` (if published):

```bash
npx skills install bolt-sprint
```

## Configuration

Set environment variables before starting your AI session:

```bash
export BOLT_BASE_URL="http://localhost:3000"  # Required: URL of your Bolt instance
export BOLT_API_TOKEN="your-token"            # Optional: required if server uses BOLT_API_TOKEN
```

Check connectivity:

```bash
curl -s "$BOLT_BASE_URL/health"
# → {"ok":true}
```

## Using the Helper Script

`scripts/bolt.sh` wraps the most common operations:

```bash
# Make it available on your PATH
export PATH="$PATH:$HOME/.claude/skills/bolt-sprint/scripts"

bolt.sh health
bolt.sh projects
bolt.sh sprints <projectId>
bolt.sh stories "sprintId=<id>&status=in_progress"
bolt.sh digest sprint <sprintId>
bolt.sh digest daily <projectId>
bolt.sh move <storyId> in_progress
bolt.sh move <storyId> completed
bolt.sh note <storyId> "Implementation complete. PR #42 opened."
bolt.sh log-event <sessionId> "Analyzing codebase" action
bolt.sh audit [projectId]
```

## File Structure

```
bolt-sprint/
├── SKILL.md                    # Skill entry point (agentskills.io spec)
├── references/
│   ├── api-reference.md        # Full endpoint docs with request/response schemas
│   └── workflows.md            # Workflow recipes for common scenarios
└── scripts/
    └── bolt.sh                 # Bash CLI helper
```

## API Coverage

| Resource | Endpoints |
|----------|-----------|
| Health | `GET /health`, `GET /ready` |
| Projects | List, create, update, delete |
| Sprints | List, create, update, start, close, assign stories |
| Stories | List (filtered), create, update, delete, move, batch move, batch patch |
| Notes | List, create |
| Labels | List, add, remove |
| Dependencies | List, add, remove |
| Files | List, create, upload (binary), delete, get content |
| Agent Sessions | List, log events |
| Audit | Changefeed |
| Digests | Sprint summary, daily project snapshot |

Key API behaviors:

- **Idempotency** — include `Idempotency-Key` on writes for safe retries
- **Pagination** — cursor-based, default limit 50, max 200
- **Field projection** — `?fields=id,title,status` to fetch only what you need
- **Delta sync** — `?updated_since=<ISO8601>` to poll for changes efficiently
- **Rate limits** — 120 writes/minute per IP

## Workflow Examples

Full recipes are in [`references/workflows.md`](references/workflows.md). Quick examples:

**Sprint standup:**
```bash
bolt.sh digest sprint $SPRINT_ID
bolt.sh stories "sprintId=$SPRINT_ID&blocked=true&fields=id,title,assignee"
```

**Pick up and complete a story:**
```bash
bolt.sh move $STORY_ID in_progress
# ... do the work ...
bolt.sh note $STORY_ID "Implemented. PR #42 opened."
bolt.sh move $STORY_ID completed
```

**Mark a blocker:**
```bash
bolt.sh patch $STORY_ID '{"blocked":true}'
bolt.sh note $STORY_ID "Blocked: waiting for DB migration approval."
```

## Spec Compliance

This skill conforms to the [agentskills.io specification](https://agentskills.io/specification):

- `SKILL.md` at the repo root with valid YAML frontmatter (`name`, `description`)
- Name matches directory: `bolt-sprint`
- Instructions use the progressive disclosure model (metadata → instructions → references)
- `allowed-tools: Bash` declared in frontmatter
- Supporting files organized under `references/` and `scripts/`

## License

MIT — see `SKILL.md` frontmatter.
