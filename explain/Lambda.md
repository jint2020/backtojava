# Lambda 表达式详解

Lambda 表达式是 Java 8 引入的最重要的特性之一，它为 Java 带来了函数式编程的概念。本文将深入讲解 Lambda 表达式的使用方法和最佳实践。

## 什么是 Lambda 表达式？
Lambda 表达式本质上是一个匿名方法，可以将代码块作为方法参数传递。它具有以下特点：
- 没有名称
- 有参数列表
- 有函数体
- 可能有返回值
- 可以抛出异常

## Lambda 表达式的语法
基本语法：
```java
(parameters) -> expression
// 或
(parameters) -> { statements; }
```

### 语法变体：
1. **无参数：**
```java
() -> System.out.println("Hello")
```

2. **单个参数：**
```java
x -> x * x
```

3. **多个参数：**
```java
(x, y) -> x + y
```

4. **带类型声明：**
```java
(int x, int y) -> x + y
```

5. **带返回值：**
```java
(String s) -> { return s.length(); }
```

## 实际应用示例

### 1. 集合排序
```java
List<String> names = Arrays.asList("Peter", "Tom", "Mary");
// 传统方式
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
// Lambda 方式
Collections.sort(names, (a, b) -> b.compareTo(a));
```

### 2. 事件处理
```java
// 传统方式
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("按钮被点击");
    }
});
// Lambda 方式
button.addActionListener(e -> System.out.println("按钮被点击"));
```

### 3. 线程创建
```java
// 传统方式
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("新线程执行");
    }
}).start();
// Lambda 方式
new Thread(() -> System.out.println("新线程执行")).start();
```

## Lambda 表达式的作用域

### 1. 访问外部变量
Lambda 表达式可以捕获外部变量（必须是 final 或实际上 final 的）：
```java
int num = 10;
Runnable r = () -> System.out.println(num);
// num = 20; // 这行会导致编译错误
```

### 2. this 关键字
Lambda 表达式中的 this 指向包含 Lambda 的类：
```java
public class Lambda {
    Runnable r1 = new Runnable() {
        @Override
        public void run() {
            System.out.println(this); // 指向 Runnable 实例
        }
    };
    Runnable r2 = () -> System.out.println(this); // 指向 Lambda 类实例
}
```

## 最佳实践

1. **保持简洁**：Lambda 表达式应该简短清晰，如果代码块过长，考虑使用普通方法。

2. **类型推断**：尽可能利用 Java 的类型推断，不要显式声明参数类型。
```java
// 不推荐
Comparator<String> c = (String s1, String s2) -> s1.compareTo(s2);
// 推荐
Comparator<String> c = (s1, s2) -> s1.compareTo(s2);
```

3. **方法引用**：当 Lambda 表达式仅调用一个已存在的方法时，使用方法引用。
```java
// 不推荐
list.forEach(s -> System.out.println(s));
// 推荐
list.forEach(System.out::println);
```

4. **避免副作用**：Lambda 表达式最好是纯函数，不要修改外部变量。

## 注意事项

1. Lambda 表达式只能用于函数式接口（只有一个抽象方法的接口）。

2. Lambda 表达式捕获的外部变量必须是 final 或实际上 final 的。

3. Lambda 表达式中的异常处理需要与函数式接口的抽象方法声明的异常兼容。

4. 在需要类型转换时，最好显式声明参数类型。

## 性能考虑
Lambda 表达式在大多数情况下性能与普通实现相当，但在以下情况可能会有性能开销：
- 频繁创建 Lambda 表达式对象
- 涉及装箱/拆箱操作
- 捕获大量外部变量

## 调试技巧
1. 使用 IDE 的断点功能
2. 使用日志输出中间结果
3. 将复杂的 Lambda 表达式拆分成多个简单的步骤

## 总结
Lambda 表达式是 Java 8 引入的强大特性，它让代码更简洁、更清晰，并支持函数式编程范式。合理使用 Lambda 表达式可以显著提高代码质量和开发效率。要注意在适当的场景使用它，并遵循最佳实践原则。