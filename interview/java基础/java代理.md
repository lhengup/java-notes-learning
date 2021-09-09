## 代理模式

使用代理对象来代替真实对象的访问，这样就可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能。

**代理模式的主要作用是扩展目标对象的功能，比如说在目标对象的某个方法执行前后你可以增加一些自定义的操作。**

分类：静态代理和动态代理



## 静态代理

**静态代理中，我们对目标对象的每个方法的增强都是手动完成的（\*后面会具体演示代码\*），非常不灵活（\*比如接口一旦新增加方法，目标对象和代理对象都要进行修改\*）且麻烦(\*需要对每个目标类都单独写一个代理类\*)。** 实际应用场景非常非常少，日常开发几乎看不到使用静态代理的场景。

上面我们是从实现和应用角度来说的静态代理，从 JVM 层面来说， **静态代理在编译时就将接口、实现类、代理类这些都变成了一个个实际的 class 文件。**

静态代理实现步骤:

1. 定义一个接口及其实现类；
2. 创建一个代理类同样实现这个接口
3. 将目标对象注入进代理类，然后在代理类的对应方法调用目标类中的对应方法。这样的话，我们就可以通过代理类屏蔽对目标对象的访问，并且可以在目标方法执行前后做一些自己想做的事情。



## 动态代理



**从 JVM 角度来说，动态代理是在运行时动态生成类字节码，并加载到 JVM 中的。**

说到动态代理，Spring AOP、RPC 框架应该是两个不得不的提的，它们的实现都依赖了动态代理。

**动态代理在我们日常开发中使用的相对较小，但是在框架中的几乎是必用的一门技术。学会了动态代理之后，对于我们理解和学习各种框架的原理也非常有帮助。**

就 Java 来说，动态代理的实现方式有很多种，比如 **JDK 动态代理**、**CGLIB 动态代理**等等。

### JDK的动态代理

1. 简介

   ​	**在 Java 动态代理机制中 `InvocationHandler` 接口和 `Proxy` 类是核心。**

   ​	`Proxy` 类中使用频率最高的方法是：`newProxyInstance()` ，这个方法主要用来生成一个代理对象。

   ```java
       public static Object newProxyInstance(ClassLoader loader,
                                             Class<?>[] interfaces,
                                             InvocationHandler h)
           throws IllegalArgumentException
       {
           ......
       }
   ```

   ​	这个方法一共有 3 个参数：

   1. **loader** :类加载器，用于加载代理对象。
   2. **interfaces** : 被代理类实现的一些接口；
   3. **h** : 实现了 `InvocationHandler` 接口的对象；

   要实现动态代理的话，还必须需要实现`InvocationHandler` 来自定义处理逻辑。 当我们的动态代理对象调用一个方法时候，这个方法的调用就会被转发到实现`InvocationHandler` 接口类的 `invoke` 方法来调用。

   ```
   public interface InvocationHandler {
   
       /**
        * 当你使用代理对象调用方法的时候实际会调用到这个方法
        */
       public Object invoke(Object proxy, Method method, Object[] args)
           throws Throwable;
   }
   ```

   `invoke()` 方法有下面三个参数：

   1. **proxy** :动态生成的代理类
   2. **method** : 与代理类对象调用的方法相对应
   3. **args** : 当前 method 方法的参数

   也就是说：**你通过`Proxy` 类的 `newProxyInstance()` 创建的代理对象在调用方法的时候，实际会调用到实现`InvocationHandler` 接口的类的 `invoke()`方法。** 你可以在 `invoke()` 方法中自定义处理逻辑，比如在方法执行前后做什么事情。



2.  **JDK动态代理的步骤**：
   1. 定义一个接口及其实现类；
   2. 自定义 `InvocationHandler` 并重写`invoke`方法，在 `invoke` 方法中我们会调用原生方法（被代理类的方法）并自定义一些处理逻辑；
   3. 通过 `Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)` 方法创建代理对象；

### 基于CGLIB的动态代理

1. 简介

   **JDK 动态代理有一个最致命的问题是其只能代理实现了接口的类。**

   **为了解决这个问题，我们可以用 CGLIB 动态代理机制来避免。**

   [CGLIB](https://github.com/cglib/cglib)(*Code Generation Library*)是一个基于[ASM](http://www.baeldung.com/java-asm)的字节码生成库，它允许我们在运行时对字节码进行修改和动态生成。CGLIB 通过继承方式实现代理。很多知名的开源框架都使用到了[CGLIB](https://github.com/cglib/cglib)， 例如 Spring 中的 AOP 模块中：如果目标对象实现了接口，则默认采用 JDK 动态代理，否则采用 CGLIB 动态代理。

   **在 CGLIB 动态代理机制中 `MethodInterceptor` 接口和 `Enhancer` 类是核心。**

   你需要自定义 `MethodInterceptor` 并重写 `intercept` 方法，`intercept` 用于拦截增强被代理类的方法。

   ```
   public interface MethodInterceptor
   extends Callback{
       // 拦截被代理类中的方法
       public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args,
                                  MethodProxy proxy) throws Throwable;
   }
   ```

   1. **obj** :被代理的对象（需要增强的对象）
   2. **method** :被拦截的方法（需要增强的方法）
   3. **args** :方法入参
   4. **methodProxy** :用于调用原始方法

   你可以通过 `Enhancer`类来动态获取被代理类，当代理类调用方法的时候，实际调用的是 `MethodInterceptor` 中的 `intercept` 方法。

2. 步骤

   1. 定义一个类；
   2. 自定义 `MethodInterceptor` 并重写 `intercept` 方法，`intercept` 用于拦截增强被代理类的方法，和 JDK 动态代理中的 `invoke` 方法类似；
   3. 通过 `Enhancer` 类的 `create()`创建代理类；



### JDK和CGLIB区别

1. **JDK 动态代理只能只能代理实现了接口的类或者直接代理接口，而 CGLIB 可以代理未实现任何接口的类。** 另外， CGLIB 动态代理是通过生成一个被代理类的子类来拦截被代理类的方法调用，因此不能代理声明为 final 类型的类和方法。
2. 就二者的效率来说，大部分情况都是 JDK 动态代理更优秀，随着 JDK 版本的升级，这个优势更加明显。





## 静态代理与动态代理的区别

1. **灵活性** ：动态代理更加灵活，不需要必须实现接口，可以直接代理实现类，并且可以不需要针对每个目标类都创建一个代理类。另外，静态代理中，接口一旦新增加方法，目标对象和代理对象都要进行修改，这是非常麻烦的！
2. **JVM 层面** ：静态代理在编译时就将接口、实现类、代理类这些都变成了一个个实际的 class 文件。而动态代理是在运行时动态生成类字节码，并加载到 JVM 中的。

