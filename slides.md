---
theme: apple-basic
highlighter: shiki
lineNumbers: true
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
css: unocss
fonts:
  sans: Inter
  mono: JetBrains Mono
layout: intro-image
image: https://rulesets.info/static/img/home-cover-night.png
---

<!-- TODO: Test order -->

<div class="absolute top-10">
  <span class="font-700">
    Author and Date
  </span>
</div>

<div class="absolute bottom-10">
  <h1>JUnit</h1>
  <p>A unit testing framework for Java</p>
</div>

<!--
More info : https://www.vogella.com/tutorials/JUnit/article.html
-->

---

# What is JUnit?

- A unit test framework for Java
- Current version is JUnit 5 that has a lot of new features from JUnit 4
- Best support with Maven and Gradle but we will using Maven in this slide

---

# JUnit 5

## A big update from JUnit 4

- JUnit 5 is composed of 3 modules:
  - JUnit Platform - a platform for launching testing frameworks on the JVM
  - JUnit Jupiter - a set of extensions to the JUnit Platform
  - JUnit Vintage - a set of extensions to the JUnit Platform that support JUnit 3 and JUnit 4

---

# Writing a test

```java {1-16|1|3|5|7-16|11-14}
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

import example.util.Calculator;

class MyFirstJUnitJupiterTests {

    private final Calculator calculator = new Calculator();

    @Test
    void addition() {
        assertEquals(2, calculator.add(1, 1));
    }

}
```
---
layout: intro-image
image: 'https://rulesets.info/static/img/changelog-cover-night2.png'
---

<div class="section-header">
  <h1>Basic Usage</h1>
  <p>Make a unit test with JUnit</p>
</div>

<style>
  .section-header {
    position: absolute;
    /* Align to left center */
    left: 5%;
    top: 50%;
    transform: translateY(-50%);
  }
</style>

---

# Each test

One class contains one or more tests

```java {1-23|5-8|10-15|17-22}
class CalculatorTest {

    Calculator calculator;

    @BeforeEach                                         
    void setUp() {
        calculator = new Calculator();
    }

    @Test                                               
    @DisplayName("Simple multiplication should work")   
    void testMultiply() {
        assertEquals(20, calculator.multiply(4, 5),     
                "Regular multiplication should work");  
    }

    @RepeatedTest(5) // This test will be repeated 5 times                       
    @DisplayName("Ensure correct handling of zero")
    void testMultiplyWithZero() {
        assertEquals(0, calculator.multiply(0, 5), "Multiple with zero should be zero");
        assertEquals(0, calculator.multiply(5, 0), "Multiple with zero should be zero");
    }
}
```
---

# Naming convention

Maven (our build tool) will run the test determined by the naming convention of the Java file :

- `**/Test*.java` -> includes all of its subdirectories and all Java filenames that *start with Test*.
- `**/*Test.java` -> includes all of its subdirectories and all Java filenames that *end with Test*.
- `**/*Tests.java` -> includes all of its subdirectories and all Java filenames that *end with Tests*.
- `**/*TestCase.java` -> includes all of its subdirectories and all Java filenames that *end with TestCase*.

We normally use `Test` or `Tests` suffix for the test class name.

---

# Assertions

- Use for verifying the expected result.
- There are many assertions in JUnit 5, all of them are static methods in `org.junit.jupiter.api.Assertions.*` package.

---

# Basic assertions

| Assert method | Description | Example |
| --- | --- | --- |
| `assertEquals` | Asserts that two objects are equal. | `assertEquals(2, calculator.add(1, 1), "fail message");` |
| `assertTrue` | Asserts that a condition is true. | `assertTrue(a > b, () → "fail message");` |
| `assertFalse` | Asserts that a condition is false. | `assertFalse(a > b, () → "fail message");` |
| `assertNull` | Asserts that an object is null. | `assertNull(a, () → "fail message");` |
| `assertNotNull` | Asserts that an object is not null. | `assertNotNull(a, () → "fail message");` |

---

# Error assertions

Testing for errors throws will done with `org.junit.jupiter.api.Assertions.expectThrows()` assert statement. Then use basic assertions to verify the error message.

```java {1-14|5-8|10|12-13}
import static org.junit.jupiter.api.Assertions.assertThrows;

@Test
void testExceptionMessage() {
    // Create an exception instance and pass it to the assertThrows() method
    Throwable exception = assertThrows(IllegalArgumentException.class, () -> {
        throw new IllegalArgumentException("a message");
    });

    String expectedMessage = "a message";

    // Use basic assertions to verify the error message
    assertEquals(expectedMessage, exception.getMessage());
}
```

---

# Multiple assertions tests

Normally when the assertion fails, the test will stop immediately. But if we want to test multiple assertions, we can use `org.junit.jupiter.api.assertAll()` to ensure that all assertions are executed.

```java {1-12|7|8-11|1-12}
import static org.junit.jupiter.api.Assertions.assertAll;

import com.helloyeew.penguin;

@Test
void testMultipleAssertions() {
    Penguin penguin = new Penguin("Pingu", "NootNoot");
    assertAll("penguin is person", // group name
        () -> assertEquals("Pingu", penguin.getFirstName()),
        () -> assertEquals("NootNoot", penguin.getLastName())
    );
}
```
If one of the assertions fails, the test will fail and list of all failed assertions will be shown like this :

```java
=> org.opentest4j.MultipleFailuresError: address name (2 failures)
  1) expected: <Pingu> but was: <null>
  2) expected: <NootNoot> but was: <null>
```

---

# Test timeout

- Use `org.junit.jupiter.api.Assertions.assertTimeout()` method to set the timeout for the test.
- `assertTimeout()` will run the test until finish to count the time take by the test.

```java {1-13|8-12}
import static org.junit.jupiter.api.Assertions.assertTimeout;

import com.helloyeew.penguin;

@Test
void testTimeout() {
    Penguin penguin = new Penguin("Pingu", "NootNoot");
    // The test will run until finish
    Boolean stillSleeping = assertTimeout(Duration.ofMillis(100), () -> {
        penguin.sleep(20000);
        return penguin.isSleeping();
    });
    assertTrue(stillSleeping);
```

If failed, JUnit will list the time that the test took and the time that's exceeded the timeout and throw `org.opentest4j.AssertionFailedError`.

```java
=> org.opentest4j.AssertionFailedError: execution exceeded timeout of 100 ms by 19900 ms
```

But if we don't want to wait until penguin wake up?

---

# Force quit test on timeout

- Use `org.junit.jupiter.api.Assertions.assertTimeoutPreemptively()` to set the timeout.
- `assertTimeoutPreemptively()` will force quit the test if it exceeds the timeout.

```java {1-13|8-12}
import static org.junit.jupiter.api.Assertions.assertTimeoutPreemptively;

import com.helloyeew.penguin;

@Test
void testTimeoutPreemptively() {
    Penguin penguin = new Penguin("Pingu", "NootNoot");
    // The test will run for only 100 ms
    Boolean stillSleeping = assertTimeoutPreemptively(Duration.ofMillis(100), () -> {
        penguin.sleep(20000);
        return penguin.isSleeping();
    });
    assertTrue(stillSleeping);
```

If failed, JUnit will throw `org.opentest4j.AssertionFailedError` immediately after exceed the timeout.

```java
=> org.opentest4j.AssertionFailedError: execution timed out after 100 ms
```
