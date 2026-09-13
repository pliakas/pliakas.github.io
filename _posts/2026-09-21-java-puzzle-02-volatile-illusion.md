---
layout: post
title: "Java Puzzle #2: The volatile Illusion"
comments: true
tags: [java-puzzles, java, concurrency]
---

Puzzle #2 in the Java Puzzles series. Last week's puzzle showed `counter++`
isn't thread-safe. So we "fixed" it the way everyone fixes it the first
time: slapped `volatile` on the field and moved on.

Nastiness: `P3` — Slack message, 👀 reacts.

**What gets printed now?**

- A) `200000` — volatile made it thread-safe
- B) `100000` — volatile serializes every write onto one thread
- C) Some unpredictable number less than 200000 (varies each run)
- D) Compile error — you can't apply `++` to a volatile field

```java
public class P02_VolatileIllusion {

    private static volatile int counter = 0;

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

`volatile` guarantees visibility — every thread sees the most recent write
— and stops the compiler/CPU from reordering instructions around that
field. It says nothing about atomicity. `counter++` is still three separate
bytecode steps (`GETFIELD`, `IADD`, `PUTFIELD`); volatile just makes each of
those three steps instantly visible to other threads, which does nothing to
stop two threads from reading the same value in the gap before either one
writes back. Same lost updates as puzzle 1 — just with a keyword that made
you feel better about it.

> 💬 I added volatile, watched the number come out wrong anyway, and
> understood real betrayal for the first time.
