# gh-pr-review JSON Schemas

JSON output schemas for each command. Optional fields are omitted rather than set to null.

## review view Output

```json
{
  "reviews": [
    {
      "id": "PRR_...",
      "state": "APPROVED|CHANGES_REQUESTED|COMMENTED|DISMISSED",
      "author_login": "username",
      "body": "...",           // omitted if empty
      "submitted_at": "...",   // omitted if absent (pending reviews)
      "comments": [            // omitted if none
        {
          "thread_id": "PRRT_...",
          "comment_node_id": "PRRC_...",  // omitted unless --include-comment-node-id
          "path": "path/to/file.go",
          "line": 21,          // omitted if null
          "author_login": "username",
          "body": "Comment text",
          "created_at": "2024-01-15T10:30:00Z",
          "is_resolved": false,
          "is_outdated": false,
          "thread": [          // replies only; sorted ascending by created_at
            {
              "comment_node_id": "PRRC_...",  // omitted unless --include-comment-node-id
              "author_login": "username",
              "body": "Reply text",
              "created_at": "2024-01-15T11:00:00Z"
            }
          ]
        }
      ]
    }
  ]
}
```

## review --start Output

```json
{
  "id": "PRR_kwDOAAABbcdEFG12",
  "state": "PENDING"
}
```

## review --add-comment Output

```json
{
  "id": "PRRT_kwDOAAABbcdEFG12",
  "path": "internal/service.go",
  "is_outdated": false,
  "line": 42
}
```

## review --submit Output

Success:
```json
{
  "status": "Review submitted successfully"
}
```

Error:
```json
{
  "status": "Review submission failed",
  "errors": [
    {
      "message": "error description",
      "path": ["mutation", "submitPullRequestReview"]
    }
  ]
}
```

## threads list Output

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

Empty result returns `[]`.

## threads resolve/unresolve Output

```json
{
  "thread_node_id": "PRRT_kwDOABC123",
  "is_resolved": true
}
```

## ID Types Reference

| Prefix | Type | Description |
|--------|------|-------------|
| `PRR_` | Pull Request Review | Identifies a review (pending or submitted) |
| `PRRT_` | Pull Request Review Thread | Identifies an inline comment thread |
| `PRRC_` | Pull Request Review Comment | Identifies individual comments within threads |
