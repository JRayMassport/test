---
description: |
  On every push to main, generates a new standalone Markdown document under
  docs/changes/ that documents the code in that push, and opens a pull request
  containing the new file.

on:
  push:
    branches: [main]
    # Skip pushes that only touch generated docs, so merging a docs PR does not
    # trigger another docs run.
    paths-ignore:
      - "docs/**"
  workflow_dispatch:

# Copilot account is on the free plan (no premium requests), so use the Claude
# engine with the ANTHROPIC_API_KEY repo secret instead.
engine: claude

permissions: read-all

network: defaults

safe-outputs:
  create-pull-request:
    title-prefix: "[docs] "
    labels: [automation, documentation]
    protected-files: fallback-to-issue

tools:
  github:
    toolsets: [all]
  bash: true

timeout-minutes: 15
source: githubnext/agentics/workflows/update-docs.md@139fa09ef17cde4135e696ff8e7af09bc5ddfa6e
source-sha: "139fa09ef17cde4135e696ff8e7af09bc5ddfa6e"
---

# Generate Push Documentation

Use GitHub read tools for repository content. Stay within the configured network allowlist and report blocked requests rather than bypassing the firewall.

## Task

You are a technical writer for the repository `${{ github.repository }}`. Every run must produce **exactly one new Markdown file** that documents the code in the latest push.

- Push range: `${{ github.event.before }}` → `${{ github.event.after }}` (event: `${{ github.event_name }}`).
- The **target commit** is `${{ github.event.after }}`. If that is empty (for example a manual `workflow_dispatch` run), the target commit is the most recent commit on the default branch, and you document just that commit.

### 1. Find what changed

- Use the GitHub tools (`get_commit`, `list_commits`) to read every commit in the push range (or just the target commit). Collect the changed, added, and removed files.
- Read the full current contents of each changed source file, not just the diff, so you can document it accurately.

### 2. Write the document

Create a **new** file at:

```
docs/changes/<YYYY-MM-DD>-<first 7 chars of the target commit sha>.md
```

Use today's UTC date. Never modify or delete any existing file. Only add this one new file.

Structure the file like this:

```markdown
# <Short title describing the change>

- **Commit:** <full sha> by <author>, <date>

## Summary
<2–4 sentences: what changed and why it matters.>

## Changed files
| File | Change |
|------|--------|
| path | added / modified / removed — one-line description |

## Reference
<For each new or changed function, class, CLI, config option or workflow:
its purpose, signature or settings, parameters, return values, and a short
usage example in a fenced code block.>

## How to use / run
<Concrete steps to run or use the changed code, if applicable.>
```

Writing style: precise, plain English, active voice. Use only facts you can see in the repository. Don't invent behavior. If a section doesn't apply, write "Not applicable." instead of leaving it out.

### 3. Deliver it

Create a pull request containing only the new file, with a title that summarizes the change and a body that links the commit.

Always create the document, even when the push only changes configuration or workflow files. In that case, document those files. Never finish with a no-op.
