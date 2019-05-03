---
layout: post
date: 2019-04-21 12:00:00
categories: coding java junit5
author_name : Thomas Pliakas
author_url : /author/thomas
author_avatar: thomas
show_avatar : true
read_time : 15
feature_image: feature-laptop
show_related_posts: true
square_related: recommend-laptop
---

Junit5 team introduced a new artifact named `junit-platform-testkit`, since 1.4 version, that provides an API in order to support the execution and results verification of a pre-defined test-plan on the JUNIT platform. The main package of the Engine Test kit is `org.junit.platform.testkit.engine` and contains all needed APIs. The main class is the `EngineTestKit` that provides two tatic factory methods: 
  * `engine()` - Creates an execution `EngineTestKit.Builder` for the TestEngine. 
  * `execute()` - Execute tests for the given "EngineDiscoveryRequest" using the TestEngine with the supplied id or `TestEgnine`. 
 
 
## Verify Statistics and Events during TestPlan execution
Using TestKit, you can assert statistics agains the events fired during the executuon of a defined TestPlan. You can "assert" statistics not only for the *tests* but also for *containers* in the TestEngine. The key statistics that are available: 
  * started()
  * succeeded()
  * failed()
  * finished()
  * aborded()
  
More details about availabe statistics can be found in [EventStatistics](https://junit.org/junit5/docs/current/api/org/junit/platform/testkit/engine/EventStatistics.html "EventStatistics JavaDoc")
