# CPU load vs I/O wait during export (conclusion)

Run `wikidata-iouring-cpu-wait`, Ural seq 3924, 2026-09-18, exit 0.
Binary: `/local/data-ssd/stoetzem/wt/bench/iouring-pre-baseline/build/qlever-server`
(`v0.5.48-36-gfdbfd14f5`, no liburing linkage).
Index: old-format Wikidata truthy backup, build 2026-08-13 (see 7.5).
Queries: `H-vocab-label-large.rq` (CONSTRUCT `turtle_export`) and
`H-vocab-label-large-select.rq` (SELECT `csv_export`).
Driver: `scripts/cpu-wait-export.sh` with `scripts/sample_cpu_stat.py`.

## Method

One export per workload, warm (page cache hot) and cold (caches cleared
before the rep). Across each request window the driver samples machine
`/proc/stat` and server process CPU time at 2 Hz. `proc_cpu_pct` is mean
busy cores (100 = one core saturated). All four reps returned HTTP 200
with full bodies (1302749672 and 674222797 bytes).

## Numbers

| Window | Wall (s) | proc_cpu_pct | usr | sys | iowait | idle |
|---|---|---|---|---|---|---|
| warm-construct | 22.63 | 115.1 | 7.1 | 0.5 | 0.1 | 92.4 |
| warm-select | 19.61 | 116.8 | 7.2 | 0.3 | 0.1 | 92.4 |
| cold-construct | 22.59 | 115.7 | 7.1 | 0.7 | 0.1 | 92.1 |
| cold-select | 19.66 | 116.8 | 7.2 | 0.4 | 0.0 | 92.4 |

Machine `usr` matches the server's own contribution (115 percent of 16
cores is 7.2 percent), so no other load polluted the windows.

## Interpretation

The waiting hypothesis is refuted. The server burns 115 to 117 percent
CPU in every window while machine I/O wait stays at 0.1 percent or
below, warm and cold alike. Cold runs match warm runs within 0.1 s, so
rereading evicted data costs no visible stall: the disk keeps up and the
CPU never idles on it. The export loop is CPU-saturated on little more
than one core. The `io_uring` motivation is therefore not overlapping
stalls. It is cutting the lookup CPU itself through locality, shared
decode work, and bulk reads.

## Provenance notes

Seqs 3906, 3917, 3919, 3922 were failed attempts (wrong index default,
port race, `set -u` bug, wedged `perf stat`). Their processes were
killed and the run directory reset before 3924, so every file cited
above comes from the single clean run.
