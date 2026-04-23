# glab-mr-review

Claude Code skills for GitLab Merge Request code review workflow.

> Post per-file, per-line inline DiffNote comments on MRs using AI-powered review agents.

---

## Table of Contents

- [Skills](#skills)
  - [`glab-mr-review` — Post Inline DiffNote Comments](#glab-mr-review--post-inline-diffnote-comments)
  - [`glab-mr-review-verify` — Verify Posted Comments](#glab-mr-review-verify--verify-posted-comments)
- [Workflow](#workflow)
- [Installation](#installation)
  - [Claude Code](#claude-code)
  - [Cursor IDE](#cursor-ide)
- [Prerequisites](#prerequisites)
  - [`glab` CLI Setup](#glab-cli-setup)
  - [`jq` (optional but recommended)](#jq-optional-but-recommended)
- [Examples](#examples)

---

## Skills

### `glab-mr-review` — Post Inline DiffNote Comments

**Triggers:** `review MR !N` · `inline code review on GitLab MR` · `add inline review comments` · GitLab MR URL

Audits a GitLab MR and posts **per-file per-line inline DiffNote comments** on the MR Changes tab — not a single summary comment.

### `glab-mr-review-verify` — Verify Posted Comments

**Triggers:** `verify comments` · `review the review` · `check the inline comments` · `audit the posted review`

Audits the quality, accuracy, and constructiveness of DiffNote comments already posted by `glab-mr-review`.

---

## Workflow

```
User → glab-mr-review → Posts inline comments on MR
                ↓
        glab-mr-review-verify → Audits the posted comments
```

1. **Eligibility check** — skip closed/merged/draft MRs
2. **Fetch diff** — via `glab api`
3. **Run 5 parallel agents** — CLAUDE.md compliance, bug scan, git history, PR patterns, comment compliance
4. **Score & filter** — keep issues ≥ 80 confidence only
5. **Post DiffNotes** — per-file inline comments anchored to specific lines
6. **Verify** — optionally audit the posted comments

---

## Installation

### Claude Code

```bash
mkdir -p ~/.claude/skills/glab-mr-review
mkdir -p ~/.claude/skills/glab-mr-review-verify

# Main review skill
cp SKILL.md ~/.claude/skills/glab-mr-review/

# Verify skill
cp glab-mr-review-verify/SKILL.md ~/.claude/skills/glab-mr-review-verify/
```

Or use symlinks (recommended — keeps skills in sync with repo):

```bash
ln -s /Users/arko/projects/glab-mr-review ~/.claude/skills/glab-mr-review
ln -s /Users/arko/projects/glab-mr-review/glab-mr-review-verify ~/.claude/skills/glab-mr-review-verify
```

> Claude Code requires skills to be a directory containing a `SKILL.md` file. A standalone `.md` file will not appear in the `/` menu.

### Cursor IDE

1. Open **Cursor Settings → Features → AI Models → Custom Instructions**
2. Add to your global/custom instructions:

   ```
   When you encounter a GitLab MR URL or "!N" MR notation, invoke the /glab-mr-review skill.
   When asked to verify, audit, or review the inline comments already posted, invoke /glab-mr-review-verify.
   ```

3. Alternatively, copy skill `.md` files to `.cursor/rules/` directory

---

## Prerequisites

### `glab` CLI Setup

1. **Install `glab`**:

   ```bash
   # macOS
   brew install glab

   # Linux (Debian/Ubuntu)
   curl -L https://gitlab.com/gitlab-org/cli/-/releases/v1/releases/packages/$(uname -m)/glab_1/latest/download \
     | tar -xz && mv bin/glab /usr/local/bin/
   ```

2. **Authenticate with GitLab**:

   ```bash
   glab auth login --hostname gitlab.bxd-dev.ch
   ```

   - Choose **GitLab instance** (not GitHub)
   - Generate a token at `https://gitlab.bxd-dev.ch/-/profile/personal_access_tokens`
     - Required scopes: `api` + `write_repository`
   - Paste the token when prompted

3. **Verify auth**:

   ```bash
   glab auth status
   glab api user --jq '.username'
   ```

4. **Set default host** (optional):

   ```bash
   glab config set gitlab gitlab.bxd-dev.ch
   ```

### `jq` (optional but recommended)

```bash
brew install jq        # macOS
sudo apt install jq    # Linux
```

---

## Examples

```bash
# Post inline review on a GitLab MR
/glab-mr-review https://gitlab.bxd-dev.ch/bxd-token-trading-platform/audit-log/-/merge_requests/96

# Verify the posted comments
/glab-mr-review-verify https://gitlab.bxd-dev.ch/bxd-token-trading-platform/audit-log/-/merge_requests/96
```

**In Cursor:** Simply mention a GitLab MR URL in conversation — the custom instruction will invoke the skill automatically.

---

## Requirements

| Requirement | Details |
|-------------|---------|
| `glab` CLI | Authenticated (`glab auth login`) |
| Claude Code / Cursor | Skill system enabled |