---
layout: post
title: Gradle - How to display junit5 test results in console
comments: true
tags: [gradle]
---

When I first use [gradle](http://gradle.org)` as the main build mechanism for a java based project (including also some kotlin code), I realised that the unit test report (based on [junit5](https://junit.org/junit5/)) was different compared to a maven report using the [surefire plugin](https://maven.apache.org/surefire/maven-surefire-plugin/). 

By executing the command `./gradlew test` the output was:

~~~
BUILD SUCCESSFUL in 1s
5 actionable tasks: 2 executed, 3 up-to-date
1:40:31 AM: Task execution finished 'test'.
~~~

I found this annoying, since I was not able to identify how many tests have executed, how many passed/failed. In general there were not any information similar to maven with surefire plugin. For that reason, I decided to explore the capabilities of gradle with Kotlin DSL to find out how I will build a better console reporting. 

My initial `build.gradle.kts’ was, used from junit5 samples found [here](https://github.com/junit-team/junit5-samples/tree/main/junit5-jupiter-starter-gradle-kotlin).

~~~kotlin
plugins {
    kotlin("jvm") version "1.5.31"
    java
}

group = "org.roukou.junit5.gradle"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    implementation(kotlin("stdlib"))
    testImplementation(platform("org.junit:junit-bom:5.8.1"))
    testImplementation("org.junit.jupiter:junit-jupiter-engine:5.8.1")
    testImplementation("org.junit.jupiter:junit-jupiter-params:5.8.1")
}

tasks.test {
    useJUnitPlatform()

}

tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile>().configureEach {
    kotlinOptions.jvmTarget = "1.8"
}
~~~

By doing some search and checking also the documentation provided by [gradle website](https://docs.gradle.org/current/userguide/java_testing.html#using_junit5) I found a simple addition that made my life a bit easier. By adding some testLogging code, I have manage to have some basic logging about the  test cases that were executed and which test are passed. 

~~~kotlin
tasks.test {
    useJUnitPlatform()
    testLogging {
        events("passed", "skipped", "failed")
    }
}
~~~

And now the output was better, but still was a room of significant improvement. 
~~~sh
╭─pliakas@tomato ~/Projects/opensource/roukou/junit5-gradle-kotlin-template 
╰─$ ./gradlew test
Task :test
HelperTest > Get all classes from Arraylist.class PASSED

Calculator :: Addition Tests > Multiple additions > org.roukou.junit5.CalculatorTest$AdditionTests.add(int, int, int)[1] PASSED

Calculator :: Addition Tests > Multiple additions > org.roukou.junit5.CalculatorTest$AdditionTests.add(int, int, int)[2] PASSED

Calculator :: Addition Tests > Multiple additions > org.roukou.junit5.CalculatorTest$AdditionTests.add(int, int, int)[3] PASSED

Calculator :: Addition Tests > Multiple additions > org.roukou.junit5.CalculatorTest$AdditionTests.add(int, int, int)[4] PASSED

Calculator :: Addition Tests > 1 + 1 = 2() PASSED

Calculator :: Division Tests > Divition by zero PASSED
BUILD SUCCESSFUL in 1s
5 actionable tasks: 1 executed, 4 up-to-date
~~~

I spent a whole day, reading and searching in various sites (including JUNIT5 documentation and gradle, and finally I end up with a better solution that the console output was improved and very close to the expecting result. So I end up with a better script that met my requirements. 

The gradle.build.kts was somehow more complicated, as shown below: 

~~~kotlin
    test {
        useJUnitPlatform()

        testLogging {
            lifecycle {
                events = mutableSetOf(TestLogEvent.FAILED, TestLogEvent.PASSED, TestLogEvent.SKIPPED)
                exceptionFormat = TestExceptionFormat.FULL

                showExceptions = true
                showCauses = true
                showStackTraces = true
                showStandardStreams = true
            }
            info.events = lifecycle.events
            info.exceptionFormat = lifecycle.exceptionFormat
        }

        val failedTests = mutableListOf<TestDescriptor>()
        val skippedTests = mutableListOf<TestDescriptor>()

        addTestListener(object : TestListener {
            override fun beforeSuite(suite: TestDescriptor) {}

            override fun beforeTest(testDescriptor: TestDescriptor) {}

            override fun afterTest(testDescriptor: TestDescriptor, result: TestResult) {
                when (result.resultType) {
                    TestResult.ResultType.FAILURE -> failedTests.add(testDescriptor)
                    TestResult.ResultType.SKIPPED -> skippedTests.add(testDescriptor)
                    else -> Unit
                }
            }

            override fun afterSuite(suite: TestDescriptor, result: TestResult) {
                if (suite.parent == null) { // root suite
                    logger.lifecycle("----")
                    logger.lifecycle("Test result: ${result.resultType}")
                    logger.lifecycle(
                        "Test summary: ${result.testCount} tests, " +
                                "${result.successfulTestCount} succeeded, " +
                                "${result.failedTestCount} failed, " +
                                "${result.skippedTestCount} skipped")
                    failedTests.takeIf { it.isNotEmpty() }?.prefixedSummary("\tFailed Tests")
                    skippedTests.takeIf { it.isNotEmpty() }?.prefixedSummary("\tSkipped Tests:")
                }
            }

            private infix fun List<TestDescriptor>.prefixedSummary(subject: String) {
                logger.lifecycle(subject)
                forEach { test -> logger.lifecycle("\t\t${test.displayName()}") }
            }

            private fun TestDescriptor.displayName() = parent?.let { "${it.name} - $name" } ?: "$name"

        })
    }
~~~

And the console reports was much more informative and helpful, similar to what I want it to have. 

~~~sh
╭─pliakas@tomato ~/Projects/opensource/roukou/junit5-gradle-kotlin-template 
╰─$ ./gradlew test

> Task :test

HelperTest > Get all classes from Arraylist.class PASSED

Calculator :: Addition Tests > Multiple additions > org.roukou.junit5.CalculatorTest$AdditionTests.add(int, int, int)[1] PASSED

Calculator :: Addition Tests > Multiple additions > org.roukou.junit5.CalculatorTest$AdditionTests.add(int, int, int)[2] PASSED

Calculator :: Addition Tests > Multiple additions > org.roukou.junit5.CalculatorTest$AdditionTests.add(int, int, int)[3] PASSED

Calculator :: Addition Tests > Multiple additions > org.roukou.junit5.CalculatorTest$AdditionTests.add(int, int, int)[4] PASSED

Calculator :: Addition Tests > 1 + 1 = 2() SKIPPED

Calculator :: Division Tests > Divition by zero PASSED
----
Test result: SUCCESS
Test summary: 7 tests, 6 succeeded, 0 failed, 1 skipped
        Skipped Tests:
                org.roukou.junit5.CalculatorTest$AdditionTests - 1 + 1 = 2()

BUILD SUCCESSFUL in 1s
5 actionable tasks: 1 executed, 4 up-to-date
~~~

You can find a sample project with the gradle configuration [here](https://github.com/pliakas/junit5-gradle-kotlin-template).
