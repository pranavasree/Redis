# Lesson 12 Expected Results

This lesson combines several Redis use cases. Exact response formatting can differ between Redis Insight, `redis-cli`, and programming-language clients.

## 1. Enterprise caching

Before the cached value exists:

```redis
GET cache:product:42
```

Expected:

```text
(nil)
```

This is a cache miss.

After:

```redis
SET cache:product:42 "..." EX 300
```

Expected:

```text
OK
```

A following `GET` returns the cached JSON string, and `TTL` returns a positive number up to approximately `300`.

After invalidation:

```redis
UNLINK cache:product:42
GET cache:product:42
```

Expected:

```text
1
(nil)
```

The next application request should reload fresh data from the source-of-truth database.

## 2. Cache lock demonstration

```redis
SET lock:cache:product:42 worker-1 NX PX 5000
```

Expected:

```text
OK
```

A second `SET ... NX` while the key exists should return:

```text
(nil)
```

This shows that only one worker acquired the temporary key.

The lab uses `UNLINK` only for demonstration. A real distributed-lock implementation must verify that the process releasing the lock still owns it.

## 3. Search and query

If Redis Search is supported, `COMMAND INFO FT.CREATE` and `COMMAND INFO FT.SEARCH` return command metadata.

The three `HSET` operations each create a hash document.

```redis
FT.CREATE idx:catalog ...
```

Expected:

```text
OK
```

### Text search

```redis
FT.SEARCH idx:catalog 'bow tie'
```

Expected total:

```text
3
```

All three product names contain “Bow Tie.”

### Identifier search

```redis
FT.SEARCH idx:catalog '@sku:{291059}'
```

Expected matching document:

```text
catalog:product:1
```

### Price range

```redis
FT.SEARCH idx:catalog '@price:[10 30]' SORTBY price ASC
```

Expected products:

```text
catalog:product:1 -> 25
catalog:product:2 -> 30
```

### Chicago search

```redis
FT.SEARCH idx:catalog '@city:{Chicago}'
```

Expected:

```text
catalog:product:1
catalog:product:2
```

### Combined search

The combined red, polka-dot, price, and Chicago query should match:

```text
catalog:product:1
```

If `FT.CREATE` is unknown, the selected Redis deployment does not expose Redis Search commands.

## 4. Session management

After `HSET` and `EXPIRE`, `HGETALL` should contain fields similar to:

```text
userId         -> user-101
cartItemCount  -> 3
cartItems      -> ["product:1","product:1","product:1"]
lastPage       -> /bowties/red
```

`TTL` returns a positive value up to approximately `1800`.

After:

```redis
HINCRBY session:shop:abc123 cartItemCount 1
```

Expected:

```text
4
```

After updating `lastPage`, it should return:

```text
/checkout
```

Refreshing with `EXPIRE ... 1800` gives the active session another 30 minutes.

## 5. Recommendation baseline

```redis
ZREVRANGE recommendations:user:101 0 2 WITHSCORES
```

Expected order from highest score to lowest:

```text
catalog:product:2 -> 0.98
catalog:product:1 -> 0.84
catalog:product:3 -> 0.71
```

This is a deterministic score-based recommendation list. Vector search can later generate or enhance these scores.

## 6. Vector-search outline

There is no exact runnable result because the lab intentionally uses placeholders for binary vectors.

A production application must:

1. Generate an embedding using a model.
2. Ensure the vector dimension matches the index.
3. Convert the vector to the format expected by the chosen Redis document type and client.
4. Send the query vector as a parameter.
5. Interpret the returned distance or similarity score.

## 7. Combined shopping flow

At the end of the lab:

- `GET cache:product:42` returns the cached product.
- The search query returns Chicago products under the chosen price.
- `HGETALL session:shop:abc123` returns the user's cart/session.
- `ZREVRANGE recommendations:user:101 ...` returns ranked recommendations.

Together, these demonstrate how one online-shopping application can use several Redis capabilities without forcing every problem into one data structure.
