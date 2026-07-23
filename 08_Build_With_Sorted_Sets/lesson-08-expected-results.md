# Lesson 8 Expected Results

## 1. First add

```redis
ZADD product:rank 4.5 BOWTIE42
```

Expected:

```text
1
```

One new member was added.

## 2. Variadic add

```redis
ZADD product:rank 4.8 BOLOTIE23 3.2 ASCOT13 4.9 BONDTIE007
```

Expected:

```text
3
```

Three new members were added.

## 3. Score lookup

```redis
ZSCORE product:rank BOWTIE42
```

Expected score:

```text
4.5
```

Redis may display floating-point values with slight formatting differences, but it should represent `4.5`.

## 4. Rank lookup

Sorted set order from low score to high score after the first four members:

```text
ASCOT13     3.2
BOWTIE42    4.5
BOLOTIE23   4.8
BONDTIE007  4.9
```

So:

```redis
ZRANK product:rank BOWTIE42
```

Expected:

```text
1
```

Ranks are zero-based.

## 5. Update a score

```redis
ZADD product:rank 2.0 BOWTIE42
```

Expected:

```text
0
```

No new member was added. An existing member's score was updated.

Now the order becomes:

```text
BOWTIE42    2.0
ASCOT13     3.2
BOLOTIE23   4.8
BONDTIE007  4.9
```

So:

```redis
ZSCORE product:rank BOWTIE42 -> 2.0
ZRANK  product:rank BOWTIE42 -> 0
```

## 6. Full range with scores

```redis
ZRANGE product:rank 0 -1 WITHSCORES
```

Expected members in ascending score order:

```text
BOWTIE42    2.0
ASCOT13     3.2
BOLOTIE23   4.8
BONDTIE007  4.9
```

## 7. Score range

```redis
ZRANGE product:rank 4 5 BYSCORE WITHSCORES
```

Expected:

```text
BOLOTIE23   4.8
BONDTIE007  4.9
```

These are the only two products with scores from `4` through `5`.

## 8. Cardinality

```redis
ZCARD product:rank
```

Expected:

```text
4
```

## 9. Increment a score

```redis
ZINCRBY product:rank 0.3 BOLOTIE23
```

If BOLOTIE23 was `4.8`, the new score becomes:

```text
5.1
```

## 10. Count in range

```redis
ZCOUNT product:rank 4 5
```

This returns how many members have scores between `4` and `5`, inclusive.

## 11. Remove one member

```redis
ZREM product:rank ASCOT13
```

Expected:

```text
1
```

If you run it again, Redis should return `0` because the member is already gone.

## Notes

- `ZADD` returns the number of **new members added**, not the number of members supplied.
- `ZRANK` counts from smallest score to largest score.
- `ZREVRANK` counts from largest score to smallest score.
- Members in a sorted set are unique.
- Scores can be integers or floating-point numbers.
