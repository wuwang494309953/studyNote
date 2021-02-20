#### Redis分布式锁

##### 要点一： 避免死锁。(set nx和ex参数)

> **nx:** 如果不存在则Set，保证获取锁的原子性
>
> **ex：**表示过期时间
>
> Redis目前支持参数，同时设置**nx**和**ex**参数可以保证获取锁和设置过期时间同时成功

---

##### 要点二：避免自己的锁被其他人释放

**场景设想：**

> 1. A获取锁并设置超时时间3秒
> 2. B等待获取锁
> 3. A锁3秒后过期释放，此时A业务还未完成
> 4. B获取到锁
> 5. A业务完成，尝试释放锁。(此时会将B获取到的锁释放掉)

**解决：**

> 加锁的时候set加一个线程标识，释放的时候用**lua脚本**先判断标识再释放



---



##### 网上找了一个实现

```java
/**
 * 加锁
 *
 * @param id
 * @return
 */
public boolean lock(String id) {
    Long start = System.currentTimeMillis();
    try {
        for (; ; ) {
            //SET命令返回OK ，则证明获取锁成功
            String lock = jedis.set(LOCK_KEY, id, params);
            if ("OK".equals(lock)) {
                return true;
            }
            //否则循环等待，在timeout时间内仍未获取到锁，则获取失败
            long l = System.currentTimeMillis() - start;
            if (l >= timeout) {
                return false;
            }
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    } finally {
        jedis.close();
    }
}
```

```java
/**
 * 解锁
 *
 * @param id
 * @return
 */
public boolean unlock(String id) {
    String script =
            "if redis.call('get',KEYS[1]) == ARGV[1] then" +
                    "   return redis.call('del',KEYS[1]) " +
                    "else" +
                    "   return 0 " +
                    "end";
    try {
        String result = jedis.eval(script, Collections.singletonList(LOCK_KEY), Collections.singletonList(id)).toString();
        return "1".equals(result) ? true : false;
    } finally {
        jedis.close();
    }
}
```

