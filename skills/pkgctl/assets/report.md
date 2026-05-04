# pkgctl report templates

Use the template matching the completed action. `cmd` is a JSON array from
the `event` emitted during execute — space-join the elements to produce a
readable shell string. Omit any line whose value is absent.

---

## install

```
Installed: <package(s)>
Command:   <cmd space-joined>
Warnings:  <stderr output that did not cause failure>
```

---

## remove

```
Removed:  <package(s)>
Command:  <cmd space-joined>
Warnings: <stderr output that did not cause failure>
```

---

## update

```
Package index updated.
Warnings: <stderr output that did not cause failure>
```

---

## search

```
Results for "<pattern>":
  • <package> — <one-line description>
  • <package> — <one-line description>
  …

Use "install <package>" to install any of the above.
```

---

## source

```
Source fetched: <package(s)>
Command:        <cmd space-joined>
Warnings:       <stderr output that did not cause failure>
```

---

## cancelled

```
Cancelled. No changes were made.
```

---

## error

```
Error:  <code>
Detail: <message verbatim from stderr JSON>
```
