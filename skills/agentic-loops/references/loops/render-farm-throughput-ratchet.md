---
name: render-farm-throughput-ratchet
goal: Push a headless-browser render pipeline to its real throughput ceiling by sharding across browser processes
cadence: on-demand (slow render / before a big video batch)
run_via: /loop
routes_to: [hotspot-scout, perf-max, memguard]
stop_condition: Two consecutive shard increases yield < 5% fps gain (knee found), OR CPU hits sustained saturation
verification_gate: Each iteration must produce a byte-valid, frame-count-correct stitched output (stream-copy verified); a faster run that drops/corrupts frames is FAILED
---

# Render-Farm Throughput Ratchet

**Goal.** Headless-browser render pipelines (Remotion-style video rendering) hit a
well-known wall: more workers, flat fps, idle CPU, because the bottleneck is browser
tabs, not cores. Sharding frame ranges across separate browser *processes* breaks it;
the `memguard` pairing keeps the push from freezing unified-memory hardware.

**Each iteration.**
1. Profile the run: confirm whether CPU is actually saturated or sitting idle (the tell that you're tab-bound, not core-bound).
2. Increase frame-range sharding across separate BROWSER PROCESSES (not more tabs), under `memguard` so unified-memory hardware doesn't thrash; re-measure render fps.
3. Record fps vs. process count; stitch segments stream-copy (frame-exact, closed segments) and confirm no corruption.

**Stop when** two consecutive shard increases yield < 5% fps gain (the knee of the curve), OR CPU hits sustained saturation.

**Verification gate.** Each iteration must produce a byte-valid, frame-count-correct stitched output (stream-copy verified). A faster run that drops or corrupts frames is FAILED. This blocks the "it's faster now" lie that ignores broken output.

## Copy-paste prompt
```
/loop Ratchet this render pipeline's throughput. Each pass: profile to confirm
whether CPU is saturated or idle (idle = tab-bound, not core-bound); increase
frame-range sharding across separate BROWSER PROCESSES (not more tabs), under
memguard, and re-measure render fps; stitch segments via stream-copy (frame-exact)
and verify the output is byte-valid with the correct frame count. A faster run that
drops or corrupts frames is FAILED. Stop when two consecutive shard increases yield
under 5% fps gain, or CPU is sustainedly saturated.
```

**Cadence / notes.** Always shard under `memguard` on unified-memory hardware. Use `hotspot-scout` to score the profile and `perf-max` when the bottleneck turns out to be in the pipeline code itself.
