# Stream API 详解

Stream API 是 Java 8 引入的一种处理集合的强大特性，它可以让你以一种声明式的方式处理数据集合。本文将深入介绍 Stream API 的使用方法和最佳实践。

## 什么是 Stream？

Stream 是元素的序列，支持串行和并行操作。它不是数据结构，不存储数据，而是在数据源（如集合）上进行操作。

## Stream 的特点

1. **不存储数据**：Stream 不是数据结构，只是数据源的视图
2. **函数式编程**：提供了函数式编程的特性
3. **惰性执行**：很多 Stream 操作是惰性执行的
4. **可消费性**：Stream 只能被消费一次
5. **无状态**：Stream 操作不会修改数据源

## 创建 Stream

### 1. 从集合创建
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream();
```

### 2. 从数组创建
```java
String[] arr = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
```

### 3. 使用 Stream.of
```java
Stream<String> stream = Stream.of("a", "b", "c");
```

### 4. 创建无限流
```java
// 生成从1开始的无限整数流
Stream<Integer> numbers = Stream.iterate(1, n -> n + 1);
// 生成随机数流
Stream<Double> randoms = Stream.generate(Math::random);
```

## Stream 操作

### 1. 中间操作（Intermediate Operations）

#### filter：过滤元素
```java
List<String> filtered = Stream.of("a", "b", "c")
    .filter(s -> s.startsWith("a"))
    .collect(Collectors.toList()); // 结果: [a]
```

#### map：转换元素
```java
List<String> upperCase = Stream.of("a", "b", "c")
    .map(String::toUpperCase)
    .collect(Collectors.toList()); // 结果: [A, B, C]
```

#### flatMap：扁平化流
```java
List<List<Integer>> lists = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
List<Integer> flatList = lists.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList()); // 结果: [1, 2, 3, 4]
```

#### distinct：去重
```java
List<Integer> distinct = Stream.of(1, 1, 2, 2, 3)
    .distinct()
    .collect(Collectors.toList()); // 结果: [1, 2, 3]
```

#### sorted：排序
```java
List<String> sorted = Stream.of("c", "b", "a")
    .sorted()
    .collect(Collectors.toList()); // 结果: [a, b, c]
```

#### peek：查看元素
```java
List<String> result = Stream.of("a", "b", "c")
    .peek(System.out::println)
    .collect(Collectors.toList());
```

### 2. 终端操作（Terminal Operations）

#### collect：收集结果
```java
// 收集为List
List<String> list = stream.collect(Collectors.toList());

// 收集为Set
Set<String> set = stream.collect(Collectors.toSet());

// 收集为Map
Map<String, Integer> map = stream.collect(
    Collectors.toMap(String::toLowerCase, String::length)
);
```

#### reduce：归约
```java
// 求和
int sum = Stream.of(1, 2, 3)
    .reduce(0, Integer::sum);

// 连接字符串
String concat = Stream.of("a", "b", "c")
    .reduce("", String::concat);
```

#### forEach：遍历
```java
Stream.of("a", "b", "c")
    .forEach(System.out::println);
```

#### count：计数
```java
long count = Stream.of("a", "b", "c")
    .count();
```

#### anyMatch/allMatch/noneMatch：匹配
```java
boolean hasA = Stream.of("a", "b", "c")
    .anyMatch(s -> s.equals("a")); // true

boolean allUpperCase = Stream.of("A", "B", "C")
    .allMatch(s -> s.equals(s.toUpperCase())); // true

boolean noZ = Stream.of("a", "b", "c")
    .noneMatch(s -> s.equals("z")); // true
```

## 高级操作

### 1. 并行流
```java
// 使用并行流提高性能
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();
```

### 2. 自定义收集器
```java
// 创建自定义收集器
Collector<String, StringBuilder, String> customCollector = Collector.of(
    StringBuilder::new,           // supplier
    StringBuilder::append,        // accumulator
    StringBuilder::append,        // combiner
    StringBuilder::toString       // finisher
);
```

### 3. 分组和分区
```java
// 分组
Map<Integer, List<String>> groups = Stream.of("a", "bb", "ccc")
    .collect(Collectors.groupingBy(String::length));

// 分区
Map<Boolean, List<Integer>> partition = Stream.of(1, 2, 3, 4, 5)
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

## 最佳实践

### 1. 使用合适的操作顺序
```java
// 推荐：先filter后map
stream.filter(predicate).map(mapper)

// 不推荐：先map后filter
stream.map(mapper).filter(predicate)
```

### 2. 正确使用并行流
```java
// 适合并行的场景
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());

// 不适合并行的场景（有状态操作）
stream.parallel()
    .sorted()
    .collect(Collectors.toList());
```

### 3. 避免副作用
```java
// 不推荐：使用外部变量
int[] sum = new int[1];
Stream.of(1, 2, 3).forEach(n -> sum[0] += n);

// 推荐：使用reduce
int sum = Stream.of(1, 2, 3).reduce(0, Integer::sum);
```

## 性能优化

### 1. 使用合适的数据结构
```java
// ArrayList适合顺序访问
List<Integer> list = new ArrayList<>();
list.stream()...

// HashSet适合去重
Set<Integer> set = new HashSet<>();
set.stream()...
```

### 2. 避免装箱拆箱
```java
// 不推荐
Stream<Integer> stream = numbers.stream().map(i -> i * 2);

// 推荐
IntStream stream = numbers.stream().mapToInt(i -> i * 2);
```

### 3. 限制流大小
```java
// 使用limit限制处理数量
stream.limit(1000).forEach(process);
```

## 调试技巧

### 1. 使用peek查看中间结果
```java
Stream.of("a", "b", "c")
    .peek(e -> System.out.println("Original: " + e))
    .map(String::toUpperCase)
    .peek(e -> System.out.println("Mapped: " + e))
    .collect(Collectors.toList());
```

### 2. 分步调试
```java
// 将复杂操作分解为多个步骤
Stream<String> filtered = stream.filter(predicate);
Stream<String> mapped = filtered.map(mapper);
List<String> result = mapped.collect(Collectors.toList());
```

## 常见陷阱

1. **流重用**：Stream 只能使用一次
2. **无限流**：注意使用 limit 或其他终止条件
3. **并行流陷阱**：注意线程安全和性能影响
4. **装箱/拆箱开销**：优先使用基本类型流

## 总结

Stream API 是 Java 8 中最强大的特性之一，它提供了一种优雅的方式来处理集合数据。合理使用 Stream API 可以：
1. 提高代码可读性
2. 减少样板代码
3. 支持并行处理
4. 提供函数式编程能力

建议在实际开发中根据具体场景选择合适的 Stream 操作，并注意性能优化和陷阱避免。