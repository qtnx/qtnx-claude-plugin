# gh-pr-review Command Reference

Complete command reference for gh-pr-review v1.6.0+.

## Global Flags

All commands support:
- `-R, --repo owner/repo` - Target repository
- `--pr <number>` or positional `<pr-number>` - Pull request number

## review Commands

### review --start

Start a new pending review.

```bash
gh pr-review review --start -R owner/repo <pr-number>
```

**Output:**
```json
{
  "id": "PRR_kwDOAAABbcdEFG12",
  "state": "PENDING"
}
```

### review --add-comment

Add inline comment to pending review.

```bash
gh pr-review review --add-comment \
  --review-id PRR_... \
  --path <file-path> \
  --line <line-number> \
  --body "Comment text" \
  -R owner/repo <pr-number>
```

**Flags:**
| Flag | Required | Description |
|------|----------|-------------|
| `--review-id` | Yes | GraphQL review ID (`PRR_...`) |
| `--path` | Yes | File path relative to repo root |
| `--line` | Yes | Line number in the diff |
| `--body` | Yes | Comment body text |

**Output:**
```json
{
  "id": "PRRT_kwDOAAABbcdEFG12",
  "path": "internal/service.go",
  "is_outdated": false,
  "line": 42
}
```

### review view

View reviews with inline comments and threads.

```bash
gh pr-review review view -R owner/repo --pr <number> [flags]
```

**Flags:**
| Flag | Description |
|------|-------------|
| `--reviewer <login>` | Filter by reviewer username (case-insensitive) |
| `--states <list>` | Comma-separated: APPROVED, CHANGES_REQUESTED, COMMENTED, DISMISSED |
| `--unresolved` | Show only unresolved threads |
| `--not_outdated` | Exclude outdated threads |
| `--tail <n>` | Keep only last n replies per thread (0 = all) |
| `--include-comment-node-id` | Include GraphQL comment IDs |

**Examples:**
```bash
# All reviews
gh pr-review review view -R owner/repo --pr 42

# Unresolved only
gh pr-review review view -R owner/repo --pr 42 --unresolved

# Specific reviewer with state filter
gh pr-review review view -R owner/repo --pr 42 --reviewer alice --states CHANGES_REQUESTED

# Latest reply per thread with comment IDs
gh pr-review review view -R owner/repo --pr 42 --tail 1 --include-comment-node-id
```

### review --submit

Submit a pending review.

```bash
gh pr-review review --submit \
  --review-id PRR_... \
  --event <event> \
  --body "Review summary" \
  -R owner/repo <pr-number>
```

**Flags:**
| Flag | Required | Description |
|------|----------|-------------|
| `--review-id` | Yes | GraphQL review ID (`PRR_...`) |
| `--event` | Yes | APPROVE, COMMENT, or REQUEST_CHANGES |
| `--body` | No | Review body/summary |

**Output (success):**
```json
{
  "status": "Review submitted successfully"
}
```

**Output (error):**
```json
{
  "status": "Review submission failed",
  "errors": [
    { "message": "mutation failed", "path": ["mutation", "submitPullRequestReview"] }
  ]
}
```

## comments Commands

### comments reply

Reply to an existing review thread.

```bash
gh pr-review comments reply \
  --thread-id PRRT_... \
  --body "Reply text" \
  -R owner/repo <pr-number>
```

**Flags:**
| Flag | Required | Description |
|------|----------|-------------|
| `--thread-id` | Yes | GraphQL thread ID (`PRRT_...`) |
| `--body` | Yes | Reply body text |
| `--review-id` | No | Include when replying from a pending review |

**Example with pending review:**
```bash
gh pr-review comments reply \
  --thread-id PRRT_kwDOAAABbcdEFG12 \
  --review-id PRR_kwDOAAABbcdEFG12 \
  --body "Addressed in latest commit" \
  -R owner/repo 42
```

## threads Commands

### threads list

List review threads.

```bash
gh pr-review threads list -R owner/repo <pr-number> [flags]
```

**Flags:**
| Flag | Description |
|------|-------------|
| `--unresolved` | Show only unresolved threads |
| `--mine` | Show only threads authored by current user |

**Output:**
```json
[
  {
    "threadId": "PRRT_kwDOABC123",
    "isResolved": false,
    "path": "internal/service.go",
    "line": 42,
    "isOutdated": false
  }
]
```

### threads resolve

Mark a thread as resolved.

```bash
gh pr-review threads resolve --thread-id PRRT_... -R owner/repo <pr-number>
```

**Output:**
```json
{
  "thread_node_id": "PRRT_kwDOABC123",
  "is_resolved": true
}
```

### threads unresolve

Mark a thread as unresolved.

```bash
gh pr-review threads unresolve --thread-id PRRT_... -R owner/repo <pr-number>
```

**Output:**
```json
{
  "thread_node_id": "PRRT_kwDOABC123",
  "is_resolved": false
}
```

## Backend Policy

All commands use GraphQL exclusively (no REST fallbacks):

| Command | GraphQL Operation |
|---------|-------------------|
| `review --start` | `addPullRequestReview` |
| `review --add-comment` | Requires `PRR_...` review node ID |
| `review view` | Aggregates reviews, comments, replies |
| `review --submit` | `submitPullRequestReview` |
| `comments reply` | `addPullRequestReviewThreadReply` |
| `threads list` | Enumerates review threads |
| `threads resolve` | `resolveReviewThread` |
| `threads unresolve` | `unresolveReviewThread` |
