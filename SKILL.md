---
name: glab-mr-review
description: Perform a code review on a GitLab Merge Request and post inline DiffNote comments on the MR Changes tab — not a single summary comment, but separate per-file per-line comments anchored to the specific code lines being reviewed. Use this when the user asks for a code review on a GitLab MR, wants inline comments on specific lines, or says to "review MR !N", "inline code review on GitLab", or "add inline review comments". Do NOT use this for GitHub PRs. Do NOT use this for a single summary comment — the output must be per-file inline DiffNotes anchored to relevant lines. Trigger whenever the user provides a GitLab MR URL or references an MR number with "!N" notation.
---

# Code Review: Inline Comments on GitLab MR

## Input

A GitLab MR URL, e.g. `https://gitlab.bxd-dev.ch/bxd-token-trading-platform/audit-log/-/merge_requests/96`

## Workflow

### Step 1 — Eligibility Check

Use `glab mr view <N> --repo <project>` to check:
- MR state (skip if closed/merged)
- Draft status (skip if draft)
- Existing reviews from you (skip if already reviewed)
- Whether it's a trivial/automated PR needing no review

If not eligible, tell the user and stop.

### Step 2 — Fetch Diff

```bash
glab api projects/<project-path>/merge_requests/<N>/diffs | jq -r '.[] | "\(.new_path)\n\(.diff)"' > /tmp/mr_diff.md
glab api projects/<project-path>/merge_requests/<N>/versions | jq -r '.[0] | "base=\(.base_commit_sha) head=\(.head_commit_sha) start=\(.start_commit_sha)"'
```

Also fetch the MR summary:
```bash
glab mr view <N> --repo <project>
```

### Step 3 — Run 5 Parallel Review Agents

Launch 5 parallel subagents (model: sonnet), each returns a list of issues with reason flagged:

**Agent 1 — CLAUDE.md compliance**: Audit changes against CLAUDE.md files. Check root `.CLAUDE.md` and any `CLAUDE.md` in modified directories.

**Agent 2 — Shallow bug scan**: Read the diff only (no extra context). Focus on large bugs, avoid nitpicks. Ignore pre-existing issues and linter catches.

**Agent 3 — Git history context**: Run `git log --oneline -10 -- <file>` and `git blame <file> | head -50` for modified files. Identify bugs in light of historical decisions.

**Agent 4 — Previous PR comments**: Check git log for prior MR patterns. Look for recurring concerns that apply to current changes.

**Agent 5 — Code comment compliance**: Read modified files and check if changes violate inline guidance.

### Step 4 — Score Issues

For each issue from agents, launch a parallel Haiku agent to score confidence 0-100 using rubric:
- 0: False positive / pre-existing
- 25: Might be real, might be false positive
- 50: Real but nitpick
- 75: Highly confident real issue, important
- 100: Absolutely certain, frequent

### Step 5 — Filter

Keep only issues scoring ≥ 80.

### Step 6 — Re-check Eligibility

Verify MR still open before posting.

### Step 7 — Post Inline Comments

For each remaining issue, post a **DiffNote** (inline comment on the Changes tab) via GitLab API:

```bash
# Get token
TOKEN=$(glab config get --host gitlab.bxd-dev.ch token)

# POST inline comment
curl -s --request POST \
  --header "PRIVATE-TOKEN: $TOKEN" \
  --header "Content-Type: application/json" \
  -d "{
    \"body\": \"<comment text — keep brief, cite file and line>\",
    \"position\": {
      \"position_type\": \"file\",
      \"base_sha\": \"<base_sha>\",
      \"start_sha\": \"<start_sha>\",
      \"head_sha\": \"<head_sha>\",
      \"old_path\": \"<file>\",
      \"new_path\": \"<file>\",
      \"old_line\": <old_line>,
      \"new_line\": <new_line>
    }
  }" \
  "https://gitlab.bxd-dev.ch/api/v4/projects/<project-encoded>/merge_requests/<N>/discussions"
```

Use `position_type: "file"` (not "text" or "revision"). Get SHA values from the versions API call in step 2.

**Line selection**: Anchor comment to the first line of the relevant change block. Use `old_line` / `new_line` to point to the line in the diff view.

**Comment format**: Be specific. Include file path, line number, and the concern. Example:
```
**values.staging.yaml:64** — `replicaCount: 1` with `autoScaling: false` removes staging's ability to scale under load during integration tests. Consider keeping HPA with `minReplicas: 1, maxReplicas: 2`.
```

**How many comments**: Post 1 per meaningful issue. 3-8 total comments is typical. If no issues qualify (score < 80), post a single note saying "No blocking issues found."

### Step 8 — Confirm

Tell the user how many inline comments were posted and link to the MR.