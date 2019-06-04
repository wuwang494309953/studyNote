### Volatile

​	**众所周知** `volatile` 作为Java关键字之一，用来修饰对象可以获得可见性，即不同线程访问该对象时保证获取的是最新的对象。

​	顺带复习一下，并发变成中一般有三个概念，分别是**原子性**,**可见性**和**有序性**。

```java
@Slf4j
public class VolatileDemo {

    public static void main(String[] args) throws InterruptedException {

        VolatileTestObj obj = new VolatileTestObj();
        new Thread(() -> {
            while (true) {
                obj.put("time：" + System.currentTimeMillis());
            }
        }).start();

        new Thread(() -> {
            while (true) {
                log.info(obj.get());
            }
        }).start();

    }

    static class VolatileTestObj {

        private String value = null;
        private boolean hasNewValue = false;

        public void put(String value) {
            while (hasNewValue) {
                // 等待，防止重复赋值
            }
            this.value = value;
            hasNewValue = true;
        }

        public String get() {
            while (!hasNewValue) {
                // 等待，防止获取到旧值
            }
            String value = this.value;
            hasNewValue = false;
            return value;
        }
    }
}
```

测试代码中 `hasNewValue` 不使用 `volatile`  关键字时，很容易阻塞在循环等待中。

> emmm.道理看起来好像很简单，本来打算自己想个测试代码出来的，结果半天测试不出来理想情况，无奈百度之，真面向百度编程.参考链接: https://juejin.im/post/5956222cf265da6c483cccf1