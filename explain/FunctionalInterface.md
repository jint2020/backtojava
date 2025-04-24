# 函数式接口详解

函数式接口是 Java 8 中支持 Lambda 表达式的核心。本文将深入介绍函数式接口的概念、使用方法和最佳实践。

## 什么是函数式接口？
函数式接口是只包含一个抽象方法的接口。这种接口的实例可以通过 Lambda 表达式、方法引用或构造方法引用来创建。

## 函数式接口的特点
1. 有且仅有一个抽象方法
2. 可以有多个默认方法
3. 可以有多个静态方法
4. 可以包含 Object 类的公共方法

## @FunctionalInterface 注解
```java
@FunctionalInterface
public interface MyFunction {
    void process(String input);
    
    // 默认方法是允许的
    default void defaultMethod() {
        System.out.println("默认实现");
    }
    
    // 静态方法是允许的
    static void staticMethod() {
        System.out.println("静态方法");
    }
}
```

## Java 8 内置的函数式接口

### 1. Function<T,R>
用于转换一个对象为另一个对象：
```java
Function<String, Integer> strLength = str -> str.length();
Integer length = strLength.apply("Hello"); // 返回 5
```

### 2. Predicate<T>
用于判断对象是否满足某个条件：
```java
Predicate<String> isEmpty = str -> str.isEmpty();
boolean result = isEmpty.test(""); // 返回 true
```

### 3. Consumer<T>
用于消费一个对象：
```java
Consumer<String> printer = str -> System.out.println(str);
printer.accept("Hello World"); // 打印 "Hello World"
```

### 4. Supplier<T>
用于提供一个对象：
```java
Supplier<String> supplier = () -> "Hello";
String str = supplier.get(); // 返回 "Hello"
```

### 5. UnaryOperator<T>
接收参数和返回值类型相同的函数：
```java
UnaryOperator<String> toUpperCase = str -> str.toUpperCase();
String result = toUpperCase.apply("hello"); // 返回 "HELLO"
```

### 6. BinaryOperator<T>
接收两个相同类型的参数并返回相同类型的结果：
```java
BinaryOperator<Integer> add = (a, b) -> a + b;
Integer sum = add.apply(5, 3); // 返回 8
```

## 函数式接口的组合

### 1. Predicate 组合
```java
Predicate<String> nonNull = str -> str != null;
Predicate<String> nonEmpty = str -> !str.isEmpty();
Predicate<String> valid = nonNull.and(nonEmpty);
```

### 2. Function 组合
```java
Function<String, Integer> strToInt = Integer::parseInt;
Function<Integer, Double> intToDouble = i -> i * 1.0;
Function<String, Double> strToDouble = strToInt.andThen(intToDouble);
```

## 自定义函数式接口

### 1. 基本示例
```java
@FunctionalInterface
public interface StringProcessor {
    String process(String input);
}

// 使用示例
StringProcessor toLowerCase = str -> str.toLowerCase();
```

### 2. 带泛型的函数式接口
```java
@FunctionalInterface
public interface Converter<F, T> {
    T convert(F from);
}

// 使用示例
Converter<String, Integer> converter = Integer::valueOf;
```

## 最佳实践

1. **优先使用标准函数式接口**
- 在创建自定义函数式接口之前，先检查 java.util.function 包中是否有合适的接口

2. **正确使用 @FunctionalInterface**
- 始终使用 @FunctionalInterface 注解标记函数式接口
- 这样可以在编译时检查接口是否符合函数式接口的要求

3. **合理使用默认方法**
- 默认方法可以帮助扩展接口功能而不破坏现有代码
- 但不要过度使用，保持接口的简单性

4. **注意异常处理**
```java
@FunctionalInterface
public interface ThrowingFunction<T, R, E extends Exception> {
    R apply(T t) throws E;
}
```

## 性能优化

1. **避免装箱拆箱**
- 使用专门的原始类型函数式接口（如 IntFunction、LongConsumer 等）
```java
// 不推荐
Function<Integer, Integer> square = x -> x * x; // 涉及装箱拆箱
// 推荐
IntFunction<Integer> square = x -> x * x; // 避免装箱拆箱
```

2. **缓存频繁使用的函数式接口实例**
```java
// 定义为静态常量
private static final Predicate<String> IS_EMPTY = String::isEmpty;
```

## 调试与测试

1. **测试函数式接口实现**
```java
@Test
public void testStringProcessor() {
    StringProcessor processor = str -> str.toUpperCase();
    assertEquals("HELLO", processor.process("hello"));
}
```

2. **处理异常**
```java
public static <T, R> Function<T, Optional<R>> wrap(ThrowingFunction<T, R, Exception> f) {
    return t -> {
        try {
            return Optional.ofNullable(f.apply(t));
        } catch (Exception e) {
            return Optional.empty();
        }
    };
}
```

## 常见陷阱和注意事项

1. **避免在函数式接口中抛出受检异常**
2. **注意并发安全性**
3. **不要过度使用函数式接口**
4. **注意泛型类型推断的限制**

## 总结
函数式接口是 Java 8 函数式编程的基础，合理使用可以提高代码的可读性和可维护性。在实际开发中，应该优先使用标准函数式接口，只有在特殊需求下才创建自定义函数式接口。