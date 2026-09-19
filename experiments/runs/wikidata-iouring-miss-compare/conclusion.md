# Scattered vs sequential vocabulary access (conclusion)

Run `wikidata-iouring-miss-compare`, Ural seq 3938, 2026-09-19, exit 0.
Binary: `/local/data-ssd/stoetzem/wt/bench/iouring-pre-baseline/build/qlever-server`
(`v0.5.48-36-gfdbfd14f5`, no liburing linkage).
Index: old-format Wikidata truthy backup, build 2026-08-13 (see 7.5).
Queries: `H-vocab-label-large.rq` plus SELECT twin (sequential class scan)
against `H-vocab-random-label.rq` (50,000 scattered subjects, CONSTRUCT).
Driver: `scripts/profile-miss-compare.sh` with `scripts/sample_cpu_stat.py`.

## Method

One warm plus two cold reps per workload (caches cleared before each
cold rep). Each rep records wall time, body bytes, storage bytes reread
(`/proc/PID/io`), machine CPU states plus server CPU time at 2 Hz. All
reps returned HTTP 200 with byte-stable bodies per workload.

## Numbers

| Window | Wall (s) | Body | Storage reread | proc_cpu_pct | iowait |
|---|---|---|---|---|---|
| large warm | 22.99 / 19.68 | 1.3 GB / 674 MB | 0 | 115 / 116 | 0.1 |
| large cold | 22.70 / 19.56 | same | 509 MB | 115 / 117 | 0.1 |
| random warm | 1.15 | 4.9 MB | 0 | 356 | 0.0 |
| random cold | 3.65 / 3.79 | same | 984 MB | 115 / 77 | 3.7 / 3.9 |

Machine `usr` matches the server's own contribution in every window, so
no other load polluted the run. Cold reps agree with each other.

## Interpretation

Scatter, not size, decides whether the disk stalls the CPU. The
sequential export repeats the saturation finding: cold matches warm
within 0.1 s at 0.1 percent wait. The scattered export inverts it: cold
needs 3.2 times the warm wall time, rereads 984 MB for a 5 MB result
(200 times the output), and machine I/O wait rises to 3.9 percent. Warm
scattered export runs at 356 percent CPU, consistent with parallel
evaluation over the 50,000 ranges, while the cold run waits on storage. This is the
workload the `io_uring` bulk-read path must serve; the large query
cannot show it.

## Method limits

Per-syscall `pread` counts were planned but are unobtainable on Ural:
`perf` tracepoints fail with no permission on `/sys/kernel/tracing`,
and `strace` attach fails with operation not permitted (both verified
2026-09-19). Storage volume (`read_bytes`) stands in as the I/O metric.
The driver still launches a bounded `perf stat` window per rep; those
files stay empty and `pread` reads `NA` throughout.
