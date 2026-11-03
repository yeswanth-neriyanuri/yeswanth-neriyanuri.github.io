---
title: "Java Streams"
description: "Quick reference for the Stream API: creating, transforming and collecting."
date: 2026-11-03
weight: 4
draft: false        # true keeps it off the live site
---

## What a stream is

A stream is a pipeline over a source (a collection, an array, a generator). It doesn't store data, and nothing runs until a **terminal operation** is called.

```
source  →  intermediate ops (lazy)  →  terminal op (runs it)
list       filter, map, sorted         collect, forEach, count
```

## Creating streams

```java
List<String> names = List.of("ana", "bob", "cy");

names.stream();                    // from a collection
Arrays.stream(new int[]{1, 2, 3}); // from an array
Stream.of("a", "b");               // from values
IntStream.range(0, 5);             // 0, 1, 2, 3, 4
```

## Common operations

| Operation | Kind | What it does |
|---|---|---|
| `filter(p)` | intermediate | keeps elements matching `p` |
| `map(f)` | intermediate | transforms each element |
| `sorted()` | intermediate | sorts (stateful) |
| `collect(c)` | terminal | gathers into a collection |
| `reduce(id, op)` | terminal | folds into one value |

```java
List<String> result = names.stream()
    .filter(n -> n.length() > 2)
    .map(String::toUpperCase)
    .toList();                     // [ANA, BOB]
```

## Collectors

```java
Map<Integer, List<String>> byLength =
    names.stream().collect(Collectors.groupingBy(String::length));
// {2=[cy], 3=[ana, bob]}
```

## flatMap

Use `flatMap` when each element turns into several elements and you want one flat stream.

```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3));
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .toList();                     // [1, 2, 3]
```

## Primitive streams

`IntStream`, `LongStream` and `DoubleStream` avoid boxing and add numeric helpers.

### Converting

```java
int total = names.stream().mapToInt(String::length).sum();   // Stream -> IntStream
List<Integer> boxed = IntStream.range(0, 3).boxed().toList(); // IntStream -> Stream
```

### Statistics

```java
IntSummaryStatistics s = IntStream.of(4, 8, 15).summaryStatistics();
s.getMin(); s.getMax(); s.getAverage(); // 4, 15, 9.0
```

## Short-circuiting and Optional

Some terminal operations stop early and return an `Optional`, because there may be no result.

```java
Optional<String> first = names.stream()
    .filter(n -> n.startsWith("b"))
    .findFirst();                  // Optional[bob]

boolean any = names.stream().anyMatch(n -> n.isEmpty()); // false
```

| Operation | Stops early | Returns |
|---|---|---|
| `findFirst()` / `findAny()` | yes | `Optional<T>` |
| `anyMatch` / `allMatch` / `noneMatch` | yes | `boolean` |
| `limit(n)` | yes | stream |
| `min` / `max` | no | `Optional<T>` |

## Parallel streams

`parallelStream()` splits work across the common ForkJoin pool.

### When it helps

- Large data sets with CPU-heavy, independent work per element.
- Sources that split cheaply, like `ArrayList` and arrays.

### When it hurts

- Small data sets, where the setup costs more than the work.
- Blocking I/O inside the pipeline, which starves the shared pool.
- Order-dependent operations like `findFirst` and `forEachOrdered`.

## Gotchas

- A stream can be consumed **once**. Reusing it throws `IllegalStateException`.
- Avoid side effects inside `map` and `filter`, especially with `parallelStream()`.
- `toList()` (Java 16+) returns an unmodifiable list; `collect(Collectors.toList())` doesn't promise either way.

## Interview questions

1. **Intermediate vs terminal operations?** Intermediate ones are lazy and return a stream; a terminal one triggers the pipeline and returns a result.
2. **`map` vs `flatMap`?** `map` gives one output per input; `flatMap` flattens each result stream into one stream.

## References

- [java.util.stream package docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html)
