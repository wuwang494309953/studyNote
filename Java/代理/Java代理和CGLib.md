#### Java中的动态代理

​		Java中的代理代理一般指的是**JDK代理**和**CGLib代理**，常见的场景有**Mybatis**的**mapper**是代理实现的，**Spring**中的**@Transactional**也是通过将类设置为代理类实现的

​		所以，记录下写两个简单的代理是怎么实现

---

###### 具体代码

1. 首先**JDK代理**应该初始化需要通过接口，所以必须先创建一个接口和一个实现类，内容就只是打印一个日志

   ```java
   public interface JDKInterface {
       void test();
   }
   
   public class JDKProxy implements JDKInterface {
       @Override
       public void test() {
           System.out.println("我是大神仙");
       }
   }
   ```

2. 创建JDK的代理类

    ```java
   @Test
   public void test1() {
       InvocationHandler invocationHandler = new InvocationHandler() {
           @Override
           public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
               //反射调用具体的实现类，所以这里需要给实现类实例
               return method.invoke(new JDKProxy(), args);
           }
       };
       //这里只能给接口类，不能用实现类接口
       JDKInterface proxy = (JDKInterface) Proxy.newProxyInstance(invocationHandler.getClass().getClassLoader(),
                  		   new Class[]{JDKInterface.class},
                          invocationHandler);
       proxy.test();
   
   }
   ```

3. 创建CGLib代理类

   ```java
   @Test
   public void test2() {
       Enhancer enhancer = new Enhancer();
       //可以用接口类也可以用实现类
       enhancer.setSuperclass(JDKProxy.class);
       enhancer.setCallback(new MethodInterceptor() {
           @Override
           public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
               //反射调用必须要用实现类
               return method.invoke(new JDKProxy(), objects);
           }
       });
       //跟上边保持一致即可
       JDKProxy proxy = (JDKProxy) enhancer.create();
       proxy.test();
   }
   ```
   
   
   
   ##### 总结
   
   ​		可以发现实现一个动态代理类还是比较简单，而使用方便**CGLib代理**比起**JDK代理**可以少创建一个接口类会方便一些
   
   