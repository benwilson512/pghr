# Pghr

Quick 'n' dirty test to see how Ecto + Postgres behave in specific microbenchmarks.

## Test Methodology


### For Ecto + Benchee

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && mix run bench/create_item_fast.exs 
Deleting all existing items ...
Starting test ...
Operating System: macOS
CPU Information: Intel(R) Core(TM) i7-4980HQ CPU @ 2.80GHz
Number of Available Cores: 8
Available memory: 16 GB
Elixir 1.9.1
Erlang 22.0.7

Benchmark suite executing with the following configuration:
warmup: 2 s
time: 10 s
memory time: 0 ns
parallel: 5
inputs: none specified
Estimated total run time: 12 s

Benchmarking create item...

Name                  ips        average  deviation         median         99th %
create item        1.36 K      734.57 μs   ±702.99%         422 μs         635 μs
```

### For pgbench

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && pgbench -f pgbench/create_item_fast.sql -n -c 10 -j 5 -T 10 pghr
transaction type: pgbench/create_item_fast.sql
scaling factor: 1
query mode: simple
number of clients: 10
number of threads: 5
duration: 10 s
number of transactions actually processed: 206051
latency average = 0.487 ms
tps = 20515.651758 (including connections establishing)
tps = 20523.650252 (excluding connections establishing)
```

## Results

The benchmark results described below were run on my 2015-era MacBook Pro. Tests were always run for 10 seconds with a 2-second warm-up. Varying levels of parallelism were used and noted in the test results.

~~I want to emphasize a performance correlation I've observed on update with size of the existing table, so I'm removing previous results. All results here are run with 10 parallel clients and a pool size of 40, which had been shown in previous tests to be among the "best" configurations.~~

With `Enum.random/1` removed from the test, this claim is disproven. Still seeing a constant difference between `pgbench` and Ecto, but there are leads to follow.

### Create Item Benchmark

Ecto:

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && mix run bench/create_item_fast.exs 
```

Pgbench:

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && pgbench -f pgbench/create_item_fast.sql -n -c 40 -j 10 -T 10 pghr
```

   ips | duration | Comments
------:|---------:|:---
 14864 |       10 | Ecto
 14794 |       30 |
 14521 |      100 |
 28058 |       10 | pgbench
 27231 |       30 |
 26002 |      100 |
 
### Update Item Benchmark (Using Raw SQL Update)

Ecto:

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && mix run bench/update_item_sql.exs
```

Pgbench:

```
$ mix ecto.drop && mix ecto.create && mix ecto.migrate && pgbench -f pgbench/update_item_fast.sql -n -c 40 -j 10 -T 30 pghr
```

   ips | seed size | duration | clients | Comments
------:|----------:|---------:|--------:|:---
 23482 |       500 |       30 |      10 | Ecto
 22443 |      5000 |       30 |      10 |
 23407 |      5000 |       30 |      20 |
 26075 |      5000 |       30 |      30 |
 29035 |      5000 |       30 |      40 |
 30618 |      5000 |       30 |      50 | (pool size: 50)
 31043 |      5000 |       30 |      60 | (pool size: 60)
 31408 |      5000 |       30 |      70 | (pool size: 70)
 32745 |      5000 |       30 |      80 | (pool size: 80)
 32175 |      5000 |       30 |      90 | (pool size: 90)
 30147 |      5000 |       30 |     100 | (pool size: 100)
 22168 |     50000 |       30 |      10 |
 21188 |    500000 |       30 |      10 |
 46024 |       500 |       30 |      10 | pgbench
 47234 |      5000 |       30 |      10 |
 47294 |      5000 |       30 |      20 |
 48151 |      5000 |       30 |      30 |
 47264 |      5000 |       30 |      40 |
 47304 |     50000 |       30 |      10 |
 46712 |    500000 |       30 |      10 |
     – |         – |        – |       – | –
 45590 |      5000 |       30 |      40 | pgbench (add notification trigger PR #16)
 10698 |      5000 |       30 |      40 | Ecto
