# Lesson 9 Expected Results

## `JSON.SET` root document

```redis
JSON.SET product:bowtie42 $ '{ ... }'
```

Expected:

```text
OK
```

The root path `$` represents the complete JSON document.

## `JSON.GET product:bowtie42`

Returns the complete JSON document in serialized JSON form. Depending on the client, quotes may appear escaped because Redis is returning a JSON-encoded string representation.

## `JSON.GET product:bowtie42 $`

Using a `$` JSONPath returns matching values in a top-level JSON array. The result conceptually contains one root document.

## Quantity path

```redis
JSON.GET product:bowtie42 $.quantity
```

Expected JSONPath-style result:

```json
[23]
```

The result is an array because a JSONPath expression can match one or more values.

## Root-level wildcard

```redis
JSON.GET product:bowtie42 $.*
```

Returns an array containing the values of all root-level properties. The values may include strings, arrays, numbers, and booleans.

## Typed updates

```redis
JSON.SET product:bowtie42 $.onsale true
```

Expected:

```text
OK
```

The stored value is a JSON boolean, not the string `"true"`.

```redis
JSON.SET product:bowtie42 $.price 9.99
```

Expected:

```text
OK
```

The stored value is a JSON number.

```redis
JSON.SET product:bowtie42 $.name '"Striped Bowtie"'
```

Expected:

```text
OK
```

The inner double quotes make the value a valid JSON string.

## Numeric increment

```redis
JSON.NUMINCRBY product:bowtie42 $.quantity 2
```

If the original quantity was `23`, the new result represents:

```json
[25]
```

## Array append

```redis
JSON.ARRAPPEND product:bowtie42 $.colors '"yellow"'
```

If the array initially contained three colors, the returned array-length result represents `4`.

## Toggle

```redis
JSON.TOGGLE product:bowtie42 $.onsale
```

The result contains the new boolean state for each matched path.

## Remove `colors`

```redis
JSON.DEL product:bowtie42 $.colors
```

Expected:

```text
1
```

One matching path was removed.

Running it again returns:

```text
0
```

because the path no longer exists.

## Nested object

```redis
JSON.SET product:bowtie42 $.inventory '{ "warehouse": "OH-1", "available": true }'
```

Expected:

```text
OK
```

Then:

```redis
JSON.GET product:bowtie42 $.inventory.warehouse
```

returns a JSONPath-style array containing:

```json
["OH-1"]
```

## Key expiration

```text
EXPIRE -> 1
TTL    -> a value close to 3600
PERSIST -> 1
```

After `PERSIST`, `TTL` returns `-1`, meaning the key exists without expiration.

## Common interpretation rule

With `$`-based JSONPath expressions, Redis JSON commands often return arrays because the path can match multiple values. Exact RESP formatting can differ between Redis Insight, redis-cli, and client libraries.
