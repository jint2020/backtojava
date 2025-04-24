# Java 8 其他改进特性详解

Java 8 除了主要的特性外，还引入了许多其他有用的改进。本文将详细介绍这些改进特性。

## Base64 编码解码

Java 8 在 `java.util` 包中添加了 Base64 编码的编码器和解码器。

### 1. 基本用法
```java
// 编码
String text = "Hello World!";
String encoded = Base64.getEncoder().encodeToString(text.getBytes());

// 解码
byte[] decoded = Base64.getDecoder().decode(encoded);
String decodedText = new String(decoded);
```

### 2. URL 安全的 Base64
```java
// URL 安全的编码
String urlEncoded = Base64.getUrlEncoder().encodeToString(text.getBytes());

// URL 安全的解码
byte[] urlDecoded = Base64.getUrlDecoder().decode(urlEncoded);
```

### 3. MIME 编码
```java
// MIME 编码
String mimeEncoded = Base64.getMimeEncoder().encodeToString(text.getBytes());

// MIME 解码
byte[] mimeDecoded = Base64.getMimeDecoder().decode(mimeEncoded);
```

## 并发包增强

### 1. LongAdder
LongAdder 提供了比 AtomicLong 更好的高并发性能。

```java
public class CounterExample {
    // 传统方式
    private AtomicLong atomicCounter = new AtomicLong();
    
    // 新方式
    private LongAdder adderCounter = new LongAdder();
    
    public void incrementAtomic() {
        atomicCounter.incrementAndGet();
    }
    
    public void incrementAdder() {
        adderCounter.increment();
    }
    
    public long getAtomicCount() {
        return atomicCounter.get();
    }
    
    public long getAdderCount() {
        return adderCounter.sum();
    }
}
```

### 2. StampedLock
StampedLock 提供了一种乐观读锁的实现，性能优于 ReadWriteLock。

```java
public class Point {
    private double x, y;
    private final StampedLock sl = new StampedLock();
    
    // 写入数据
    public void move(double deltaX, double deltaY) {
        long stamp = sl.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            sl.unlockWrite(stamp);
        }
    }
    
    // 乐观读
    public double distanceFromOrigin() {
        long stamp = sl.tryOptimisticRead();
        double currentX = x, currentY = y;
        if (!sl.validate(stamp)) {
            stamp = sl.readLock();
            try {
                currentX = x;
                currentY = y;
            } finally {
                sl.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
}
```

### 3. CompletableFuture
CompletableFuture 已经在专门的文档中详细介绍过。

## Nashorn JavaScript 引擎

Nashorn 是 Java 8 引入的新的 JavaScript 引擎，替代了旧的 Rhino。

### 1. 基本用法
```java
public class NashornExample {
    public void executeScript() throws ScriptException {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("nashorn");
        
        // 执行 JavaScript 代码
        engine.eval("print('Hello from JavaScript!');");
    }
    
    public void executeFunction() throws ScriptException, NoSuchMethodException {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("nashorn");
        
        // 定义并执行函数
        engine.eval("function add(a, b) { return a + b; }");
        
        Invocable invocable = (Invocable) engine;
        Object result = invocable.invokeFunction("add", 1, 2);
        System.out.println(result); // 输出 3
    }
}
```

### 2. Java 和 JavaScript 交互
```java
public class JavaScriptInterop {
    public static class Person {
        private String name;
        
        public Person(String name) {
            this.name = name;
        }
        
        public String greet() {
            return "Hello, " + name + "!";
        }
    }
    
    public void interopExample() throws ScriptException {
        ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
        
        // 将 Java 对象传递给 JavaScript
        engine.put("person", new Person("John"));
        
        // 在 JavaScript 中使用 Java 对象
        engine.eval("print(person.greet());");
    }
}
```

## 其他小改进

### 1. 参数名称反射
Java 8 允许在编译时保留参数名称，并通过反射API获取它们。

```java
public class ParameterNames {
    public void example(@NotNull String name, @Min(1) int age) {
        Method method = getClass().getDeclaredMethod("example", String.class, int.class);
        Parameter[] parameters = method.getParameters();
        
        for (Parameter parameter : parameters) {
            System.out.println("参数名: " + parameter.getName());
            System.out.println("注解: " + Arrays.toString(parameter.getAnnotations()));
        }
    }
}
```

### 2. 新的日期时间格式化工具
```java
public class DateFormatExample {
    public void formatExample() {
        LocalDateTime now = LocalDateTime.now();
        
        // 使用预定义的格式
        DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
        String formatted = now.format(formatter);
        
        // 自定义格式
        DateTimeFormatter customFormatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH时mm分ss秒");
        String customFormatted = now.format(customFormatter);
    }
}
```

### 3. 集合的改进
```java
public class CollectionImprovements {
    public void examples() {
        // List 的 removeIf 方法
        List<String> list = new ArrayList<>();
        list.removeIf(s -> s.isEmpty());
        
        // Map 的新方法
        Map<String, Integer> map = new HashMap<>();
        
        // 计算值
        map.compute("key", (k, v) -> v == null ? 1 : v + 1);
        
        // 不存在时才放入
        map.putIfAbsent("key", 0);
        
        // 合并值
        map.merge("key", 1, Integer::sum);
    }
}
```

## 性能改进

### 1. JVM 改进
- 改进的垃圾收集器
- 更好的内存管理
- 启动时间优化

### 2. 永久代移除
- 永久代被元空间（Metaspace）替代
- 动态调整元空间大小
- 减少 OutOfMemoryError 的发生

## 最佳实践

1. **Base64 使用建议**
```java
// 对于大文件，使用流式处理
try (InputStream input = new FileInputStream("input.txt");
     OutputStream output = Base64.getEncoder().wrap(new FileOutputStream("output.txt"))) {
    byte[] buffer = new byte[1024];
    int len;
    while ((len = input.read(buffer)) != -1) {
        output.write(buffer, 0, len);
    }
}
```

2. **并发工具选择**
- 简单计数器使用 LongAdder
- 需要原子操作时使用 AtomicLong
- 需要读写锁时考虑 StampedLock

3. **JavaScript 引擎使用建议**
- 重用 ScriptEngine 实例
- 注意内存管理
- 使用 try-with-resources 管理资源

## 调试技巧

### 1. Nashorn 调试
```java
public class NashornDebug {
    public void debugScript() {
        ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
        engine.put("console", new Object() {
            public void log(Object msg) {
                System.out.println(msg);
            }
        });
        
        // 在 JavaScript 中使用 console.log
        engine.eval("console.log('调试信息');");
    }
}
```

### 2. 并发调试
```java
public class ConcurrencyDebug {
    public void debugConcurrency() {
        LongAdder adder = new LongAdder();
        // 添加调试信息
        adder.add(1);
        System.out.println("当前值: " + adder.sum());
    }
}
```

## 总结

Java 8 的其他改进提供了：

1. **更方便的工具类**
- Base64 编解码
- 更强大的并发工具
- JavaScript 集成

2. **性能提升**
- 并发性能改进
- JVM 优化
- 内存管理改进

3. **开发体验改善**
- 更多的工具方法
- 更好的 API 设计
- 更强的调试能力

使用建议：
1. 根据场景选择合适的工具
2. 注意性能影响
3. 遵循最佳实践
4. 合理使用新特性