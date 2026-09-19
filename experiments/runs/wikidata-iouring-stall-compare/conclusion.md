# CPU load during disk I/O on a long export (conclusion)

Run `wikidata-iouring-stall-compare`, Ural seq 3939, 2026-09-19, exit 0.
Binary: `/local/data-ssd/stoetzem/wt/bench/iouring-pre-baseline/build/qlever-server`
(`v0.5.48-36-gfdbfd14f5`, no liburing linkage).
Index: old-format Wikidata truthy backup, build 2026-08-13 (see 7.5).
Queries: `H-vocab-label-large.rq` (sequential control) against
`H-vocab-random-label-x500k.rq` (500,000 scattered subjects, generated
by scaling the 50k pattern tenfold).
Driver: `scripts/profile-stall-compare.sh` with `scripts/sample_cpu_stat.py`.

## Method

One warm plus two cold reps per workload (caches cleared before each
cold rep). Each rep records wall time, body bytes, storage bytes reread
(`/proc/PID/io`), and 2 Hz samples of machine CPU, server CPU time, and
server read rate. The analysis conditions CPU load on high-I/O
intervals: samples at or above half the peak sample read rate. All reps
returned HTTP 200 with byte-stable bodies per workload.

## Numbers

| Window | Wall (s) | Body | Storage reread | proc_cpu_pct | iowait | peak MB/s | high share | high cores |
|---|---|---|---|---|---|---|---|---|
| large warm | 22.75 | 1.3 GB | 0 | 115.1 | 0.0 | 0 | 1.00 | 1.15 |
| large cold | 22.51 / 22.23 | same | 509 MB | 115.8 / 115.5 | 0.0 | 80 | 0.16 / 0.11 | 1.39 / 1.38 |
| scatter warm | 9.76 | 48 MB | 0 | 111.5 | 0.1 | 0 | 1.00 | 1.11 |
| scatter cold | 13.99 / 14.05 | same | 2009 MB | 90.9 / 90.4 | 1.4 | 545 / 576 | 0.25 | 1.48 / 1.41 |

Machine `usr` matches the server's own contribution in every window, so
no other load polluted the run. Cold reps agree with each other.

## Interpretation

The long window bounds the overlap prize. On the scattered export the
cold-minus-warm wall gap is 4.2 s but the CPU gap is only 2.1 s, so
about 2 s, or 15 percent of the cold wall, is non-CPU stall. Even
during peak reads near 550 MB/s the server burns 1.4 cores, which is
why machine I/O wait peaks at 1.4 percent: the stalls are many short
blocking reads, not long idle stretches. Perfect overlap could recover
at most that 15 percent share. Everything beyond it needs less CPU per
lookup, which is the batching story of 7.7. The sequential control
shows no stall at all: its CPU never dips during read peaks.
