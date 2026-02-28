---
name: ai-summary-comment
description: Posts or updates automated progress comments on GitHub PRs/Issues with expandable sections. Use after completing any fix-issue phase (pre-flight, test, gate, fix). Triggers on 'post comment to PR', 'update PR progress', 'post summary comment'.
metadata:
  author: dotnet-aspnetcore
  version: "3.0"
compatibility: Requires GitHub CLI (gh) authenticated with access to dotnet/aspnetcore repository.
---

# AI Summary Comment Skill

This skill posts automated progress comments to GitHub Pull Requests and Issues during the fix-issue workflow. Comments use **nested expandable `<details>` sections**, providing rich context to maintainers and contributors.

**⚠️ Self-Contained Rule**: All content in PR/Issue comments must be self-contained. Never reference local files like `CustomAgentLogsTmp/` — GitHub users cannot access your local filesystem.

**✨ Key Features**:
- **Single Unified Comment**: ONE comment per PR/Issue containing ALL sections
- **Nested Expandable Sections**: Top-level "Expand Full Review" with nested phase sections
- **Duplicate Prevention**: Finds existing `<!-- AI Summary -->` comment and updates it
- **DryRun Support**: Use `DRY_RUN=1` to preview changes locally before posting
- **Auto-Loading State Files**: Automatically finds and loads state files from `CustomAgentLogsTmp/PRState/`

## Comment Format

The AI Summary comment uses nested expandable sections matching this visual layout:

```
🤖 AI Summary

▼ 📊 Expand Full Review
  ────────────────────────────────────
  ► 🔍 Pre-Flight — Context & Validation
  ────────────────────────────────────
  ► 🧪 Test — Bug Reproduction
  ────────────────────────────────────
  ► 🚦 Gate — Test Verification & Regression
  ────────────────────────────────────
  ► 🔧 Fix — Analysis & Comparison
```

Each section is a collapsible `<details>` block with descriptive titles.

## Usage

```bash
# Auto-loads all phases from CustomAgentLogsTmp/PRState/ISSUE-{number}/PRAgent/
bash .github/skills/ai-summary-comment/scripts/post-ai-summary-comment.sh 21384 65567

# Dry run
DRY_RUN=1 bash .github/skills/ai-summary-comment/scripts/post-ai-summary-comment.sh 21384 65567
```

## Expected Directory Structure

Scripts auto-load from the fix-issue skill's output directory:

```
CustomAgentLogsTmp/PRState/ISSUE-{IssueNumber}/PRAgent/
├── pre-flight/
│   └── content.md
├── test/
│   └── content.md
├── gate/
│   └── content.md
├── try-fix/
│   ├── content.md              # Summary with comparison table
│   └── attempt-{N}/
│       ├── approach.md         # What was tried
│       ├── result.txt          # PASS / FAIL
│       ├── fix.diff            # git diff of changes
│       └── analysis.md         # Why it worked/failed (optional)
```

## Integration with fix-issue Skill

After the fix phase completes in the fix-issue workflow, invoke this skill:

```
Invoke skill: ai-summary-comment

Post the AI summary comment on PR #YYYYY for issue #XXXXX.
```

The skill will:
1. Read all phase directories from `CustomAgentLogsTmp/PRState/ISSUE-{N}/PRAgent/`
2. Build nested expandable `<details>` sections for each phase
3. Post (or update) a unified `<!-- AI Summary -->` comment on the PR

## Technical Details

- Comments identified by HTML marker `<!-- AI Summary -->`
- Existing comments are updated (not duplicated) when posting again
- Uses `gh api` for create/update operations
- Repository: `dotnet/aspnetcore`
