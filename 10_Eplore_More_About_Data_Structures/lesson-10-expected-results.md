# Lesson 10 Expected Results

This lesson is an overview of several advanced structures. Exact formatting may differ between Redis Insight, `redis-cli`, and client libraries.

## HyperLogLog

After adding:

```text
day 1: alice, bob, chuck
day 2: bob, dave, emma
```

the approximate counts should be close to:

```text
PFCOUNT visitors:day1 -> 3
PFCOUNT visitors:day2 -> 3
PFCOUNT visitors:all  -> 5
```

HyperLogLog is approximate. Its purpose is to estimate unique counts using very little memory, not to return the original members.

`PFADD` returns `1` when the HyperLogLog's internal representation changed and `0` when it did not. It does not return the number of elements supplied.

## Bloom filter

```text
BF.EXISTS ... alice -> 1
BF.EXISTS ... linda -> 0
```

Interpretation:

```text
0 -> definitely not present
1 -> probably present
```

A Bloom filter can return a false positive, but it does not return a false negative for items successfully added.

Adding `alice` again may return `0` because the filter reports that the item was probably already present.

## Streams

Each `XADD ... *` returns an entry ID similar to:

```text
1721680000000-0
```

The exact IDs are generated at runtime and will differ.

```text
XLEN clickstream:bowtie -> 3
```

`XRANGE` returns entries from oldest to newest.  
`XREVRANGE ... COUNT 2` returns the newest two entries first.

## Geospatial index

`GEOPOS` returns stored coordinates. Redis may return slightly adjusted coordinates because locations are encoded internally.

The distance between New York City and Fort Lauderdale is approximately 1,700 kilometers; the exact Redis result may vary slightly.

A 2,000-kilometer `GEOSEARCH` centered on New York City should include:

```text
new-york-city
ft-lauderdale
```

and should not include Denver.

## Bitmaps

After setting user offsets `101`, `205`, and `999`:

```text
GETBIT ... 101 -> 1
GETBIT ... 500 -> 0
BITCOUNT day1  -> 3
```

The OR operation across both days represents users active on at least one day:

```text
101, 205, 500, 999
```

so:

```text
BITCOUNT active:users:two-days -> 4
```

## Bitfields

The first command stores:

```text
price  -> 1000
owners -> 0
```

Reading the two unsigned 16-bit fields returns:

```text
1000
0
```

Incrementing the owner count returns `1`.

With `OVERFLOW SAT`, increasing a `u16` price beyond its maximum saturates at:

```text
65535
```

rather than wrapping around.

## Cuckoo filter

Expected membership behavior:

```text
event-101 before deletion -> 1
event-999                 -> 0
event-101 after deletion  -> 0
```

Cuckoo filters support deletion, unlike standard Bloom filters.

## Count-Min Sketch

After the increments:

```text
click    -> approximately 10
purchase -> approximately 2
search   -> approximately 5
missing  -> approximately 0
```

Count-Min Sketch estimates frequencies and can over-count due to hash collisions.

## Top-K

The three most frequent terms should include:

```text
redis
kafka
```

The third position depends on the algorithm state and supplied stream; `spring` or `rabbitmq` may appear depending on frequency/tie behavior.

Top-K is approximate and designed for finding frequent items in streams.

## t-digest

Given the values:

```text
10, 12, 15, 18, 20, 25, 30, 40, 80, 120
```

the median should be around the middle of the distribution, and the 95th percentile should be near the upper end. Both are estimates.

```text
TDIGEST.MIN -> 10
TDIGEST.MAX -> 120
```


## Time series

After adding three samples:

```text
timestamp 1000 -> 21.5
timestamp 2000 -> 22.1
timestamp 3000 -> 23.0
```

`TS.GET temperature:store-1` returns the latest sample:

```text
3000
23.0
```

`TS.RANGE temperature:store-1 - +` returns all samples in chronological order.

Time series commands require support in the selected Redis deployment.

## Vector search

`COMMAND INFO FT.CREATE` and `COMMAND INFO FT.SEARCH` confirm whether search commands are available.

The vector example contains placeholders because an embedding is a binary array of floating-point numbers. A Java, Python, JavaScript, or other client normally converts a numeric vector into the required binary representation before storing or querying it.
