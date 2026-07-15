# PATTERNS — the Levelbook sibling-files registry & doctrine

Levelbook products are **sibling repos that deliberately copy shared
plumbing** — `trenchnote`, `loopcheck`, `slipsign`, `mainline`. They share one
architecture (PocketBase backend; static vanilla HTML/CSS + Alpine.js frontend;
no build step; vendored libraries; append-only ledgers with derived status) but
**not one codebase**. This file is the map of what got copied, where the copies
now live, and where they have drifted.

Read it before you touch a file that has siblings. Its one job is to make
silent divergence loud.

---

## 1. Purpose: fork-and-own, and why there is no shared library (yet)

**Fork-and-own** means: when a new product needs the offline write queue, or the
auth helper, or the service worker, we *copy the working file from a sibling and
adapt it in place* — we do not import it from a shared package. Every product
owns a complete, standalone copy of its plumbing. The copy carries a header
comment naming the sibling it was adapted from (see `ml-sync.js`: "adapted from
TrenchNote's tn-sync.js").

Why no shared library:

- **No build step is a feature.** These apps are static files served straight
  from `pb_public/`. A shared library means a bundler, a version, a publish
  step, and a release dance — exactly the machinery we deleted on purpose.
- **Products diverge legitimately.** MainLine records carry photos; TrenchNote
  movements don't. LoopCheck's field tier is accountless; the others aren't.
  A premature shared abstraction would fight these real differences.
- **Two products isn't a pattern.** You cannot design the right shared seam
  from two examples. You need three or four in production, rubbing against each
  other, before the true shape is visible.

**The cost we accept:** a bug fixed in one copy is not fixed in its siblings.
That is the whole risk this document exists to manage.

**The extraction trigger.** When **3+ active products are churning the same
file** — real commits, in the same month, fixing the same class of thing in
parallel copies — that is the signal the shape has stabilized and the copying
tax now exceeds the library tax. At that point, extract the stabilized piece
into an **MIT-licensed `fieldkit`** (permissive on purpose, so the shared core
can be vendored into products under any license) and have the products vendor
it. Until then: copy, and keep this registry honest.

---

## 2. The sibling-files registry

Paths are relative to each product repo root. "—" means the product has no
copy of that pattern. Divergences are the load-bearing column — read them.

### Offline write queues

| Repo | File | Notes |
|------|------|-------|
| TrenchNote | `pb_public/tn-sync.js` | Reference impl. IndexedDB, one store; FIFO replay per phone; pre-generated record ids make replay idempotent; visible pending-count badge, red on failure. |
| MainLine | `pb_public/ml-sync.js` | Adapted from `tn-sync.js`. Same rules, **plus photos**: queued entry holds the field object + `{name, blob}` array, replay rebuilds a multipart FormData POST so record+photos land atomically. |
| LoopCheck | `pb_public/calibration.html`, `pb_public/tag.html`, `pb_public/notice_batch.html` (inline) | **Divergence:** no shared sync file. Per-page **localStorage** outboxes (e.g. `lc_calibration_outbox`), not IndexedDB, not FIFO-idempotent. Lighter, but the tn-sync guarantees don't apply — audit each page on its own. |
| SlipSign | `pb_public/index.html` (inline) | **Divergence:** the offline store is a **vault**, not a queue — IndexedDB stores `drafts` / `signatures` / `events`. Different shape because a signature is the artifact, not a replayable move. |

### Auth helpers

| Repo | File | Notes |
|------|------|-------|
| TrenchNote | `pb_public/tn-auth.js` | The one deliberate exception to "every page self-contained" (its ADR 0001). `TN.requireLogin()` + `TN.fetch()`; token in localStorage; used by 8 pages. |
| LoopCheck | `pb_public/lc-auth.js` | `window.lcAuth`, loaded before Alpine, ~15 pages. **Divergence:** field tier is **accountless** (public reads + one public punch write), so most field pages never call it — it gates office/authenticated actions only. No self-signup. |
| MainLine | `pb_public/ml-auth.js` | Adapted from `tn-auth.js`. `ML.requireLogin()` + `ML.fetch()`, same model. |
| SlipSign | — | Single-page app; no separate auth helper. |

### Service workers (+ the bump-VERSION rule)

| Repo | File | Notes |
|------|------|-------|
| TrenchNote | `pb_public/sw.js` | Reference impl (its ADR 0008). Two caches: SHELL cache-first, API network-first with `X-TN-Cached-At` staleness stamp. Writes pass straight through. |
| MainLine | `pb_public/sw.js` | Adapted from TrenchNote's; `X-ML-Cached-At`. |
| SlipSign | `pb_public/sw.js` | **Divergence:** shell-only, **no API caching** — ticket data must never come from an HTTP cache. Version key is `const CACHE = "slipsign-shell-v2"`, not `VERSION`. |
| LoopCheck | — | **Divergence:** no service worker at all. Offline resilience is per-page localStorage while the tab stays open; a cold load needs signal. |

**The bump-VERSION rule:** the service worker precaches the app shell. A deploy
that changes any shelled file **must bump the version constant in `sw.js`**
(`const VERSION = 'vN'` in TN/ML; the `slipsign-shell-vN` string in SlipSign) —
that is what makes the browser install the new worker and re-download the shell.
Forget the bump and devices serve stale HTML forever.

### Design tokens / shared color set

| Repo | File | Notes |
|------|------|-------|
| LoopCheck | `pb_public/app.css` | The **only** product with a shared stylesheet. `--ink #14161a`, `--safety #ff5a1f`, plus the severity A/B/C color language. |
| TrenchNote | inline in every `.html` | Same values as LoopCheck (`--ink #14161a`, `--safety #ff5a1f`) — but duplicated into each page's `<style>`. |
| MainLine | inline in every `.html` | `--ink #14161a` matches, but the accent is renamed `--brand`. **Divergence:** same intent, different token name. |
| SlipSign | inline in every `.html` | **Divergence (widest):** `--ink #1a2330` and `--accent #e8640c` (+ `--accent-dark`) — the near-black and the orange have both drifted to different hex than the rest of the family. |

### Station parsing

| Repo | File | Notes |
|------|------|-------|
| MainLine | `pb_public/ml-station.js` | **MainLine-only.** The single place station strings are parsed/formatted. Stores both `station_ft` (decimal feet, for math) and `station_display` (`"14+52.5"`). No deps, no DOM; `tests.html` beats on it with messy real inputs. Nothing to keep in sync — listed so nobody re-invents it in a sibling. |

### Seed / demo scripts

| Repo | File | Notes |
|------|------|-------|
| TrenchNote | `scripts/seed_demo.sh`, `scripts/seed_local_june2026.sh`, `scripts/setup.sh` | |
| LoopCheck | `scripts/seed_demo.sh`, `scripts/setup.sh` | Data lives in `seed/*.json` template files. |
| MainLine | `scripts/seed_demo.sh`, `scripts/setup.sh` | |
| SlipSign | `scripts/seed-demo.sh`, `scripts/setup.sh`, `scripts/seed-assets/` | **Divergence:** hyphenated `seed-demo.sh` vs the others' `seed_demo.sh`. Ships sample `photo.jpg` / `signature.png`. |

### Smoke-test scripts

| Repo | File | Notes |
|------|------|-------|
| LoopCheck | `scripts/smoke_test.sh` | |
| SlipSign | `scripts/test-rules.sh` | Exercises PocketBase API rules. |
| TrenchNote | `tests/gang_boxes.ps1` | A single PowerShell scenario test. |
| MainLine | — | No smoke test yet. |
| — | — | **Divergence:** no shared harness; different languages (`sh` vs `ps1`), different scope. A candidate for consolidation if a third product needs one. |

### QR label pages

| Repo | File | Notes |
|------|------|-------|
| TrenchNote | `pb_public/labels.html` | Renders QR labels via vendored `qrcode.min.js`. Also vendors `jsQR.min.js` for scanning (TrenchNote-only). |
| LoopCheck | `pb_public/labels.html` | `qrcode.min.js`. |
| MainLine | `pb_public/labels.html` | `qrcode.min.js`. |
| SlipSign | — | No label page — signatures, not tags. |

### Signature capture + on-device PDF — **SlipSign is the reference**

| Repo | File | Notes |
|------|------|-------|
| SlipSign | `pb_public/index.html` + `vendor/signature_pad.umd.min.js`, `vendor/pdf-lib.min.js`, `vendor/sha256.min.js` | Canvas signature pad → payload SHA-256 → **PDF generated on-device with pdf-lib**, no server render, so it works the moment the signature lands in a trench. Any sibling that later needs signatures or on-device PDF **copies from here** rather than reinventing. |

### Domain derivation helpers (embody "derived status, never stored")

| Repo | File | Notes |
|------|------|-------|
| MainLine | `pb_public/ml-restoration.js` | Zone status + temporary-patch age, derived on read; never stored in PocketBase. |
| TrenchNote | `pb_public/tn-containers.js`, `pb_public/tn-inspect.js` | Gang-box effective-location derivation; inspection state. Product-specific, but the same doctrine — status is computed, not persisted. |

---

## 3. Doctrine (short, quotable)

- **When you fix a bug in a sibling file, check its siblings.**
- **Core notifies about one event at capture time; the paid sidecar aggregates,
  schedules, and analyzes.**
- **Capability the sidecar needs lands in core first, for everyone.**
- **Derived status, never stored.**
- **Append-only ledgers; corrections are new records.**

---

## 4. Maintenance rule — the pointer-header convention

Each sibling file should carry a **one-line header comment pointing here**, e.g.:

```
// Sibling file — see PATTERNS.md (offline write queues). Fix here → check siblings.
```

**Do not add these headers in a sweep now.** Adding a comment to every `sw.js`
is a shell change that would force a pointless `VERSION` bump — and a re-deploy —
across every product on the same day, for zero user benefit. Instead: **add the
header the next time you touch each file for a real reason.** The convention
propagates for free, one honest edit at a time.

---

## 5. The SESSION-NOTES convention

Any effort too big for one Claude Code session gets a **`SESSION-NOTES.md`** at
the repo root. It exists so the NEXT session — which starts cold — can resume in
minutes instead of re-deriving state. It is written for a reader with **zero
memory of previous sessions**.

**Rules:**

- **One file, one effort.** If a second multi-session effort starts in the same
  repo before the first ends, that's a signal to finish the first. (If truly
  unavoidable: `SESSION-NOTES-<slug>.md`.)
- **Committed, not gitignored** — it must survive machine changes and be visible
  to cloud sessions. Because these repos are public: **nothing goes in it that
  isn't already implied by the public docs.** No IPs, no credentials, no
  deployment specifics, no exploit-level security detail — "rule X is still
  permissive" is fine when the README already says so; "here is how you'd abuse
  it from this URL" is not.
- **Two zones.** The top block is **LIVING STATE** — overwritten freely each
  session so the current truth is always first. The session log below is
  **APPEND-ONLY** — one dated entry per session, newest at the top of the log,
  never edited afterward. (Same ledger discipline as the products: current state
  is derived reading; history is immutable.)
- **The "Next session: start here" line is the whole point** — one imperative
  instruction naming a file, a command, or a checkpoint, not a paragraph of
  context. If you can't write it, the session isn't done wrapping up.
- **Resume ritual:** every resuming prompt begins *"Read CLAUDE.md, then
  SESSION-NOTES.md, then continue the effort."* The notes file never duplicates
  CLAUDE.md's constraints — it **links** to them.
- **Completion:** the final session **deletes the file in the same commit that
  ships the last piece** (git history keeps it), and moves anything of lasting
  value into the real docs — decisions to ADRs, discoveries to the developer
  guide. A `SESSION-NOTES.md` in a repo always means "effort in flight."

**File template:**

```markdown
# SESSION-NOTES — <effort name>

> Read CLAUDE.md FIRST — this file never repeats its constraints, it links to them.
> Delete this file in the same commit that ships the last piece of this effort.

**Goal (definition of done):** <one sentence — the observable end state>
**Started:** <YYYY-MM-DD>   **Status:** checkpoint <N> of <M>

## Current state (living — overwrite freely)
- **Done:** <bullets>
- **In progress:** <bullets>
- **Not started:** <bullets>

## Next session: start here
<one imperative instruction — a file, a command, or a checkpoint>

## Decisions made mid-effort
- <YYYY-MM-DD> <one-liner>. (Anything hard to reverse gets promoted to a real
  ADR before the effort ends — note its number here once assigned.)

## Gotchas discovered
- <one line each — things that cost >15 min and would cost the next session the same>

## Session log (append-only, newest first — never edit old entries)
### <YYYY-MM-DD>
- Attempted: <what>
- Landed: <commits>
- Learned: <what>
- Stopped because: <why it stopped where it did>
```
