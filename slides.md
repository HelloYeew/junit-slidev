---
theme: apple-basic
highlighter: shiki
lineNumbers: true
info: JUnit 5 - A unit testing framework for Java
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
    HelloYeew
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

- At first JUnit had only one modules, but now JUnit 5 has 3 modules
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
        assertEquals(20, calculator.multiply(4, 5), "Regular multiplication should work");  
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

---

# Repeat test

- Use `org.junit.jupiter.api.RepeatedTest` annotation to repeat the test.

```java
import org.junit.jupiter.api.RepeatedTest;
import static org.junit.jupiter.api.Assertions.assertEquals;

@RepeatedTest(10)
void testRepeated() {
    assertEquals(2, calculator.add(1, 1));
}
```

---

# Disable test

- Use `@Disabled` annotation to disable the test.

```java
import org.junit.jupiter.api.Disabled;

@Disabled("Disabled until I wake up")
@Test
void testDisabled() {
    // Some test here!
}
```

---
layout: intro-image
image: 'https://rulesets.info/static/img/status-cover-night.jpg'
---

<div class="section-header">
  <h1>Dynamic & parametized tests</h1>
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

# Dynamic tests

- Dynamic test method are annotated with `@TestFactory` annotation and allow you to create multiple tests of type `DynamicTest` with your code.
- This can return `Iterable` or `Stream` of `DynamicTest` objects.
- JUnit 5 creates and runs all dynamic tests during test execution.
- Note : `@BeforeAll` and `@AfterAll` will not work with dynamic tests.

```java
class DynamicTestCreationTest {

    @TestFactory
    Stream<DynamicTest> testDifferentMultiplyOperations() {
        MyClass tester = new MyClass();
        int[][] data = new int[][] { { 1, 2, 2 }, { 5, 3, 15 }, { 121, 4, 484 } };
        return Arrays.stream(data).map(entry -> {
            int m1 = entry[0];
            int m2 = entry[1];
            int expected = entry[2];
            return dynamicTest(m1 + " * " + m2 + " = " + expected, () -> {
                assertEquals(expected, tester.multiply(m1, m2));
            });
        });
    }

    // class to be tested
    class MyClass {
        public int multiply(int i, int j) {
            return i * j;
        }
    }
}
```

---

# Parametized tests

- JUnit use `junit-jupiter-params` library to create parametized tests. (You need to add this library to your project)
- Parametized test method are annotated with `@ParameterizedTest` annotation and allow you to create multiple tests with your code.
- Use `@<something>Source` annotation to provide the data for the test.

---

# Data sources example

| Annotation | Description |
| --- | --- |
| `@ValueSource(ints = { 1, 2, 3 })` | Define an array of test values. Permissible types are String, int, long, or double. |
| `@EnumSource(value=MyEnum.class, names = { "ONE", "TWO" })` | Pass Enum constants as test class. With the optional attribute names you can choose which constants should be used. Otherwise all attributes are used. |
| `@MethodSource(names = "genTestData")` | The result of the named method is passed as argument to the test. |
| `@CsvSource({ "1, 2, 3", "4, 5, 9" })` | Expects strings to be parsed as Csv. The delimiter is ','. |
| `@ArgumentsSource(MyArgumentsProvider.class)` | Specifies a class that provides the test data. The referenced class has to implement the ArgumentsProvider interface. |

---

# Example

```java
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;

public class UsingParameterizedTest {

    public static int[][] data() {
        return new int[][] { { 1 , 2, 2 }, { 5, 3, 15 }, { 121, 4, 484 } };
    }

    @ParameterizedTest
    @MethodSource(value =  "data")
    void testWithStringParameter(int[] data) {
        MyClass tester = new MyClass();
        int m1 = data[0];
        int m2 = data[1];
        int expected = data[2];
        assertEquals(expected, tester.multiply(m1, m2));
    }

    // class to be tested
    class MyClass {
        public int multiply(int i, int j) {
            return i * j;
        }
    }
}
```

---
layout: intro-image
image: 'https://www.helloyeew.dev/assets/555559.jpg'
---

<div class="section-header">
  <h1>More info</h1>
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

# Nested test

- The `@Nested` annotation can be used to annotate inner classes which also contain tests.
- This allow to group test and have additional `@BeforeEach` and `@AfterEach` methods.
- Additional rules
  - All nested test classes must be non-static inner classes.
  - The nested test classes are annotated with `@Nested` annotation so that the runtime can recognize the nested test classes.
  - A nested test class can contain `Test` methods, one `@BeforeEach` method, and one `@AfterEach` method.

---

# Test order

- JUnit test order is a deterministic but unpredictable order. -> Use `@TestMethodOrder` annotation to change the order.
  - `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)` - Allows to use the @Order(int) annotation on methods to define order
  - `@TestMethodOrder(MethodOrderer.DisplayName.class)` - runs test method in alphanumeric order of display name
  - `@TestMethodOrder(MethodOrderer.MethodName.class)` - runs test method in alphanumeric order of method name
  - Custom implementation - Implement your own `MethodOrderer` via the `orderMethods` method, which allows you to call `context.getMethodDescriptors().sort(..)`

---

# Test order example

```java {1-21|9-13|15-19}
import org.junit.jupiter.api.MethodOrderer.OrderAnnotation;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;

@TestMethodOrder(OrderAnnotation.class)
class OrderAnnotationDemoTest {

    @Test
    @Order(1)
    void firstOne() {
        // test something here
    }

    @Test
    @Order(2)
    void secondOne() {
        // test something here
    }

}
```

---

# Test suites

- Test suites are a way to group tests together and set up some common configuration for them.

```java{1-10|5-7}
import org.junit.platform.suite.api.SelectPackages;
import org.junit.platform.suite.api.Suite;
import org.junit.platform.suite.api.SuiteDisplayName;

@Suite
@SuiteDisplayName("JUnit Platform Suite Demo")
@SelectPackages("penguin")
public class SuiteDemo {

}
```

---
layout: intro-image
---

<div class="section-header">
  <h1>Let's see some real 'game' tests!</h1>
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