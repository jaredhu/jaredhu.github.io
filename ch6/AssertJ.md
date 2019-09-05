# AssertJ - 流畅的断言java库

https://assertj.github.io/doc/

## 1.概述

本文档的目标是为使用AssertJ编写测试断言的程序员提供全面的参考文档。

### 1.1。什么是AssertJ Core？

AssertJ是一个java库，提供丰富的断言，真正有用的错误消息，提高了测试代码的可读性，并且设计为在您喜欢的IDE中非常容易使用。

以下是AssertJ断言的一些示例：

```java
// entry point for all assertThat methods and utility methods (e.g. entry)
import static org.assertj.core.api.Assertions.*;

// basic assertions
assertThat(frodo.getName()).isEqualTo("Frodo");
assertThat(frodo).isNotEqualTo(sauron);

// chaining string specific assertions
assertThat(frodo.getName()).startsWith("Fro")
                           .endsWith("do")
                           .isEqualToIgnoringCase("frodo");

// collection specific assertions (there are plenty more)
// in the examples below fellowshipOfTheRing is a List<TolkienCharacter>
assertThat(fellowshipOfTheRing).hasSize(9)
                               .contains(frodo, sam)
                               .doesNotContain(sauron);

// as() is used to describe the test and will be shown before the error message
assertThat(frodo.getAge()).as("check %s's age", frodo.getName()).isEqualTo(33);

// exception assertion, standard style ...
assertThatThrownBy(() -> { throw new Exception("boom!"); }).hasMessage("boom!");
// ... or BDD style
Throwable thrown = catchThrowable(() -> { throw new Exception("boom!"); });
assertThat(thrown).hasMessageContaining("boom");

// using the 'extracting' feature to check fellowshipOfTheRing character's names
assertThat(fellowshipOfTheRing).extracting(TolkienCharacter::getName)
                               .doesNotContain("Sauron", "Elrond");

// extracting multiple values at once grouped in tuples
assertThat(fellowshipOfTheRing).extracting("name", "age", "race.name")
                               .contains(tuple("Boromir", 37, "Man"),
                                         tuple("Sam", 38, "Hobbit"),
                                         tuple("Legolas", 1000, "Elf"));

// filtering a collection before asserting
assertThat(fellowshipOfTheRing).filteredOn(character -> character.getName().contains("o"))
                               .containsOnly(aragorn, frodo, legolas, boromir);

// combining filtering and extraction (yes we can)
assertThat(fellowshipOfTheRing).filteredOn(character -> character.getName().contains("o"))
                               .containsOnly(aragorn, frodo, legolas, boromir)
                               .extracting(character -> character.getRace().getName())
                               .contains("Hobbit", "Elf", "Man");

// and many more assertions: iterable, stream, array, map, dates, path, file, numbers, predicate, optional ...
```

## 2.AssertJ Core快速入门

本指南适用于AssertJ核心模块。

- AssertJ Core 2.x处于维护模式，它只会收到错误修正。

### 2.1。获取assertj-core库

AssertJ Core工件位于Maven中央存储库中。

#### 2.1.1。支持的Java版本

AssertJ Core主要版本依赖于不同的Java版本：

- AssertJ Core 3.x需要Java 8或更高版本
- AssertJ Core 2.x需要Java 7或更高版本

请注意，AssertJ Core 3.x包含所有AssertJ Core 2.x功能，并添加了Java 8特定的功能（如lambdas的异常断言）。

#### 2.1.2。Android支持

AssertJ **没有正式支持Android，**但主要是Android兼容：

- AssertJ Core 3.x与Android API Level 26+兼容，但软断言和假设除外。
- AssertJ Core 2.x与Android API Level 26+和API级别<26兼容Android，`Path`断言除外。

#### 2.1.3。Maven的

```xml
<dependency>
  <groupId>org.assertj</groupId>
  <artifactId>assertj-core</artifactId>
  <!-- use 2.9.1 for Java 7 projects -->
  <version>3.13.2</version>
  <scope>test</scope>
</dependency>
```

### 使用Assertions类入口点

该`Assertions`班是你需要开始使用AssertJ唯一的类，它提供了所有你需要的方法。

或者，您的测试类可以实现`WithAssertions`访问相同的方法。

一个`Assertions`静态导入来统治它们......

```java
import static org.assertj.core.api.Assertions.*;
```

......如果你愿意，可以选择......

```java
import static org.assertj.core.api.Assertions.assertThat;  // main one
import static org.assertj.core.api.Assertions.atIndex; // for List assertions
import static org.assertj.core.api.Assertions.entry;  // for Map assertions
import static org.assertj.core.api.Assertions.tuple; // when extracting several properties at once
import static org.assertj.core.api.Assertions.fail; // use when writing exception tests
import static org.assertj.core.api.Assertions.failBecauseExceptionWasNotThrown; // idem
import static org.assertj.core.api.Assertions.filter; // for Iterable/Array assertions
import static org.assertj.core.api.Assertions.offset; // for floating number assertions
import static org.assertj.core.api.Assertions.anyOf; // use with Condition
import static org.assertj.core.api.Assertions.contentOf; // use with File assertions
```

#### 2.2.1。替代入口点

AssertJ提供了其他入口点类，特别是`WithAssertions`接口和`BDDAssertions`用于替换`assertThat`的BDD样式断言`then`。

`WithAssertions` 例：

```java
import org.assertj.core.api.WithAssertions;

public class WithAssertionsExamples extends AbstractAssertionsExamples implements WithAssertions {

  // the data used are initialized in AbstractAssertionsExamples.

  @Test
  public void withAssertions_examples() {

    // assertThat methods come from WithAssertions - no static import needed
    assertThat(frodo.age).isEqualTo(33);
    assertThat(frodo.getName()).isEqualTo("Frodo").isNotEqualTo("Frodon");

    assertThat(frodo).isIn(fellowshipOfTheRing);
    assertThat(frodo).isIn(sam, frodo, pippin);
    assertThat(sauron).isNotIn(fellowshipOfTheRing);

    assertThat(frodo).matches(p -> p.age > 30 && p.getRace() == HOBBIT);
    assertThat(frodo.age).matches(p -> p > 30);
  }
}
```

`BDDAssertions` 例：

```java
import static org.assertj.core.api.BDDAssertions.then;

public class BDDAssertionsExamples extends AbstractAssertionsExamples {

  // the data used are initialized in AbstractAssertionsExamples.

  @Test
  public void withAssertions_examples() {

    // then methods come from BDDAssertions.then static
    then(frodo.age).isEqualTo(33);
    then(frodo.getName()).isEqualTo("Frodo").isNotEqualTo("Frodon");

    then(frodo).isIn(fellowshipOfTheRing);
    then(frodo).isIn(sam, frodo, pippin);
    then(sauron).isNotIn(fellowshipOfTheRing);

    then(frodo).matches(p -> p.age > 30 && p.getRace() == HOBBIT);
    then(frodo.age).matches(p -> p > 30);
  }
}
```

#### 2.2.2。IDE配置

您可以配置IDE，以便在开始键入`as`和触发代码完成时，`assertThat`将显示在建议的完成中。

Eclipse :. 转至：窗口>首选项> Java>编辑器>内容辅助>收藏夹>新类型。输入：`org.assertj.core.api.Assertions`并单击“确定”。检查您是否`org.assertj.core.api.Assertions.*`在收藏夹中看到。

Intellij Idea：不需要特殊配置，只需开始输入`asser`然后两次调用完成（Ctrl-Space）。



