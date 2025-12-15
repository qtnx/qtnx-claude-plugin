---
name: gh-pr-review
description: This skill provides guidance for using the gh-pr-review GitHub CLI extension to view and manage inline PR review comments from the terminal. Use this skill when working with pull request reviews, viewing inline code review threads, replying to comments, starting/submitting reviews, resolving threads, or when needing structured JSON output for PR review workflows. Triggers on requests like "show PR review comments", "reply to this thread", "start a review", "resolve threads", "view unresolved comments", or any PR code review task.
---

# GitHub PR Review CLI Extension

## Overview

The `gh-pr-review` extension brings **inline PR review comments** to the terminal. GitHub's built-in `gh` tool does not show inline comments or review threads - this extension fills that gap.

## Prerequisites

Ensure the extension is installed:

```bash
gh extension install agynio/gh-pr-review
# Update existing installation
gh extension upgrade agynio/gh-pr-review
```

## Core Capabilities

### 1. View Review Threads

To view all reviews and inline comments for a PR:

```bash
gh pr-review review view -R owner/repo --pr <number>
```

Common filters:
- `--reviewer <login>` - Filter by reviewer
- `--states APPROVED,CHANGES_REQUESTED,COMMENTED,DISMISSED` - Filter by state
- `--unresolved` - Show only unresolved threads
- `--not_outdated` - Exclude outdated threads
- `--tail <n>` - Keep only last n replies per thread
- `--include-comment-node-id` - Include GraphQL comment IDs

Example - View unresolved threads from a specific reviewer:
```bash
gh pr-review review view -R owner/repo --pr 42 --reviewer alice --unresolved
```

### 2. Start a Pending Review

To start a new pending review and get the review ID:

```bash
gh pr-review review --start -R owner/repo <pr-number>
```

Output includes `"id": "PRR_..."` - save this for adding comments.

### 3. Add Inline Comments

To add comments to a pending review (requires `PRR_...` review ID):

```bash
gh pr-review review --add-comment \
  --review-id PRR_kwDOAAABbcdEFG12 \
  --path path/to/file.go \
  --line 42 \
  --body "Your comment here" \
  -R owner/repo <pr-number>
```

### 4. Reply to Thread

To reply to an existing review thread:

```bash
gh pr-review comments reply \
  --thread-id PRRT_kwDOAAABbcdEFG12 \
  --body "Your reply" \
  -R owner/repo <pr-number>
```

To reply from a pending review, add `--review-id`:
```bash
gh pr-review comments reply \
  --thread-id PRRT_kwDOAAABbcdEFG12 \
  --review-id PRR_kwDOAAABbcdEFG12 \
  --body "Reply from pending review" \
  -R owner/repo <pr-number>
```

### 5. Submit Review

To finalize and submit a pending review:

```bash
gh pr-review review --submit \
  --review-id PRR_kwDOAAABbcdEFG12 \
  --event APPROVE|COMMENT|REQUEST_CHANGES \
  --body "Review summary" \
  -R owner/repo <pr-number>
```

### 6. Manage Threads

List threads:
```bash
gh pr-review threads list -R owner/repo <pr-number>
gh pr-review threads list --unresolved --mine -R owner/repo <pr-number>
```

Resolve/unresolve threads:
```bash
gh pr-review threads resolve --thread-id PRRT_... -R owner/repo <pr-number>
gh pr-review threads unresolve --thread-id PRRT_... -R owner/repo <pr-number>
```

## Workflow: Complete Review Cycle

1. **Start review**: `gh pr-review review --start ...` -> save `PRR_...` ID
2. **Add comments**: `gh pr-review review --add-comment --review-id PRR_... ...`
3. **View current state**: `gh pr-review review view ...`
4. **Submit review**: `gh pr-review review --submit --review-id PRR_... --event REQUEST_CHANGES ...`
5. **After fixes, resolve threads**: `gh pr-review threads resolve --thread-id PRRT_... ...`

## ID Types

| ID Prefix | Type | Usage |
|-----------|------|-------|
| `PRR_...` | Review ID | For `--review-id` in add-comment, submit, reply |
| `PRRT_...` | Thread ID | For `--thread-id` in reply, resolve, unresolve |
| `PRRC_...` | Comment ID | Returned with `--include-comment-node-id` |

## JSON Output Structure

All commands output structured JSON. For detailed schemas, see `references/schemas.md`.

## Resources

### references/

- `commands.md` - Complete command reference with all flags and examples
- `schemas.md` - JSON output schemas for each command
- `agents.md` - Agent-focused workflows and best practices
