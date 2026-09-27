# Profiler Routing: Hotspot Scout

Which tool produces which evidence, per stack. Prefer sampling profilers on the
real workload (low overhead, prod-shaped); reach for tracing/instrumented tools
only when sampling can't answer the question, and never trust timings taken
under heavy instrumentation.

## Node / V8

| Dimension | Tool | Notes |
|---|---|---|
| CPU | `node --cpu-prof <app>` → `.cpuprofile` (open in Chrome DevTools), or `npx 0x <app>` for a direct flamegraph | clinic.js `flame`/`doctor` if installed |
| MEM | `node --heap-prof`, heap snapshots in DevTools | Watch retained size growth across snapshots for leaks |
| LOCK/waits | Event-loop lag: `perf_hooks.monitorEventLoopDelay()`; clinic `bubbleprof` for async waterfalls | A laggy loop with idle CPU = blocked on sync I/O or a long task |
| I/O | `strace -c -f` around the process; count `fsync`/`read`/`write`; for HTTP, log per-leg timings | N+1 queries show up as repeated short DB round-trips |

Gotcha: don't time runs with `--inspect` attached; inspector overhead pollutes
the baseline. Profile run and timing run can be separate; the flamegraph names
the path, the clean runs price it.

## Python

| Dimension | Tool | Notes |
|---|---|---|
| CPU | `py-spy record -o evidence/cpu.svg --pid <PID>` (attach, no code change) or `py-spy record -- python app.py` | `--idle` to include off-CPU; `top` mode for live triage |
| MEM | `memray run -o out.bin python app.py` → `memray flamegraph`; `tracemalloc` for targeted diffs | scalene gives CPU+mem in one pass if installed |
| LOCK/waits | `py-spy dump` repeatedly on a stuck/slow process. Recurring identical stacks = the wait; GIL contention shows as one thread on-CPU, rest waiting | asyncio: check for accidental blocking calls in coroutines |
| I/O | `strace -c`, plus app-level timing around DB/network calls | cProfile call counts expose N+1 patterns cheaply |

Gotcha: cProfile's tracing overhead (often 1.3–2×) distorts ratios on tight
loops. Use it for call counts and structure, py-spy for time shares.

## Rust

| Dimension | Tool | Notes |
|---|---|---|
| CPU | `cargo flamegraph` (perf + inferno), or `samply record ./target/release/app` | MUST be `--release` with `debug = true` for symbols |
| MEM | `heaptrack ./app` or dhat; for steady-state RSS, sample `/proc/<pid>/status` over the run | Alloc-count hot paths matter even when RSS is flat (allocator churn) |
| LOCK/waits | `perf sched` / off-CPU flamegraphs; tokio apps: `tokio-console` for stalled tasks and starved executors | A blocking call inside an async task starves the whole worker |
| I/O | `strace -c`, `iostat -x 1` during the run | Buffered-vs-unbuffered writes and fsync storms dominate many "slow" Rust CLIs |

## Browser / frontend

| Dimension | Tool | Notes |
|---|---|---|
| CPU | Chrome DevTools Performance panel recording on the real interaction; Long Tasks in the trace | React: Profiler panel for re-render storms |
| MEM | DevTools Memory panel, heap snapshot diffing across the interaction | Detached DOM nodes = the classic leak |
| Waits | Network panel waterfall; Web Vitals (LCP / INP / CLS) as the p75 metrics | INP regressions are usually one long task, findable in the trace |

Headless evidence: `npx lighthouse <url> --output json` into `evidence/` for a
repeatable baseline.

## System-wide / "is it even the code?"

- `pidstat -urd 1`, `iostat -x 1`, `vmstat 1`: split CPU vs iowait vs swap.
- `/proc/pressure/{cpu,io,memory}` (PSI): the box's own contention verdict.
- `strace -c -f -p <PID>`: syscall census; `fsync` and `futex` counts are the
  fast tells for I/O-bound and lock-bound respectively.
- `perf top`: quick live look when you don't know which process is hot.
- If the MACHINE is the bottleneck (swap, cache bloat, competing builds), that's
  system-performance-remediation's job. Note it in HOTSPOTS.md and stop; the
  code list would be ranking noise.

## Databases

- Postgres: `EXPLAIN (ANALYZE, BUFFERS)` on the queries the workload actually
  issues; `pg_stat_statements` ranked by `total_exec_time` IS a scored list;
  fold its top rows into HOTSPOTS.md as `io`/`cpu` rows with the query as the
  hotspot name.
- SQLite: `EXPLAIN QUERY PLAN`; count fsyncs (`strace -c`). Journal mode and
  per-row transactions are the usual culprits.

## Local LLM / inference (only if the system under test calls a local model endpoint, e.g. Ollama :11434 / vLLM :8000)

- Metrics that matter: TTFT, tokens/s, batch occupancy, from vLLM's `/metrics`
  or timed Ollama calls; the model server is a hotspot ROW, not a black box.
- Known cliff on unified memory: two models resident at once (responses
  0.6s → 16s). Verify only one model is loaded before baselining anything
  that calls Ollama.
- vLLM `--gpu-memory-utilization` ≤ 0.75 on unified-memory hardware (GB10
  class). Above that, system RAM starvation makes *everything else* profile
  slow. (Discrete-VRAM boxes don't have this cliff.)

## Unified-memory measurement notes (e.g. NVIDIA DGX Spark / GB10; skip on discrete-GPU or plain laptops)

- Wrap heavy profiled runs: `memguard run --profile <X> -- CMD` (if memguard
  is installed; on other machines, run directly). A profiling
  session that freezes the box yields no list.
- Memory truth: `/proc/meminfo` + PSI. nvidia-smi memory numbers are N/A on UMA.
- One GPU-heavy app at a time during baseline runs. A background finetune or
  resident model silently inflates every number you record.
- arm64: `perf` works; some prebuilt profiler binaries are x86-only. Prefer
  pip/cargo-installed tools (py-spy, samply, flamegraph) which ship arm64 builds.
