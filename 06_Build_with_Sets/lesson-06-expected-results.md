# Lesson 6 Expected Results

## Initial `SADD`

```redis
SADD product:views:bowtie42 alice bob chuck dave
```

Expected result:

```text
4
```

Four new members were added.

## Adding duplicates

```redis
SADD product:views:bowtie42 alice bob
```

Expected:

```text
0
```

Both members already exist.

```redis
SADD product:views:bowtie42 eve alice
```

Expected:

```text
1
```

Only `eve` is new.

## `SMEMBERS`

The set contains:

```text
alice
bob
chuck
dave
eve
```

Redis sets are unordered, so the display order may differ.

## `SCARD`

Expected:

```text
5
```

## Membership

```text
SISMEMBER ... alice -> 1
SISMEMBER ... frank -> 0
```

`SMISMEMBER` returns one integer per requested member.

## Removing Chuck and Dave

```redis
SREM product:views:bowtie42 chuck dave
```

Expected:

```text
2
```

The remaining members are:

```text
alice
bob
eve
```

Their display order is not guaranteed.

Running the same `SREM` again returns:

```text
0
```

## Set operations

Given:

```text
users:java   = alice, bob, dave
users:spring = bob, chuck, dave
```

Union:

```text
alice, bob, chuck, dave
```

Intersection:

```text
bob, dave
```

Java minus Spring:

```text
alice
```
