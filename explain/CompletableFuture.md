# CompletableFuture 详解

CompletableFuture 是 Java 8 引入的一个强大的异步编程工具，它弥补了 Future 接口的不足，提供了非常强大的函数式编程能力。本文将详细介绍 CompletableFuture 的使用方法和最佳实践。

## 为什么需要 CompletableFuture？

传统的 Future 接口存在以下限制：
1. 不能手动完成
2. 不能进行链式操作
3. 无法合并多个异步操作
4. 没有异常处理机制
5. 不支持回调函数

## CompletableFuture 基础

### 1. 创建 CompletableFuture
```java
// 创建一个完成的 CompletableFuture
CompletableFuture<String> completed = CompletableFuture.completedFuture("结果");

// 创建一个异步执行的 CompletableFuture
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 执行异步任务
    return "异步结果";
});

// 创建一个不返回结果的异步任务
CompletableFuture<Void> voidFuture = CompletableFuture.runAsync(() -> {
    // 执行异步任务
    System.out.println("异步任务执行完成");
});
```

### 2. 获取结果
```java
// 同步获取结果
String result = future.get(); // 可能抛出异常
String result2 = future.get(1, TimeUnit.SECONDS); // 带超时的获取

// 获取结果（如果没有完成则使用默认值）
String result3 = future.getNow("默认值");

// 等待完成
future.join(); // 类似于get，但抛出非受检异常
```

## 链式操作

### 1. 转换结果（thenApply, thenApplyAsync）
```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenApply(String::toUpperCase);

// 异步转换
CompletableFuture<String> asyncFuture = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenApplyAsync(s -> s + " World");
```

### 2. 消费结果（thenAccept, thenRun）
```java
// 处理结果
CompletableFuture<Void> future = CompletableFuture
    .supplyAsync(() -> "结果")
    .thenAccept(System.out::println);

// 执行后续操作（不使用结果）
CompletableFuture<Void> future2 = CompletableFuture
    .supplyAsync(() -> "结果")
    .thenRun(() -> System.out.println("完成"));
```

### 3. 组合（thenCompose, thenCombine）
```java
// 串行组合两个Future
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World"));

// 并行组合两个Future
CompletableFuture<String> future2 = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenCombine(
        CompletableFuture.supplyAsync(() -> " World"),
        (s1, s2) -> s1 + s2
    );
```

## 异常处理

### 1. 处理异常（exceptionally, handle）
```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        if (true) throw new RuntimeException("错误");
        return "结果";
    })
    .exceptionally(ex -> "发生异常：" + ex.getMessage());

// 使用handle处理正常结果和异常
CompletableFuture<String> future2 = CompletableFuture
    .supplyAsync(() -> "结果")
    .handle((result, ex) -> {
        if (ex != null) {
            return "发生异常：" + ex.getMessage();
        }
        return result;
    });
```

### 2. 异常恢复（whenComplete）
```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "结果")
    .whenComplete((result, ex) -> {
        if (ex != null) {
            System.out.println("发生异常：" + ex.getMessage());
        } else {
            System.out.println("结果：" + result);
        }
    });
```

## 多任务协作

### 1. 等待所有任务完成（allOf）
```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "结果1");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "结果2");
CompletableFuture<String> future3 = CompletableFuture.supplyAsync(() -> "结果3");

// 等待所有任务完成
CompletableFuture<Void> allFuture = CompletableFuture.allOf(
    future1, future2, future3);

// 获取所有结果
allFuture.thenRun(() -> {
    String result1 = future1.join();
    String result2 = future2.join();
    String result3 = future3.join();
    System.out.println(result1 + result2 + result3);
});
```

### 2. 任意任务完成（anyOf）
```java
CompletableFuture<Object> anyFuture = CompletableFuture.anyOf(
    future1, future2, future3);

anyFuture.thenAccept(result -> 
    System.out.println("第一个完成的结果：" + result));
```

## 实际应用示例

### 1. 异步HTTP请求
```java
public class AsyncHttpClient {
    public CompletableFuture<String> asyncGet(String url) {
        return CompletableFuture.supplyAsync(() -> {
            // 模拟HTTP请求
            try {
                Thread.sleep(1000);
                return "Response from " + url;
            } catch (InterruptedException e) {
                throw new CompletionException(e);
            }
        });
    }
    
    // 使用示例
    public void example() {
        asyncGet("http://api1.example.com")
            .thenCombine(
                asyncGet("http://api2.example.com"),
                (result1, result2) -> result1 + " + " + result2
            )
            .thenAccept(System.out::println)
            .exceptionally(ex -> {
                System.err.println("请求失败：" + ex.getMessage());
                return null;
            });
    }
}
```

### 2. 异步缓存加载
```java
public class AsyncCache<K, V> {
    private final Map<K, CompletableFuture<V>> cache = new ConcurrentHashMap<>();
    
    public CompletableFuture<V> get(K key, Function<K, V> loader) {
        return cache.computeIfAbsent(key, k ->
            CompletableFuture.supplyAsync(() -> loader.apply(k))
                .whenComplete((v, ex) -> {
                    if (ex != null) {
                        cache.remove(key);
                    }
                }));
    }
}
```

### 3. 并行任务处理
```java
public class ParallelProcessor {
    public CompletableFuture<List<String>> processItems(List<String> items) {
        List<CompletableFuture<String>> futures = items.stream()
            .map(item -> CompletableFuture.supplyAsync(() -> processItem(item)))
            .collect(Collectors.toList());
            
        return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList()));
    }
    
    private String processItem(String item) {
        // 处理单个项目
        return item.toUpperCase();
    }
}
```

## 最佳实践

### 1. 线程池管理
```java
// 创建自定义线程池
ExecutorService executor = Executors.newFixedThreadPool(10);

// 使用自定义线程池
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "结果", executor)
    .thenApplyAsync(String::toUpperCase, executor);

// 记得关闭线程池
executor.shutdown();
```

### 2. 超时处理
```java
public class TimeoutExample {
    public <T> CompletableFuture<T> withTimeout(
            CompletableFuture<T> future, 
            long timeout, 
            TimeUnit unit) {
        CompletableFuture<T> timeoutFuture = timeoutAfter(timeout, unit);
        return future.applyToEither(timeoutFuture, Function.identity());
    }
    
    private <T> CompletableFuture<T> timeoutAfter(long timeout, TimeUnit unit) {
        CompletableFuture<T> result = new CompletableFuture<>();
        Delayer.delay(() -> result.completeExceptionally(
            new TimeoutException()), timeout, unit);
        return result;
    }
}
```

### 3. 错误处理最佳实践
```java
public class ErrorHandlingExample {
    public CompletableFuture<String> robustOperation() {
        return CompletableFuture
            .supplyAsync(this::riskyOperation)
            .handle((result, ex) -> {
                if (ex != null) {
                    // 记录异常
                    logger.error("操作失败", ex);
                    // 返回降级结果
                    return "降级结果";
                }
                return result;
            })
            .thenApply(this::postProcess);
    }
}
```

## 性能考虑

1. **线程池配置**
- 根据任务特性选择合适的线程池
- 避免使用默认的 ForkJoinPool.commonPool()
- 监控线程池的使用情况

2. **资源管理**
- 及时关闭线程池
- 避免创建过多的 CompletableFuture 实例
- 注意内存使用

3. **任务调度**
- 合理使用异步方法
- 避免过度嵌套
- 适当使用 thenApplyAsync 和 thenComposeAsync

## 调试技巧

### 1. 使用 whenComplete 进行调试
```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "结果")
    .whenComplete((result, ex) -> {
        System.out.println("处理结果：" + result);
        if (ex != null) {
            ex.printStackTrace();
        }
    });
```

### 2. 异常追踪
```java
public class DebugUtil {
    public static <T> CompletableFuture<T> trace(CompletableFuture<T> future) {
        return future.whenComplete((result, ex) -> {
            if (ex != null) {
                System.err.println("异常堆栈：");
                ex.printStackTrace();
            } else {
                System.out.println("成功结果：" + result);
            }
        });
    }
}
```

## 总结

CompletableFuture 提供了：

1. **强大的异步编程能力**
- 链式操作
- 异常处理
- 任务组合

2. **灵活的任务编排**
- 串行执行
- 并行执行
- 条件执行

3. **完善的异常处理机制**
- 异常恢复
- 异常转换
- 降级处理

使用建议：
1. 合理使用线程池
2. 注意异常处理
3. 避免过度嵌套
4. 注意性能影响

合理使用 CompletableFuture 可以显著提高应用程序的性能和响应性，同时保持代码的可维护性。