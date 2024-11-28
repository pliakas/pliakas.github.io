---
layout: post
title: JDK12 - The Collector::teeing() collector explained
comments: true
tags: [java]
---

In JDK12, a new feature (out for 32) in the Stream API has been introduced and is a new collector, called `teeing()` which is provided by the Collectors utility class. The `teeing()` collector takes takes three arguments, two Collectors and a BiFunction for merging the results from the two collectors. The full story behind this new feature can be found [here](https://bugs.openjdk.java.net/browse/JDK-8209685 "teeing() collector history").

The `teeing()` collector takes three arguments, as a typical collector, as shown in the method signature below:

~~~java
public static <T, R1, R2, R>
    Collector<T, ?, R> teeing(Collector<? super T, ?, R1> downstream1,
                              Collector<? super T, ?, R2> downstream2,
                              BiFunction<? super R1, ? super R2, R> merger)
~~~
More information about `Collector::teeing()` can be found [here](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/stream/Collectors.html "Collectors::teeing javadoc").

The two collecctors are operating in the same stream of type `T` and produce different results `R1`and `R2`. The the merger function using these two results to produce a final result type of `R` using BiFunction functional interface. This new feature provide the posibility collect element using two collectors (even re-using existing collectors) and produce result out of these two collectors' results.

A `Collectors::teeing()` is a **Concurrent** and/or **Unordered**, if both collectors are and does not has any **Identity_Finish** since always must merge teh resultgs of the used Collectors. 

## Example 1 ###
Assuming that you want to collect the longest and shorted words out of a stream of words.

~~~java
    @Test
    @Order( 2 )
    @EnabledOnJre( JRE.JAVA_12 )
    @DisplayName( "Find the longest and the shortest line out of  a List of strings using Collectors::teeing from Java 12" )
    void calculatesMinMaxLines_New()
    {
        List<String> input = List.of(
            "alfa", "bravo", "charlie", "delta", "echo", "foxtrot", "golf", "hotel");

        int max = input.stream().mapToInt( String::length ).max().orElse( -1 );
        int min = input.stream().mapToInt( String::length ).max().orElse( -1 );

        var result = input.stream().collect( teeing(
            Collectors.filtering( s -> s.length() == max, toList() ),
            Collectors.filtering( s -> s.length() ==  min, toList() ),
            Pair::of
        ) );

        assertAll( "Longest and Shortest Words",
            () -> assertEquals( List.of( "charlie", "foxtrot"), result.getLeft() ),
            () -> assertEquals( List.of( "alfa", "echo", "golf"), result.getRight() ) );
    }
~~~

## Example 2 ###
Another example is if you want to provide some statistics about Stream of integers. The we can easiliy calculate the average of a stream of integers using the above collector. 

~~~java
    @Test
    @Order( 3 )
    @EnabledOnJre( JRE.JAVA_12 )
    @DisplayName( "Calculate the average of Stream of integers" )
    void calculateAverageOfIntegers()
    {
        List<Integer> input = List.of( 1, 4, 2 ,7, 4, 6, 5);

        var result = input.stream().collect(
            teeing(
                summingDouble( number -> number ),
                counting(),
                ( sum, count ) -> sum / count
            )
        );

        assertEquals( 4.142857142857143, result );
    }
~~~

You can find additional example for using the `Collector::teeing()`at my github repo [here](https://github.com/pliakas/roukou-blog). 
