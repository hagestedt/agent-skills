---
name: Model effort benchmark
description: >-
  Use when measuring cost vs quality by re-running a frozen past-work pack
  across models and effort/thinking levels on any LLM provider.
  Provider-agnostic harness; no vendor CLI required.
---
# Model × effort benchmark (generic)

Use this when you need a **frozen past work pack** re-run across **models and effort (or thinking) levels** so cost vs quality is measured, not guessed. Provider-agnostic: works with any LLM API that exposes model choice, usage/$ (or token counts you can price), and optionally a thinking/effort knob.

For Anthropic Claude Code / Platform command shortcuts, use the companion skill **Claude model effort benchmark** after this harness is set up.

## Inputs (required)

1. **Work pack name** — short slug (e.g. `support-tickets-v1`, `doc-qa-v1`).
2. **Frozen cases** — 10–30 past items with fixed inputs:
   - `cases/` — one file or record per case (prompt + context)
   - `checks/` — acceptance: must-include facts, schema, binary pass, or rubric 1–5
3. **Model list** — pass explicit IDs/names for the provider you use (frontier + mid + small). Names change; do not hardcode vendors in the pack.
4. **Effort / thinking levels** — at least two points on the provider’s scale (e.g. low/high, or temperature/thinking-budget equivalents). Prefer three: low / medium / high.
5. **Optional third axis** — `prompt_current` vs `prompt_cleaned` (anti-patterns stripped: verify-twice rituals, maximally-thorough boosters, mandatory scratchpads, contradictory rules, stale few-shots).

## Hard rules

- Cases are **frozen**. Do not edit gold mid-run to chase a score.
- Fair $ comparisons need a stable request prefix: keep volatile IDs/timestamps out of the system prompt; don’t reorder tools mid-matrix; don’t change effort mid-case unless that is the variable under test.
- One cell = one (model, effort, prompt_variant) × full case set (or a declared train split).
- Never invent $. Log from API usage objects / provider cost reports only. If only tokens are available, convert with a documented price table and version it.
- Hold out a **test** set if you will auto-search configs; never tune on the held-out set.
- No org-specific channels, repos, customers, or people in this skill — put those in the pack or the routine that calls it.

## Steps

### 1. Pack layout

```
bench/<pack-slug>/
  PACK.md          # what this work is, OUT bans, scorer definition
  cases/           # frozen inputs
  checks/          # gold / automated checks
  runs/            # one folder per matrix run (timestamped)
```

Write `PACK.md` first: purpose, case count, pass definition, forbidden outputs / leak bans, provider + price table version.

### 2. Define the scorer

Prefer automated:

- schema / JSON valid
- required strings or facts present
- forbidden strings absent
- optional rubric only when a human or secondary model grades

Record scorer version in `PACK.md`.

### 3. Build the matrix

| | effort low | medium | high |
|---|---|---|---|
| model A | cell | cell | cell |
| model B | cell | cell | cell |
| model C | cell | cell | cell |

Optional: duplicate the matrix for `prompt_cleaned`.

### 4. Run each cell

For each cell:

1. Set model + effort/thinking **before** the conversation.
2. Run every case with identical tools, timeouts, and decoding settings (except the matrix axes).
3. Log per case: pass/fail, $, latency, tool-call count, cache hit rate if the provider exposes it.
4. Aggregate: pass rate, mean/median $, p50 latency, total tool calls.

Write `runs/<timestamp>/RESULTS.md` with the full table.

### 5. Read the curve

For each model, table pass rate vs $ across effort:

- **Steep then flat** — stop raising effort after the last meaningful gain.
- **Flat everywhere** — task isn’t thinking-bound; invest in caching, batching, or prompt cleanup, not more effort.
- **Stronger@low vs weaker@high** — prefer the cheaper cell that still clears the pass bar.

One-line finding required in RESULTS.md: *“Effort stopped paying after __ on __.”*

### 6. Recurrence

Re-run the same pack when a frontier model ships, monthly, or after a major prompt rewrite. Keep prior `runs/` folders; diff pass rate and $ — don’t overwrite history.

### 7. Share / export

Ship `cases/` + `checks/` + `PACK.md` + latest `RESULTS.md`. Strip secrets, customer PII, and internal cost programs. The matrix table is the portable artifact.

## Done bar

- [ ] PACK.md + frozen cases + checks exist
- [ ] Full model × effort table filled with real metrics
- [ ] “Effort stopped paying after X” line written
- [ ] Receipt path reported to the requester

## Anti-patterns

- Tuning gold after seeing failures
- Comparing cells with different tools or timeouts
- Changing effort mid-conversation inside a cell
- Publishing vendor marketing benches as if they were your work shape
- Embedding one vendor’s CLI commands as required steps (put those in a vendor-specific companion skill)
