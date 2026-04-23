---
name: glab-mr-review-verify
description: Verify that inline DiffNote comments posted by glab-mr-review are accurate, constructive, and follow best practices. Use this after glab-mr-review has posted comments on a GitLab MR — it audits the comments themselves, not the code. Trigger when the user asks to "verify comments", "review the review", "check the inline comments", or "audit the posted review".
---

# Review the Review: Verify Inline DiffNote Comments

## Input

A GitLab MR URL that has already received inline DiffNote comments from `glab-mr-review`.

## Workflow

### Step 1 — Fetch Existing Comments

Use GitLab API to retrieve all DiffNotes on the MR:

```bash
# Get token
TOKEN=$(glab config get --host gitlab.bxd-dev.ch token)

# Fetch diff notes (inline comments on changes)
curl -s --header "PRIVATE-TOKEN: $TOKEN" \
  "https://gitlab.bxd-dev.ch/api/v4/projects/<project-encoded>/merge_requests/<N>/discussions?fields=notes" | jq -r '.[] | select(.notes[].position != null) | .notes[] | "\n---\nfile:\(.position.new_path // .position.old_path)\nline:\(.position.new_line // .position.old_line)\nauthor:\(.author.username)\nbody:\(.body)"'
```

Also fetch the MR diff for context.

### Step 2 — Audit Each Comment

Launch a parallel Haiku agent to evaluate each comment against:

**Accuracy**: Does the issue actually exist in the code? Is the concern valid?

**Constructiveness**: Is the feedback actionable? Does it explain *why* it's a problem?

**Tone**: Is it professional and respectful? Could it be perceived as condescending?

**Completeness**: Are edge cases or related concerns missing?

**Duplication**: Is this comment already covered by another comment?

### Step 3 — Score Comments

Rate each comment 0-100:
- 0: Wrong or misleading
- 25: Technically correct but unhelpful
- 50: Useful but could be improved
- 75: Good comment, minor polish needed
- 100: Excellent — specific, actionable, well-explained

### Step 4 — Report

Summarize:
- Comments that need revision or deletion
- Comments that are excellent
- Any missed issues the reviewer should add
- Overall comment quality score (average of scores)

### Step 5 — Optional: Post Follow-up

If requested, post a reply comment acknowledging excellent comments or suggesting improvements to weak ones.