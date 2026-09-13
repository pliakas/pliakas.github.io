---
layout: post
title: "Java Puzzle #1: The Overly Confident Counter"
comments: true
tags: [java-puzzles, java, concurrency]
---

Kicking off a new series here: **Java Puzzles**. Every week I'll drop one of
the puzzles from my [conference talk repo](https://github.com/pliakas/java-puzzles)
here — a single self-contained `.java` file with a surprising runtime
behavior. Guess the answer, run it yourself, then check the solution below.

Nastiness: `P4` — warm-up, everyone survives.

**What gets printed?**

- A) `200000`
- B) `100000`
- C) Some unpredictable number less than 200000 (varies each run)
- D) Compile error

```java
public class P01_LostUpdate {

    private static int counter = 0;

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            for (int i = 0; i < 100_000; i++) {
                counter++;
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println(counter);
    }
}
```

## Solution

**Answer: C**

`counter++` is read-modify-write — three separate bytecode steps (`GETFIELD`,
`IADD`, `PUTFIELD`). Two threads can both read the same value before either
writes back, silently dropping increments. No crash, no warning — just a
number that's wrong differently every single run.

> 💬 This code doesn't have a bug. It has a MOOD. Run it ten times, get ten
> different feelings about your career choices.
