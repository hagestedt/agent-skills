---
name: Claude model effort benchmark
description: >-
  Use when benchmarking model×effort on Anthropic Claude Platform/Claude Code
  and you want /claude-api prompt-audit, cost-optimize, and hillclimb on top of
  the generic frozen-pack harness.
---
# Claude model × effort benchmark

Use this when the harness runs on **Anthropic Claude Platform / Claude Code** and you want the built-in `/claude-api` commands. For the provider-agnostic pack layout, matrix, scorer, and “effort stopped paying” rule, follow **Model effort benchmark** first (`../model-effort-benchmark/SKILL.md`) — this skill only adds Claude-native accelerators.

Source map (public Anthropic guidance): prompt caching, instruction anti-patterns, effort calibration; commands live in the `claude-api` skill.

## When to use which command

| Command | When |
| --- | --- |
| `/claude-api prompt-audit` | Migrating to a frontier Claude model, or before a `prompt_cleaned` matrix axis. Scans prompts, skills, tool descriptions (app code and Claude Code config such as CLAUDE.md / skills). Removes anti-patterns that waste tokens on frontier models (verify-twice, maximally thorough, mandatory scratchpads, contradictory rules, stale few-shots, dated thinking settings). |
| `/claude-api cost-optimize` | App uses the Claude API and you want a spend audit. Profiles token spend (Admin API, response usage logs, or request-building code). Applies caching, prompt-audit, output bounds, batching. With an eval, also sweeps effort and model. |
| `/claude-api hillclimb` | You already have a frozen eval (train/test). Searches model × effort × prompt changes to cut $ while holding baseline quality; scores the final config on held-out test. |

## Steps (Claude path)

1. **Build the pack** with [Model effort benchmark](../model-effort-benchmark/SKILL.md) (`PACK.md`, `cases/`, `checks/`).
2. **Baseline matrix** — run model × effort once on the current prompt; write `runs/<ts>-baseline/RESULTS.md`.
3. **Prompt-audit axis (recommended)**
   - Run `/claude-api prompt-audit` on the pack’s system prompt / skills / tool descriptions.
   - Save the cleaned prompt as `prompt_cleaned` (do not overwrite the baseline prompt file without versioning).
   - Re-run the matrix (or hillclimb) on the cleaned prompt.
4. **Cost-optimize (optional, app code)**
   - If the workload is an application calling the Messages API, run `/claude-api cost-optimize`.
   - Apply safe wins that don’t change the scorer: cache breakpoints, batch for offline jobs, output bounds.
   - Re-measure the same pack.
5. **Hillclimb (optional)**
   - Split cases into train/test in `PACK.md`.
   - Run `/claude-api hillclimb` from the baseline config.
   - Accept only configs that clear the train bar; report held-out test pass rate and $ in RESULTS.md.
6. **Cache hygiene (manual checks Claude cares about)**
   - Same model for a cached prefix; byte-exact prefix; watch TTL (prefer longer TTL if tools/subagents block >5 minutes).
   - Keep volatile timestamps/IDs out of the system prompt; avoid reordering tool definitions mid-run.
   - Don’t change effort mid-conversation unless on models that allow it without breaking cache (per current Claude Platform docs).
   - Log cache hit rate from Console / diagnostics when available.
7. **Finish** — same done bar as the generic skill, plus note which `/claude-api` commands were run.

## Done bar

- [ ] Generic harness complete (pack + matrix + “effort stopped paying” line)
- [ ] If used: prompt-audit before/after noted
- [ ] If used: cost-optimize / hillclimb outputs linked from RESULTS.md
- [ ] No invented $; Claude usage objects or Admin reports only

## Anti-patterns

- Running hillclimb without a held-out test set
- Treating Anthropic public benches as a substitute for your frozen work pack
- Editing gold after hillclimb failures
- Putting customer PII or internal Claude contract $ into the shared pack
