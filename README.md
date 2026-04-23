# glab-mr-review

Claude Code skills for GitLab Merge Request code review workflow.

## Skills

### `glab-mr-review` — Post Inline DiffNote Comments

Triggers: "review MR !N", "inline code review on GitLab MR", "add inline review comments", GitLab MR URL

Audits a GitLab MR and posts per-file per-line inline DiffNote comments on the Changes tab.

### `glab-mr-review-verify` — Verify Posted Comments

Triggers: "verify comments", "review the review", "check the inline comments", "audit the posted review"

Audits the quality, accuracy, and constructiveness of DiffNote comments already posted by `glab-mr-review`.

## Workflow

```
User → glab-mr-review → Posts inline comments on MR
                ↓
        glab-mr-review-verify → Audits the posted comments
```

## Installation

### Claude Code

```bash
mkdir -p ~/.claude/skills/glab-mr-review
cp -r skills/* ~/.claude/skills/glab-mr-review/
```

### Cursor IDE

1. Open Cursor Settings → Features → AI Models → Custom Instructions
2. Add the following to your global/custom instructions:

```
When you encounter a GitLab MR URL or "!N" MR notation, invoke the /glab-mr-review skill.
When asked to verify, audit, or review the inline comments already posted, invoke /glab-mr-review-verify.
```

Alternatively, copy `skills/` to `.cursor/rules/` or reference the skill descriptions in your custom instructions.

## Prerequisites

### `glab` CLI Setup

1. **Install glab**:

   ```bash
   # macOS
   brew install glab

   # Linux (Debian/Ubuntu)
   curl -L https://gitlab.com/gitlab-org/cli/-/releases/v1/releases/packages/$(uname -m)/glab_1/latest/download  | tar -xz && mv bin/glab /usr/local/bin/

   # Or use your package manager
   ```

2. **Authenticate with GitLab**:

   ```bash
   glab auth login --hostname gitlab.bxd-dev.ch
   ```

   - Choose **GitLab instance** (not GitHub)
   - Provide your GitLab URL (e.g., `https://gitlab.bxd-dev.ch`)
   - Generate a token at `https://gitlab.bxd-dev.ch/-/profile/personal_access_tokens` with `api` and `write_repository` scopes
   - Paste the token when prompted

3. **Verify auth**:

   ```bash
   glab auth status
   glab api user --jq '.username'
   ```

4. **Set default host** (optional — saves passing `--repo` on every call):

   ```bash
   glab config set gitlab gitlab.bxd-dev.ch
   ```

### `jq` (optional but recommended)

Many skill steps use `jq` for JSON parsing:

```bash
brew install jq        # macOS
sudo apt install jq    # Linux
```

## Requirements

- `glab` CLI authenticated (`glab auth login`)
- Claude Code or Cursor with skill system enabled

## Examples

```bash
/glab-mr-review https://gitlab.bxd-dev.ch/bxd-token-trading-platform/audit-log/-/merge_requests/96

/glab-mr-review-verify https://gitlab.bxd-dev.ch/bxd-token-trading-platform/audit-log/-/merge_requests/96
```