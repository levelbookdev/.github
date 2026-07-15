# Contributing to Levelbook

This is the **org-wide** contributing guide. It applies to every Levelbook
product repo (`trenchnote`, `loopcheck`, `slipsign`, `mainline`) unless that
repo ships its own `CONTRIBUTING.md`, which takes precedence.

Bug reports, field stories, and doc fixes are always welcome as issues. Each
product's own `CLAUDE.md` holds its non-negotiable ethos and locked tech stack —
read it before sending code for that product.

## The one org-wide rule: sibling files carry a pointer header

Levelbook products **fork-and-own** their shared plumbing — they copy working
files between repos rather than share a library (see [PATTERNS.md](PATTERNS.md)
for the full doctrine and the file-by-file registry). The standing risk is
**silent divergence**: a bug fixed in one copy but not its siblings.

The mitigation is a one-line header comment on every copied ("sibling") file,
pointing back to the registry:

```
// Sibling file — see PATTERNS.md (offline write queues). Fix here → check siblings.
```

Name the pattern family in the parentheses so the reader can jump straight to
the right registry row.

### Which files are sibling files

Any file that exists as an adapted copy in more than one product — the families
tracked in the [PATTERNS.md registry](PATTERNS.md#2-the-sibling-files-registry):
offline write queues, auth helpers, service workers, design tokens, seed/demo
scripts, smoke-test scripts, and QR label pages. Product-only files (e.g.
MainLine's `ml-station.js`) are listed there too, so nobody re-invents them in a
sibling.

### How to apply the rule

- **Add the header the next time you touch a sibling file for a real reason** —
  a bug fix, a feature, a refactor. Piggyback it on that edit.
- **Never do a dedicated sweep to add headers.** A standalone comment change to a
  service worker forces a `VERSION` bump and a re-deploy across every product for
  zero user benefit. One honest edit at a time; the convention propagates for
  free. This is the maintenance rule in [PATTERNS.md §4](PATTERNS.md#4-maintenance-rule--the-pointer-header-convention).
- **When you fix a bug in a sibling file, check its siblings** — the header
  exists to make that check reflexive. If the same fix applies to a sibling,
  apply it there in the same change (or open an issue naming the sibling).

## Multi-session efforts: SESSION-NOTES.md

Any effort too big for one working session gets a `SESSION-NOTES.md` at the repo
root so the next (cold-start) session can resume in minutes. The full convention
— living-state vs. append-only log, the "Next session: start here" line, the
resume ritual, and completion — lives in
[PATTERNS.md §5](PATTERNS.md#5-the-session-notes-convention).
