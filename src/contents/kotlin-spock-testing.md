---
author: Szymon Marcinkiewicz
datetime: 2022-04-03T22:00:00Z
title: "Testing Kotlin with Spock - final drawbacks"
slug: kotlin-spock-testing
featured: true
draft: false
tags:
  - kotlin
  - spock
ogImage: ""
description: How Kotlin's final classes affect testing with Spock
---

Last time when I was writing unit tests in Spock for Kotlin application, I've faced interesting problem.

Take a look at the example below - I've declared function `test` which returns `false`, but in test I've mocked it to return `true`.

**Application code:**
```kotlin
class KotlinClass {
    fun test(): Boolean {
        println("Executing KotlinClass.test method...")
        return false
    }
}
```

**Testing code:**
```groovy
def 'test mock in kotlin'() {
    when:
    def kotlin = Mock(KotlinClass)
    kotlin.test() >> true // Declare that mock should return true

    then:
    def result = kotlin.test()
    result == true // Verify that calling mock results in true
}
```

Running this test failed with error message:
> ```Cannot create mock for class KotlinClass because Java mocks cannot mock final classes. If the code under test is written in Groovy, use a Groovy mock.```

Reason for that is because [in Kotlin all classes are final by default](https://kotlinlang.org/docs/inheritance.html).
Fix for this issue was very quick, I've just had to add `open` modifier before `class`.

```kotlin
open class KotlinClass
```

But after applying that fix, strange thing happened. Running test resulted in yet another error:

```
Executing real KotlinClass.test method

Condition not satisfied:

result == true
|      |
false  false
```

Take a look at the first line of the result - it says `Executing real KotlinClass.test method`!
It occurs that Spock had not mocked our `test` method, despite explicit declaration via:

```groovy
def kotlin = Mock(KotlinClass)
kotlin.test() >> true // Declare that mock should return true
```

### But why?

[In Kotlin, all methods are final by default](https://kotlinlang.org/docs/inheritance.html#overriding-methods). Spock is unable to create Mock or Stub on `final` class or method.

### How can I solve this?

No-brainer solution is same as in case of `final class` - you can just use `open` modifier before function itself:

```kotlin
open fun test(): Boolean
```

But if you don't want your Spock tests to affect application code itself (_you really shouldn't want it_), you can use [spock-mockable](https://github.com/joke/spock-mockable) as it allows creating Stubs and Mocks on final (and even private) classes or methods.
After adding `spock-mockable` dependency apply `@Mockable` annotation on Spock specification with _"problematic"_ class as argument.

```groovy
@Mockable(KotlinClass)
class SpockTest extends Specification {
    def 'test mock in kotlin'() {
        when:
        def kotlin = Mock(KotlinClass)
        kotlin.test() >> true // Declare that mock should return true

        then:
        def result = kotlin.test()
        result == true // Verify that calling mock results in true
    }
}
```

Doing so, you keep your code untouched (without any `open` modifier on class or method).
