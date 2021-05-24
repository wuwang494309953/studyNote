#### ThreadLocal使用

​		**ThreadLocal**在平时项目可以说得上的是个神器了，在一些公共字段的传参可以节省很多逻辑，使用非常方便，但是也有些限制就是在异步的情况下，这次主要就是探讨一下



##### T1:先来个简单的使用

```java
@Slf4j
public class ThreadLocalTest {

    private static final ThreadLocal<String> threadLocal = new ThreadLocal();

    @Test
    public void test1() {
        threadLocal.set("我是周Q");
        sout();
    }

    public void sout() {
        log.info(threadLocal.get());
    }

}
```

输出：

![1621503700794](../../studyNote/图床/截图/1621503700794.png)

符合预期输出，接下来试试多线程的情况



##### T2:多线程版

```java
@Slf4j
public class ThreadLocalTest {

    private static final ThreadLocal<String> threadLocal = new ThreadLocal();

    @Test
    public void test2() {
        threadLocal.set("我是周Q");
        new Thread(this::sout).start();
    }

    public void sout() {
        log.info(threadLocal.get());
    }

}
```

输出：

![1621503992151](../../studyNote/图床/截图/1621503992151.png)

输出找不到值了，emmm，想起**TheadLocal**的兄弟**InheritableThreadLocal**可以解决类似的问题。尝试下



##### T3:InheritableThreadLocal版

```
@Slf4j
public class ThreadLocalTest {

    private static final ThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();

    @Test
    public void test2() {
        inheritableThreadLocal.set("我是周Q");
        new Thread(this::sout).start();
    }

    public void sout() {
        log.info(inheritableThreadLocal.get());
    }

}
```

输出：

![1621504188694](../../studyNote/图床/截图/1621504188694.png)

输出结果正确。

但是这样也有个问题，就是**InheritableThreadLocal**只有在新建线程的时候才有效，如果使用线程池的情况，有可能出现问题

```java
@Slf4j
public class ThreadLocalTest {

    private static final ThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();

    @Test
    public void test3() {

        CountDownLatch countDownLatch = new CountDownLatch(1000);
        ExecutorService executorService = Executors.newFixedThreadPool(10);

        for (int i = 0; i < 1000; i++) {
            inheritableThreadLocal.set("我是周Q" + i);
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    sout();
                    countDownLatch.countDown();
                }
            });
        }
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void sout() {
        log.info("什么鬼: {}", inheritableThreadLocal.get());
    }

}
```

![1621828200065](../../studyNote/图床/截图/1621828200065.png)

可以发现预期结果应该打印0-999，而且每个数字只打印一次，结果与预期不符。这是因为线程池中十个线程没有重复创建，所以**InheritableThreadLocal**失效了

这种情况下阿里开源过一个**TransmittableThreadLocal**专门用来解决这种情况

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>transmittable-thread-local</artifactId>
    <version>2.12.1</version>
</dependency>
```

```java
@Slf4j
public class ThreadLocalTest {

    private static final ThreadLocal<String> transmittableThreadLocal = new TransmittableThreadLocal<>();

    @Test
    public void test3() {

        CountDownLatch countDownLatch = new CountDownLatch(1000);
        // ExecutorService executorService = Executors.newFixedThreadPool(10);
        ExecutorService executorService = TtlExecutors.getTtlExecutorService(Executors.newFixedThreadPool(10));

        for (int i = 0; i < 1000; i++) {
            transmittableThreadLocal.set("我是周Q" + i);
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    sout();
                    countDownLatch.countDown();
                }
            });
        }
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void sout() {
        log.info("什么鬼: {}", transmittableThreadLocal.get());
    }

}
```

![1621834790085](../../studyNote/图床/截图/1621834790085.png)

可以看到最后打印到了999.结果正确。所以在使用了线程池的情况下使用**TransmittableThreadLocal**



#### 总结

- 如果不用异步直接使用**ThreadLocal**即可
- 在不使用线程池的情况下可以使用**InheritableThreadLocal**
- 在使用了线程池的情况下需要使用**TransmittableThreadLocal**。比如在使用到**SpringMVC**中的**@Async**注解时，一般会自定义一个线程池，这种情况下需要使用**TransmittableThreadLocal**，自定义线程池时也需要封装一下