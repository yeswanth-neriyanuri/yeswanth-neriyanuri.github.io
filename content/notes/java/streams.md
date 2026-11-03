---
title: "Java Streams"
description: "Quick reference for the Stream API: creating, transforming and collecting."
date: 2026-11-03
draft: false        # true keeps it off the live site
ShowToc: true       # table of contents from the ## headings
TocOpen: false
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

## Gotchas

- A stream can be consumed **once**. Reusing it throws `IllegalStateException`.
- Avoid side effects inside `map` and `filter`, especially with `parallelStream()`.
- `toList()` (Java 16+) returns an unmodifiable list; `collect(Collectors.toList())` doesn't promise either way.

## Interview questions

1. **Intermediate vs terminal operations?** Intermediate ones are lazy and return a stream; a terminal one triggers the pipeline and returns a result.
2. **`map` vs `flatMap`?** `map` gives one output per input; `flatMap` flattens each result stream into one stream.

## References

- [java.util.stream package docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html)
