# 方法引用与构造器引用详解

方法引用是 Java 8 引入的一种简化 Lambda 表达式的语法糖，它提供了一种更简洁的方式来表示仅仅调用一个已存在方法的 Lambda 表达式。

## 方法引用的类型

### 1. 静态方法引用
语法：`类名::静态方法名`

```java
// Lambda 表达式
Function<String, Integer> parser1 = str -> Integer.parseInt(str);
// 方法引用
Function<String, Integer> parser2 = Integer::parseInt;

// 示例
List<String> numbers = Arrays.asList("1", "2", "3");
List<Integer> integers = numbers.stream()
    .map(Integer::parseInt)
    .collect(Collectors.toList());
```

### 2. 实例方法引用
语法：`实例对象::实例方法名`

```java
// Lambda 表达式
Consumer<String> printer1 = str -> System.out.println(str);
// 方法引用
Consumer<String> printer2 = System.out::println;

// 示例
String text = "Hello";
Supplier<String> getter1 = () -> text.toString();
Supplier<String> getter2 = text::toString;
```

### 3. 对象的实例方法引用
语法：`类名::实例方法名`

```java
// Lambda 表达式
Comparator<String> comp1 = (s1, s2) -> s1.compareToIgnoreCase(s2);
// 方法引用
Comparator<String> comp2 = String::compareToIgnoreCase;

// 示例
List<String> names = Arrays.asList("Alice", "bob", "Charlie");
names.sort(String::compareToIgnoreCase);
```

### 4. 构造器引用
语法：`类名::new`

```java
// Lambda 表达式
Supplier<List<String>> listSupplier1 = () -> new ArrayList<>();
// 构造器引用
Supplier<List<String>> listSupplier2 = ArrayList::new;

// 带参数的构造器引用
Function<String, Person> personCreator = Person::new;
```

## 使用场景

### 1. 集合操作
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// 打印集合元素
names.forEach(System.out::println);

// 转换为大写
List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 2. Optional 类中的使用
```java
Optional<String> name = Optional.of("John");
Optional<String> upperName = name.map(String::toUpperCase);
```

### 3. 自定义类的方法引用
```java
class Person {
    private String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
    
    public static Person create(String name) {
        return new Person(name);
    }
}

// 静态方法引用
Function<String, Person> creator = Person::create;

// 实例方法引用
List<Person> persons = Arrays.asList(new Person("John"), new Person("Jane"));
List<String> names = persons.stream()
    .map(Person::getName)
    .collect(Collectors.toList());
```

## 最佳实践

### 1. 选择合适的方法引用类型
```java
// 不同场景下的最佳选择
// 静态方法
Function<String, Integer> parser = Integer::parseInt;

// 实例方法
String prefix = "Mr. ";
Function<String, String> addPrefix = prefix::concat;

// 对象方法
Comparator<String> comparator = String::compareToIgnoreCase;

// 构造器
Supplier<StringBuilder> supplier = StringBuilder::new;
```

### 2. 避免过度使用
```java
// 有时 Lambda 表达式更清晰
// 不推荐
Function<Integer, Integer> square = number -> Math::pow.apply(number, 2);
// 推荐
Function<Integer, Integer> square = number -> number * number;
```

### 3. 利用类型推断
```java
// 编译器可以推断类型时，使用方法引用更简洁
List<String> list = Arrays.asList("a", "b", "c");
list.forEach(System.out::println);
```

## 性能考虑

1. **方法引用vs Lambda表达式**
- 方法引用在性能上与等价的 Lambda 表达式相同
- 方法引用可能在内存使用上略有优势，因为它直接引用现有方法

2. **构造器引用的重用**
```java
// 缓存构造器引用
private static final Supplier<List<String>> LIST_SUPPLIER = ArrayList::new;

public List<String> createList() {
    return LIST_SUPPLIER.get();
}
```

## 常见问题和解决方案

### 1. 重载方法的引用
```java
class Utility {
    static void print(String s) {}
    static void print(Object o) {}
}

// 需要明确类型来解决歧义
Consumer<String> stringPrinter = Utility::print; // 仍然有歧义
Consumer<String> explicitPrinter = (Consumer<String>) Utility::print; // 明确类型
```

### 2. 泛型方法引用
```java
class Generic<T> {
    T transform(T t) {
        return t;
    }
}

// 使用泛型方法引用
Generic<String> stringGeneric = new Generic<>();
Function<String, String> transformer = stringGeneric::transform;
```

## 调试技巧

1. **转换为 Lambda 表达式**
- 在调试时，可以临时将方法引用转换为等价的 Lambda 表达式
- 这样可以更容易地设置断点和检查中间值

2. **使用日志**
```java
List<String> names = Arrays.asList("Alice", "Bob");
names.stream()
    .peek(System.out::println) // 用于调试的方法引用
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

## 总结

方法引用是 Java 8 中一个强大的特性，它可以让代码更加简洁和易读。合理使用方法引用可以：
1. 提高代码可读性
2. 减少冗余代码
3. 利用现有方法
4. 使代码更加函数式

但要注意在适当的场景使用方法引用，并在需要时配合 Lambda 表达式使用。