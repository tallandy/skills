---
name: improve-codebase-performance
description: Measured performance improvement for a specific part of a codebase. Use when the user wants to speed up a code path, reduce latency, improve throughput, lower memory or CPU usage, optimise database queries, improve frontend rendering performance, shrink build/test/runtime cost, or investigate a suspected performance bottleneck without changing behaviour.
---

# Improve Codebase Performance

Improve performance only where there is a measured problem. Preserve behaviour, public interfaces, accessibility, security, correctness, and readability unless the user explicitly chooses a trade-off.

**This is the skill:** turn "this feels slow" into a tight measurement loop, prove the bottleneck, make the smallest high-leverage change, and show before/after numbers.

When exploring the codebase, use the project's domain glossary if it exists and respect ADRs in the area you're touching. Performance work that ignores domain constraints often optimises the wrong thing.

## Phase 1 - Define the Performance Target

Name the exact thing being improved:

- **Operation** - the route, command, query, render path, build step, test suite, job, or function.
- **Metric** - latency, throughput, memory, CPU, allocations, bundle size, frames dropped, query count, cold start, or build time.
- **Scenario** - input size, dataset, viewport, concurrency, cache state, environment, and flags.
- **Budget** - existing target if one exists; otherwise a concrete "better than baseline by X" target.

If the user gives a vague target, infer a sensible one from the code path and say what you chose. Do not optimise the whole codebase.

## Phase 2 - Build a Measurement Loop

Do not change code until there is a repeatable measurement.

Prefer the fastest loop that exercises the real path:

1. Existing benchmark or perf test.
2. Focused unit/integration benchmark at the public interface.
3. HTTP/CLI harness around the route, command, or job.
4. Browser trace or component profiling for UI rendering.
5. Database `EXPLAIN` / query plan plus representative fixture data.
6. Production-like trace, log sample, flamegraph, HAR, or captured payload replay.
7. Throwaway harness only when no real seam exists.

Make the loop stable:

- Warm caches when the production path is warm; cold-start when that is the problem.
- Run enough iterations to reduce noise.
- Pin random seeds, clock inputs, fixture sizes, and concurrency where possible.
- Record median and tail values when latency matters; average alone hides pain.

If you cannot build a measurement loop, stop and ask for the missing artifact: representative input, dataset, trace, profile, access to the environment, or permission to add temporary instrumentation.

## Phase 3 - Establish Baseline + Guardrails

Capture the current numbers before editing.

Also lock behaviour:

- Run relevant correctness tests.
- Add or identify a regression test when the optimisation changes logic.
- Record user-visible output for snapshot/diff comparison when tests are thin.
- For UI work, check interaction, layout, accessibility, and loading states.

The baseline should include enough context that another agent could reproduce it: command, environment, commit, dataset, sample size, and measured values.

## Phase 4 - Profile Before Hypothesising

Use profiling to find where time or memory actually goes. Do not start with favourite optimisations.

Choose tools by stack:

- **JavaScript/TypeScript** - browser Performance panel, React profiler, Node `--prof`, `performance.mark`, bundle analyser, test runner timing.
- **Database** - `EXPLAIN` / `EXPLAIN ANALYZE`, query timing, row counts, indexes, N+1 query inspection, lock waits.
- **Python** - `cProfile`, `py-spy`, `pytest-benchmark`, allocation tracing.
- **PHP** - Xdebug profiler, Blackfire, Tideways, XHProf/XHGui, Laravel Telescope/Debugbar for request/query inspection, PHPUnit timing.
- **Go** - `pprof`, benchmarks, trace, allocation profiles.
- **Rust** - criterion benchmarks, flamegraphs, allocation checks.
- **JVM/.NET** - JFR, async-profiler, dotnet-trace, benchmark harnesses.
- **Frontend** - Web Vitals, long tasks, layout shifts, render counts, network waterfalls, bundle chunks.

Only instrument boundaries that distinguish causes. Tag temporary logs/probes with a unique prefix like `[PERF-9f2a]` so cleanup is mechanical.

## Phase 5 - Rank Bottlenecks

Present a short ranked list before changing code:

```
1. <bottleneck>
   Evidence: <profile/baseline result>
   Prediction: If fixed, <metric> should improve by <expected amount/range>.
   Risk: <correctness/readability/operational risk>
```

Pick the top bottleneck unless the user chooses another. Ignore optimisations that are not visible in the profile, unless they remove known algorithmic complexity from the measured path.

## Phase 6 - Optimise One Variable

Make one performance change at a time.

Good moves:

- Remove repeated work from the hot path.
- Replace an accidental algorithmic cost with a better data structure or query shape.
- Push filtering/aggregation to the right layer.
- Batch I/O that is currently chatty.
- Cache only when invalidation, memory growth, and freshness are understood.
- Reduce rendering work, network waterfalls, bundle cost, or unnecessary subscriptions.
- Move expensive setup out of loops or per-request paths.

Bad moves:

- Trading correctness for speed without explicit user approval.
- Adding global caches with unclear lifetime.
- Micro-optimising code outside the measured bottleneck.
- Making interfaces wider or more surprising to hide implementation cost.
- Keeping a faster version that only wins on toy fixtures.

After each change, rerun the measurement loop and the behaviour guardrails. If numbers do not improve, revert your own change or explain why the trade-off is still worth keeping.

## Phase 7 - Verify Under Shape

A performance fix is not done when one happy-path benchmark improves.

Check the shape that matters for the code path:

- Small, normal, and large inputs.
- Cold and warm cache if both exist.
- Low and high concurrency when relevant.
- Empty, sparse, and dense datasets.
- Desktop and mobile viewports for UI work.
- Memory after repeated runs, not just during the first run.

Watch for shifted cost: faster server but slower client, fewer queries but more memory, faster median but worse p95, smaller bundle but delayed interaction.

## Phase 8 - Cleanup + Report

Before declaring done:

- [ ] Original baseline command rerun.
- [ ] Correctness tests/guardrails pass.
- [ ] Temporary `[PERF-...]` probes removed.
- [ ] Throwaway harnesses deleted or clearly marked if intentionally kept.
- [ ] Before/after numbers reported with commands and confidence.
- [ ] Trade-offs and residual risks named.

Final report format:

```
Target: <operation + scenario>
Metric: <metric>
Baseline: <number> using <command/tool>
After: <number> using <same command/tool>
Change: <plain English summary>
Confidence: <why the result is trustworthy / what noise remains>
Risk: <behaviour, operational, or maintainability risk>
```

If the work reveals that the current module has no good measurement seam, poor locality, or hidden coupling that blocks safe optimisation, say so explicitly and recommend an architectural follow-up.
