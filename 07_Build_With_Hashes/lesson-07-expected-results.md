# Lesson 7 Expected Results

## First `HSET`

```redis
HSET product:bowtie42 name "Awesome Bowtie"
```

Expected:

```text
1
```

One new field was created.

## Complete product record

```redis
HSET product:bowtie42   sku BOWTIE42   name "Awesome Bowtie"   color red   description "This awesome bowtie is the same color of red as the Redis logo."   quantity 23
```

Expected:

```text
4
```

Five field/value pairs were supplied, but `name` already existed. Redis created four new fields and updated the existing `name` field.

## Product hash contents

The hash contains these fields:

```text
sku         -> BOWTIE42
name        -> Awesome Bowtie
color       -> red
description -> This awesome bowtie is the same color of red as the Redis logo.
quantity    -> 23
```

`HGETALL` may display fields in a different order. Hash fields do not have a guaranteed presentation order.

## `HLEN`

Expected:

```text
5
```

## `HEXISTS`

```text
HEXISTS product:bowtie42 sku   -> 1
HEXISTS product:bowtie42 price -> 0
```

## Updating `color`

```redis
HSET product:bowtie42 color crimson
```

Expected:

```text
0
```

No new field was created; the existing field was updated.

## Decrementing quantity

```redis
HINCRBY product:bowtie42 quantity -1
```

Expected:

```text
22
```

## Decimal price

```text
HSET price 24.99       -> 1
HINCRBYFLOAT price 5.00 -> 29.99
```

The exact textual decimal representation is determined by Redis, but it should represent `29.99`.

## `HSETNX`

```text
First HSETNX createdBy lesson-07 -> 1
Second HSETNX createdBy ...      -> 0
HGET createdBy                   -> lesson-07
```

## Removing `color`

```text
HDEL product:bowtie42 color -> 1
HEXISTS ... color           -> 0
HDEL ... color again        -> 0
```

## Expiration

```text
EXPIRE product:bowtie42 3600 -> 1
TTL product:bowtie42          -> a value close to 3600
PERSIST product:bowtie42      -> 1
TTL product:bowtie42          -> -1
```

`-1` means the key exists but has no expiration.
