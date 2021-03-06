---
layout: post
date: 2019-04-21 12:00:00
categories: coding java jdk11
author_name : Thomas Pliakas
author_url : /author/thomas
author_avatar: thomas
show_avatar : true
read_time : 10
feature_image: feature-laptop
show_related_posts: true
square_related: recommend-laptop
---
One of 90 features that have been introduced in JDK 11, is the support of the local variables as lambda parameters. This features was described in [JEP-323: Local-Variables-Syntac for Lambda Parameters](https://openjdk.java.net/jeps/323) and can be considered as an extension of [JEP-286: Local-Variable Type Inference](http://openjdk.java.net/jeps/286). Now, the developes does not longer need to explicitly state the type of a local-variable but instead, use `var` as depicted in the code exceprt below, since it extends the use of this syntax to the parameters of Lambda expressions. 

``` java
    List.of( "alfa", "bravo", "charlie", "delta", "echo", "foxtrot" ).stream()
            .filter( word -> ( word.length() & 1) == 1  )
            .map( (var word) -> word.toUpperCase() )
            .collect( toList() ).forEach( System.out::println );
```
This enhancement can be used when someone wants to add an annotation to the Lambda parameter. It was not possible to do this without a type being declared until the introduction of this JEP. To avoid having to use the explicit type we can use var to simplify things, as shown below:

``` java
List.of( "alfa", "bravo", "charlie", "delta", "echo", "foxtrot" )
    .stream()
    .filter( word -> ( word.length() & 1) == 1  )
    .map( ( @NotNull  var word ) -> word.toUpperCase() )
    .collect( toList() ).forEach( System.out::println );
```
