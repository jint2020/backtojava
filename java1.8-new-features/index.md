# Java 1.8 新特性学习文档

Java 1.8（又称 Java 8）是 Java 语言历史上一次重要的版本更新，带来了许多革命性的新特性。以下将对 Java 8 的主要新特性进行详细讲解，并配以示例代码，帮助初学者快速掌握。

---

## 目录
1. Lambda 表达式
2. 函数式接口
3. 方法引用与构造器引用
4. Stream API
5. 新的日期时间 API（java.time 包）
6. 默认方法与静态方法（接口增强）
7. Optional 类
8. 重复注解与类型注解
9. CompletableFuture（异步编程）
10. 其它改进

---

## 1. Lambda 表达式
Lambda 表达式是 Java 8 最重要的特性之一，使得函数式编程成为可能。它可以让你更简洁地表示匿名函数。

**语法：**
```java
(parameters) -> expression
(parameters) -> { statements; }
```

**示例：**
```java
// 传统写法
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};

// Lambda 写法
Runnable r2 = () -> System.out.println("Hello Lambda");
```

**常见用法：**
- 代替匿名内部类
- 与集合、Stream API 配合

---

## 2. 函数式接口
函数式接口是只包含一个抽象方法的接口。Java 8 新增了 `@FunctionalInterface` 注解，用于标记函数式接口。

**示例：**
```java
@FunctionalInterface
public interface MyFunction {
    void doSomething();
}
```

Java 8 内置了常用的函数式接口，如 `Predicate<T>`、`Function<T,R>`、`Supplier<T>`、`Consumer<T>` 等。

---

## 3. 方法引用与构造器引用
方法引用是对已有方法的直接引用，语法为 `类名::方法名` 或 `对象::方法名`。

**示例：**
```java
// 静态方法引用
Function<String, Integer> strToInt = Integer::parseInt;

// 实例方法引用
Consumer<String> printer = System.out::println;

// 构造器引用
Supplier<List<String>> listSupplier = ArrayList::new;
```

---

## 4. Stream API
Stream API 用于对集合进行高效、声明式的数据处理（如过滤、排序、聚合等）。

**示例：**
```java
List<String> list = Arrays.asList("a", "b", "c");
list.stream()
    .filter(s -> !s.equals("b"))
    .forEach(System.out::println); // 输出 a c
```

**常用操作：**
- filter、map、reduce、collect、forEach、sorted、distinct、limit、skip

---

## 5. 新的日期时间 API（java.time 包）
Java 8 引入了全新的日期时间 API，解决了旧版 `Date`、`Calendar` 的诸多问题。

**常用类：**
- `LocalDate`、`LocalTime`、`LocalDateTime`、`Instant`、`Duration`、`Period`、`DateTimeFormatter`

**示例：**
```java
LocalDate date = LocalDate.now();
LocalTime time = LocalTime.now();
LocalDateTime dateTime = LocalDateTime.now();

// 格式化
String str = dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
System.out.println(str);
```

---

## 6. 默认方法与静态方法（接口增强）
接口可以有默认实现的方法（default method）和静态方法。

**示例：**
```java
interface MyInterface {
    default void defaultMethod() {
        System.out.println("默认方法");
    }
    static void staticMethod() {
        System.out.println("静态方法");
    }
}
```

---

## 7. Optional 类
`Optional` 用于防止空指针异常（NullPointerException），是一种容器对象。

**示例：**
```java
Optional<String> opt = Optional.ofNullable(getName());
opt.ifPresent(System.out::println);
String name = opt.orElse("默认值");
```

---

## 8. 重复注解与类型注解
- **重复注解**：同一个注解可在同一位置多次使用。
- **类型注解**：注解可用于任何类型声明处。

**示例：**
```java
@MyAnnotation("A")
@MyAnnotation("B")
public class Demo {}
```

---

## 9. CompletableFuture（异步编程）
`CompletableFuture` 支持异步编程和函数式风格的回调。

**示例：**
```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenAccept(System.out::println);
```

---

## 10. 其它改进
- Base64 编码解码 API
- 并发包增强（如 LongAdder、StampedLock）
- Nashorn JavaScript 引擎
- 新的重复注解语法

---

# 总结
Java 8 的新特性极大提升了 Java 的开发效率和表达能力。建议初学者多动手实践，结合示例代码深入理解每个特性。

---

> 本文档适合 Java 初学者系统学习 Java 8 新特性，欢迎收藏与分享。
