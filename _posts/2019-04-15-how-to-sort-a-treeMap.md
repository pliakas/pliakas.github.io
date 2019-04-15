---
layout: post
date: 2019-04-15 12:10:00
categories: coding java collections
author_name : Thomas Pliakas
author_url : /author/thomas
author_avatar: thomas
show_avatar : true
read_time : 20
feature_image: feature-laptop
show_related_posts: true
square_related: recommend-laptop
---
In order to sort a Map, with different ways ( by key, by value ) either using natural order or custom ordering, java provides a class java.util.TreeMap, which can sort a map in different ways. The above examples shows how to sort a map by key and by value. Furthermore it provides also an alternative method for java 8 using streams and lambdas.

``` java
    public final class CustomCollectors {
        private CustomCollectors() {}
        
        public static <T> Collector<T, ?, T[]> toArray( IntFunction<T[]> convert ) {
        
        return Collectors.collectingAndThen( Collectors.toList(),
                                             list -> list.toArray( convert.apply( list.size() ) ) );
    }
}
```
