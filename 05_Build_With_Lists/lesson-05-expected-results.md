# Lesson 5 Expected Results

## Alice after three LPUSH commands

```text
Index 0 -> ASCOT13
Index 1 -> BOLOTIE23
Index 2 -> BOWTIE42
```

## Alice RPOP order

```text
BOWTIE42
BOLOTIE23
ASCOT13
(nil)
```

`LPUSH` plus `RPOP` behaves as a FIFO queue.

## Bob after five LPUSH commands

```text
Index 0  / -5 -> ASCOT12
Index 1  / -4 -> BOLOTIE52
Index 2  / -3 -> BOWTIE13BLU
Index 3  / -2 -> BOWTIE23GRN
Index 4  / -1 -> BOWTIE42RED
```

## Expected ranges

### `LRANGE products:recent:bob 0 2`

```text
ASCOT12
BOLOTIE52
BOWTIE13BLU
```

### `LRANGE products:recent:bob 1 3`

```text
BOLOTIE52
BOWTIE13BLU
BOWTIE23GRN
```

### `LRANGE products:recent:bob 2 5`

```text
BOWTIE13BLU
BOWTIE23GRN
BOWTIE42RED
```

Redis safely stops at the end of the list even though index `5` is outside this five-item list.

### `LRANGE products:recent:bob -3 -1`

```text
BOWTIE13BLU
BOWTIE23GRN
BOWTIE42RED
```

### `LINDEX`

```text
LINDEX products:recent:bob 0  -> ASCOT12
LINDEX products:recent:bob -1 -> BOWTIE42RED
```
