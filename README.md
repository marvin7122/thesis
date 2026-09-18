# Thesis artifacts

Public artifact store for the bachelor thesis on the QLever export path.

Each directory mirrors the run path in the (private) thesis repository and
freezes the exact inputs a chapter claim depends on: index build settings,
index build metadata, driver scripts, and query files. Files are
byte-identical copies of the artifacts used on the benchmark machine.

## Layout

`experiments/runs/<run>/<files>` mirrors the thesis repository path, so a
footnote link stays traceable to the run that produced the numbers.

## Runs

- `experiments/runs/wikidata-iouring-pre-baseline/`: served index
  configuration for the pre-`io_uring` export baseline (Chapter 7).
  `wikidata.settings.json` is the index build configuration.
  `wikidata.meta-data.json` records the builder commit, the build date,
  and the vocabulary type.
