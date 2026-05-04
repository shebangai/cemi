# pkgctl reference

## Output event types (stdout, newline-delimited JSON)

| `type`    | When emitted                              | Key fields                              |
|-----------|-------------------------------------------|-----------------------------------------|
| `event`   | Every dispatch (dry-run or live)          | `ts`, `action`, `cmd`, `dry_run`       |
| `journal` | After a successful `install`              | `action`, `packages`                   |
| `dry_run` | When `dry_run: true`; command not run     | `cmd` (array)                           |

## Error object (stderr, non-zero exit)

```json
{
  "type":    "error",
  "code":    "<ERROR_CODE>",
  "message": "<human-readable detail>",
  "usage":   { ... },
  "example": { ... }
}
```

## Error codes

| Code                 | Meaning                                                                | Recovery                                              |
|----------------------|------------------------------------------------------------------------|-------------------------------------------------------|
| `LOCKED`             | Another pkgctl instance holds the flock lock                           | Wait 5 s, retry the failed step once                  |
| `NOT_ROOT`           | Tool was not invoked as root (via sudo)                                | Surface to user; do not retry                         |
| `NO_JQ`              | jq is not installed on the system                                      | Surface to user; do not retry                         |
| `INVALID_SCHEMA`     | Input JSON failed type/structure validation                            | Fix field types and retry                             |
| `INVALID_ACTION`     | `action` value not in the allowed enum                                 | Check spelling; retry with correct action             |
| `INVALID_FLAG`       | Flag not in the allowed whitelist                                      | Remove the flag and retry                             |
| `NO_PACKAGES`        | `packages` missing or empty for an action that requires it             | Ask user to clarify; do not retry                     |
| `UNSUPPORTED_SYSTEM` | No supported package manager found on this host                        | Surface to user; do not retry                         |
| `UNSUPPORTED_ACTION` | Requested action not supported on this host's package manager          | Surface to user; do not retry                         |
| *(any other)*        | Unrecognised error condition                                           | Surface `code` and `message` verbatim; do not retry   |

## Allowed flags

| Flag             | Effect                                                                 |
|------------------|------------------------------------------------------------------------|
| `-y`             | Pass `--yes` / `-y` to the package manager                            |
| `--no-weak-deps` | Skip optional/recommended deps (the tool maps this flag internally)   |

## Runtime files

| File      | Path                         | Scope   | Purpose                                        |
|-----------|------------------------------|---------|------------------------------------------------|
| Journal   | `/tmp/pkgctl-journal.ndjson` | session | Append-only NDJSON audit log of installs       |
| Lockfile  | `/tmp/pkgctl.lock`           | session | flock-based mutual exclusion across invocations|
