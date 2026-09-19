# Sequential device speed (conclusion)

Run `wikidata-ssd-sequential`, Ural seq 3940, 2026-09-19, exit 0.
Device `/dev/md1`, RAID0 of two Samsung SSD 990 PRO 4TB.
Driver: `scripts/ssd-sequential-read.sh`.

## Method

Three O_DIRECT reads of 8 GiB from an index file on the data
filesystem. Direct I/O bypasses the page cache, so the numbers reflect
the device, and nothing is polluted or cleared.

## Numbers

Three reps agree: 6.1 to 6.2 GB/s sequential.

## Interpretation

The scattered export peaks near 0.55 GB/s, which is 9 percent of the
device sequential maximum. The disk is not the limit. The gap between
observed and sequential speed is the cost of small scattered reads,
which is exactly what batching and bulk submission address.
