# afl-kit

## afl-cmin.py
Reimplement afl-cmin in python. Use less memory, less disk space, and faster.

### Features/enhancement
 - Support the same command line flags as original afl-cmin.
 - dedup by hash of file content in the beginning.
 - `-i DIR` can be specified multiple times. Also support globbing.
 - `-w WORKERS` to specify number of workers.
 - `--as_queue` output filename like `id:000001,hash:value`.

So, you can use afl-cmin.py in workflow like this

1. Run many instances of afl-fuzz and have multiple queues in `sync_dir.1` directory
2. `afl-cmin.py -i 'sync_dir.1/*/queue' -o sync_dir.2/prev/queue --as_queue ...`
3. Run another batch of afl-fuzz in `sync_dir.2`. They will automatically sync queue from `sync_dir.2/prev/queue`.

### Non-scientific performance test: 

program     | worker | temp disk (mb) | memory | time (min)
----------- | ------ | -------------- | ------ | ----------
afl-cmin    |     1  |      9782      | 7.8gb  | 27
[afl-pcmin] |     8  |      9762      | 7.8gb  | 13.8
afl-cmin.py |     1  |       359      | <50mb  | 11.9
afl-cmin.py |     8  |      1136      | <250mb | 1.8

[afl-pcmin]: https://github.com/bnagy/afl-trivia

Detail of this table
- the input are 79k unique files, total 472mb. the output are 5k files, total 39mb.
- `temp disk` is the size of `.traces` folder after run with `AFL_KEEP_TRACES=1`.

## tmin.py
Similar to afl-tmin, but minimize by different conditions.

### Features/enhancement
 - Support similar command line flags as afl-tmin.
 - Use similar minimization heuristic as afl-tmin.
 - Instead of classifying input by coverage, tmin.py classifies input by
   program output and terminal conditions. Supportted conditions:
     * `--stdout`: stdout contains given string
     * `--stderr`: stderr contains given string
     * `--crash`: program terminated by any signal
     * `--returncode`: program exits with given returncode
     * `--signal`: program terminated by given signal
     * `--timeout`: program terminated due to timeout

### Examples
 - Minimize input while makes sure [exploitable] still output `EXPLOITABLE`.

    `tmin.py -i file.in -o file.out -m none --stdout "'EXPLOITABLE'" -- ~/src/exploitable/triage.py './w3m -T text/html -dump @@'`

 - Minimize input while makes sure the program is still killed by SIGABRT (i.e. assert() fail)

    `tmin.py -i file.in -o file.out --signal 6 -- /path/to/program @@`

[exploitable]: https://github.com/jfoote/exploitable

## License
Apache License 2.0. Copyright 2016 Google Inc.

This is not an official Google product.

