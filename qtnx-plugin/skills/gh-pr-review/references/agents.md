# Agent Workflows for gh-pr-review

Best practices and workflows for LLMs and automated agents using gh-pr-review.

## Why gh-pr-review is LLM-Friendly

1. **Single command replaces multi-call API chains** - One `review view` command returns the entire assembled review structure instead of list reviews -> list threads -> list comments.

2. **Deterministic, stable output** - Consistent formatting, stable ordering, predictable field names.

3. **Compact JSON** - Only essential fields, no URLs/hashes/unused metadata. Reduces token usage.

4. **Pre-joined review threads** - Threads come fully reconstructed with inline context.

5. **Server-side filters** - Options like `--unresolved` and `--tail` reduce payload size.

## Common Agent Workflows

### 1. PR Review Summary

To generate a summary of review status:

```bash
# Get all reviews with unresolved threads
gh pr-review review view -R owner/repo --pr 42 --unresolved

# Parse output to:
# - Count total reviews by state
# - List unresolved threads with file locations
# - Identify blocking reviewers (CHANGES_REQUESTED state)
```

### 2. Automated Code Review

To perform automated code review:

```bash
# 1. Start pending review
REVIEW_ID=$(gh pr-review review --start -R owner/repo 42 | jq -r '.id')

# 2. Add comments (repeat for each finding)
gh pr-review review --add-comment \
  --review-id "$REVIEW_ID" \
  --path "src/main.go" \
  --line 42 \
  --body "Consider using constants instead of magic numbers" \
  -R owner/repo 42

# 3. Submit review
gh pr-review review --submit \
  --review-id "$REVIEW_ID" \
  --event COMMENT \
  --body "Automated review completed. Found 3 suggestions." \
  -R owner/repo 42
```

### 3. Thread Resolution Workflow

To resolve threads after fixes:

```bash
# 1. List unresolved threads
THREADS=$(gh pr-review threads list --unresolved -R owner/repo 42)

# 2. For each thread, check if related code was modified
# (Agent logic to verify fix)

# 3. Resolve addressed threads
gh pr-review threads resolve --thread-id PRRT_... -R owner/repo 42
```

### 4. Review Discussion Bot

To respond to review comments:

```bash
# 1. Get thread context
gh pr-review review view -R owner/repo --pr 42 --include-comment-node-id

# 2. Find threads requiring response (e.g., questions)

# 3. Reply with context
gh pr-review comments reply \
  --thread-id PRRT_... \
  --body "This was addressed in commit abc123. The change..." \
  -R owner/repo 42
```

## Token Optimization Strategies

### Reduce Output Size

```bash
# Only unresolved, non-outdated threads
gh pr-review review view -R owner/repo --pr 42 --unresolved --not_outdated

# Keep only latest reply (reduces thread history)
gh pr-review review view -R owner/repo --pr 42 --tail 1

# Filter to specific reviewer
gh pr-review review view -R owner/repo --pr 42 --reviewer important-reviewer
```

### Batch Operations

```bash
# Get thread IDs first (smaller payload)
THREAD_IDS=$(gh pr-review threads list --unresolved -R owner/repo 42 | jq -r '.[].threadId')

# Then resolve in batch
for id in $THREAD_IDS; do
  gh pr-review threads resolve --thread-id "$id" -R owner/repo 42
done
```

## Error Handling

All commands exit non-zero on failure and emit structured JSON errors:

```bash
# Check exit code
if ! gh pr-review review --submit ... ; then
  echo "Review submission failed"
  # Error details in stdout JSON
fi
```

Common errors:
- Invalid `PRR_...` ID format (numeric ID passed instead of GraphQL ID)
- Thread not found
- Permission denied
- Rate limiting

## Integration Tips

1. **Always capture review ID** - When starting a review, immediately save the `PRR_...` ID for subsequent operations.

2. **Use `--include-comment-node-id`** - When building responses that reference specific comments.

3. **Handle empty arrays** - `threads list` returns `[]` when no matches, not null.

4. **Prefer filters over post-processing** - Use server-side filters (`--unresolved`, `--states`, etc.) to minimize token usage.

5. **Check review state before submission** - Pending reviews have no `submitted_at` field.
