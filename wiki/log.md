# Activity Log

Append-only record of every agent operation against this wiki. One line per operation. Never edit or delete existing lines — if a line is wrong, append a correction.

**Format:**

```
YYYY-MM-DD | OP | target | files-touched | note
```

- `OP` ∈ `INGEST` · `QUERY` · `CURATE` · `LINT`
- `target` — source slug (INGEST), topic slug (QUERY), short description (CURATE), or `-` (LINT)
- `files-touched` — comma-separated relative paths, or `-` if none
- `note` — free-form; use for LINT issue counts or CURATE before→after hints

See [AGENTS.md](AGENTS.md) for the full contract.

---
