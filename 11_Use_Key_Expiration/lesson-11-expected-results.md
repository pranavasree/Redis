# Lesson 11 Expected Results

## `EXPIRE`

```redis
SET bowtie:color red
EXPIRE bowtie:color 60
```

Expected:

```text
OK
1
```

`EXPIRE` returns:

```text
1 -> expiration was set
0 -> expiration was not set
```

A return value of `0` can mean the key does not exist or a conditional option prevented the update.

## `TTL`

Immediately after setting a 60-second expiration:

```redis
TTL bowtie:color
```

Expected:

```text
A value from 0 through 60
```

The number decreases as time passes.

When the key expires:

```redis
GET bowtie:color
```

Expected:

```text
(nil)
```

Then:

```redis
TTL bowtie:color
```

Expected:

```text
-2
```

## TTL special return values

```text
-1 -> key exists but has no expiration
-2 -> key does not exist
```

Example:

```redis
SET bowtie:pattern striped
TTL bowtie:pattern
```

Expected:

```text
-1
```

## `EXPIREAT` and `EXPIRETIME`

```redis
EXPIREAT bowtie:pattern 4481067600
EXPIRETIME bowtie:pattern
```

Expected:

```text
1
4481067600
```

`4481067600` represents `2112-01-01 05:00:00 UTC`.

`EXPIREAT` sets an absolute Unix timestamp in seconds.  
`EXPIRETIME` returns that absolute expiration timestamp.

## `PERSIST`

```redis
PERSIST bowtie:pattern
```

Expected:

```text
1
```

The expiration was removed.

Then:

```redis
TTL bowtie:pattern
EXPIRETIME bowtie:pattern
```

Expected:

```text
-1
-1
```

The key exists and is persistent.

Running `PERSIST` again would return `0` because the key no longer has an expiration.

## Atomic `SET` expiration

```redis
SET session:abc123 "user-101" EX 60
```

Expected:

```text
OK
```

The key is created with a TTL in one command.

```redis
SET cache:product:42 "..." PX 5000
PTTL cache:product:42
```

Expected `PTTL`:

```text
A value from 0 through 5000
```

## Conditional expiration

### `NX`

```text
First EXPIRE ... NX  -> 1
Second EXPIRE ... NX -> 0
```

The first succeeds because the key had no expiration.  
The second fails because the key already has one.

### `XX`

```text
EXPIRE ... XX -> 1
```

It succeeds because the key already has an expiration.

### `GT`

The command succeeds only when the new TTL is greater than the current TTL.

### `LT`

The command succeeds only when the new TTL is less than the current TTL.

## Replacing a value

```redis
SET lesson:replace first EX 120
SET lesson:replace second
TTL lesson:replace
```

Expected:

```text
-1
```

A normal `SET` replaces the key value and clears the existing TTL.

With:

```redis
SET lesson:replace third KEEPTTL
```

the existing expiration is preserved.

## Updating without replacing the key

Commands such as these normally preserve the existing key TTL:

```text
INCR
HSET
LPUSH
```

The lab should show a positive TTL after each update.

## `GETEX`

```redis
GETEX token:reset EX 60
```

Returns the current string value and sets a 60-second expiration.

```redis
GETEX token:reset PERSIST
```

Returns the value and removes the expiration.

After `PERSIST` behavior:

```redis
TTL token:reset
```

Expected:

```text
-1
```

## Short expiration demo

```redis
SET demo:short "I disappear soon" EX 5
```

After waiting around six seconds:

```redis
GET demo:short
TTL demo:short
```

Expected:

```text
(nil)
-2
```

## Timing note

TTL values are time-sensitive. A result such as `59`, `58`, or `57` after setting a 60-second expiration is correct.
