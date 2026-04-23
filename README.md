# glab-mr-review

Claude Code skill for performing code review on GitLab Merge Requests with inline DiffNote comments.

## What it does

When invoked, this skill:
1. Checks MR eligibility (open, not draft, not already reviewed)
2. Fetches the diff and MR metadata via `glab` CLI
3. Runs 5 parallel AI agents to audit: CLAUDE.md compliance, bug scan, git history, previous PR patterns, code comment compliance
4. Scores each issue (0-100), filters to confidence ≥ 80
5. Posts per-file per-line inline comments (DiffNotes) on the MR Changes tab
6. Reports summary with link to MR

## Installation

Place `SKILL.md` in your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/glab-mr-review
cp SKILL.md ~/.claude/skills/glab-mr-review/
```

Then invoke with:

```
/glab-mr-review <GitLab-MR-URL>
```

Example:
```
/glab-mr-review https://gitlab.bxd-dev.ch/bxd-token-trading-platform/audit-log/-/merge_requests/96
```

## Requirements

- `glab` CLI authenticated (`glab auth login`)
- Claude Code with skill system enabled

## Skill name in Claude Code

`/glab-mr-review` — triggers on phrases like "review MR !N", "inline code review on GitLab MR", "add inline comments to MR"