---
name: pkgctl
description: >
  Use this skill whenever the user wants to manage system-level packages —
  even if they never mention a package manager by name. Trigger on: "install
  X", "get me X", "I need X on this machine", "set up X", "remove X",
  "uninstall X", "purge X", "update my packages", "upgrade the system",
  "search for a tool that does X", "find a package for X", "get the source
  for X". Also trigger when the user hits a missing-binary error and asks you
  to fix it — that is an implicit install request. Do NOT use for
  language-level package managers (pip, npm, cargo, gem, brew).
compatibility: Requires bash and jq.
allowed-tools: discovered_tool_PackageControl
---

# pkgctl

`discovered_tool_PackageControl` is the only way to interact with the system
package manager. All output is structured JSON — no text scraping needed.

### Invocation — STDIN only

The tool reads the JSON payload from STDIN — never pass it as a CLI argument.
Output goes to stdout (NDJSON events) and stderr (structured JSON error on
failure). See [references/REFERENCE.md](references/REFERENCE.md) for event
types, error schema, and flag details.

---

## Action decision table

Determine the action from the user's request, then follow every step in that
row exactly. Never skip or combine steps.

| Action    | Auto-update cache? | Packages field? | Dry-run? | Confirm? | Destructive warn? |
|-----------|--------------------|-----------------|----------|----------|-------------------|
| `search`  | First time only ‡  | Required        | No       | No       | No                |
| `update`  | —                  | Must omit       | No       | No       | No                |
| `install` | First time only ‡  | Required        | Yes      | Yes      | No                |
| `remove`  | No                 | Required        | Yes      | Yes      | Yes ★             |
| `source`  | First time only ‡  | Required        | Yes      | Yes      | No                |

‡ Run `action: update` automatically before the first cache-dependent action
  of the session (`search`, `install`, or `source`). Skip once the cache has
  been warmed for the session. `remove` operates on the local package database
  and does not need a cache update.  
★ For `remove`, warn explicitly that the operation is irreversible before
  asking for confirmation.

---

## Step details

**Auto-update cache** — Call `discovered_tool_PackageControl` with
`action: update`. Omit the `packages` field entirely. Wait for an `event`
type on stdout before continuing.

**Dry-run** — Call `discovered_tool_PackageControl` with `dry_run: true`.
The tool returns a `dry_run` event without executing anything. Space-join the
`cmd` array and show it to the user as the command that would run. Do not
proceed to Confirm until the user has seen it.

**Confirm** — Ask the user to approve. Wait for an affirmative reply. If the
user declines, stop and use the `cancelled` template in [assets/report.md](assets/report.md).

**Destructive warn** — Before confirming a `remove`, state clearly that
removing a package cannot be undone. Then ask for confirmation once.

**Execute** — Call `discovered_tool_PackageControl` again with `dry_run`
omitted (or set to `false`). This is a separate call from the dry-run — the
tool does not remember prior calls.

**Report** — Read [assets/report.md](assets/report.md) and use the template
matching the completed action. Fill in all available fields from the event
payload; omit any line whose value is absent.

---

## Error handling

On failure, read `code` and `message` from the stderr JSON. Consult
[references/REFERENCE.md](references/REFERENCE.md) for the full code list,
meanings, and recovery actions.

---

## Gotchas

- **Never pre-flight check.** Do not verify that `pkgctl`, `sudo`, or the OS
  version exist via a shell tool. The tool handles this internally.
- **`search` requires a packages field.** Pass the search term or pattern in
  `packages` — omitting it returns `NO_PACKAGES`.
- **`update` must not include a packages field.** Passing one causes
  `INVALID_SCHEMA`.
- **`source` may not be available on all systems.** If the tool returns
  `UNSUPPORTED_ACTION`, surface this to the user rather than retrying.
- **Errors go to stderr, events go to stdout.** Read both streams; do not
  assume the tool is silent on failure.
- **Never search before install.** If a named package doesn't exist the tool
  returns `NO_PACKAGES`. Pre-emptive searching wastes a round-trip.
- **Flags are whitelisted.** Only `-y` and `--no-weak-deps` are accepted.
  Any other flag returns `INVALID_FLAG`.
- **Dry-run and execute are separate calls.** The tool is stateless. After a
  dry-run, call it again without `dry_run` to execute — do not assume the
  prior intent is remembered.

---

## Runtime files

See [references/REFERENCE.md](references/REFERENCE.md).
