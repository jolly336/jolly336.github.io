---
title: Java代理技术
date: 2018-08-03 00:21:09
tags: Java
categories: Java
---

有些时候，我们不希望有些对象直接被外界所访问，而是通过一个代理对象来访问，已达到控制权限，或者添加一些额外的操作，这种方式就成为代理。代理可以实现解耦，隐藏实现类的具体细节。。。

<img src="http://wanghaoxun.com/java_logo.jpg" width="60%" height="30%">

<!-- more -->

> 注意：以下文章代码来自[codeKK网站](http://a.codekk.com/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)

## 代理

* 静态代理，委托类（被代理类）和代理类实现同一个接口，来达到控制函数的目的。可以在代理类中函数调用委托类函数时，做一些处理。例如，加个权限判断，日志等
* 动态代理，不用手动创建委托类，使用java api来动态生成代理类对象，可以统一对委托类接口里的所有函数集中处理，比如，集中判断，添加日志等等。

## api

* InvocationHandler
```java
    public Object invoke(Object proxy, Method method, Object[] args)
                                          
    * proxy表示通过Proxy.newProxyInstance()生成的代理类对象
    * metho表示代理对象被调用的函数
    * args表示代理对象被调用的函数的参数
    调用代理对象的每个函数最终都是调用了InvocationHandler的invoke函数。invoke函数中我们可以通过对method做一些判断，从而对某些函数特殊处理。
    
    说明：为什么第一个参数要返回来proxy？返回来proxy可以获取一些proxy的信息，后续操作还可以继续使用
```

* Proxy

```java
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
                                          
    * loader表示类加载器
    * interfaces表示委托类的接口，生成代理类时需要实现这些接口
    * h是InvocationHandler实现类对象，负责连接代理类和委托类的中间类

```

## 例子

* 公共接口

```java
public interface Operate {

    public void operateMethod1();

    public void operateMethod2();

    public void operateMethod3();
}

```

* 委托类（被代理类）

```java
public class OperateImpl implements Operate {

    @Override
    public void operateMethod1() {
        System.out.println("Invoke operateMethod1");
        sleep(110);
    }

    @Override
    public void operateMethod2() {
        System.out.println("Invoke operateMethod2");
        sleep(120);
    }

    @Override
    public void operateMethod3() {
        System.out.println("Invoke operateMethod3");
        sleep(130);
    }

    private static void sleep(long millSeconds) {
        try {
            Thread.sleep(millSeconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
````
Operate是一个接口，定了了一些函数，我们要统计这些函数的执行时间。
OperateImpl是委托类，实现Operate接口。每个函数简单输出字符串，并等待一段时间。
动态代理要求委托类必须实现了某个接口，比如这里委托类OperateImpl实现了Operate

* InvocationHandler

```java
public class TimingInvocationHandler implements InvocationHandler {

    private Object target;

    public TimingInvocationHandler() {}

    public TimingInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long start = System.currentTimeMillis();
        Object obj = method.invoke(target, args);
        System.out.println(method.getName() + " cost time is:" + (System.currentTimeMillis() - start));
        return obj;
    }
}
```

target属性表示委托类对象。

InvocationHandler是负责连接代理类和委托类的中间类必须实现的接口。其中只有一个invoke(...)函数要去实现。

* 通过 Proxy 类静态函数生成代理对象

```java
public class Main {
    public static void main(String[] args) {
        // create proxy instance
        TimingInvocationHandler timingInvocationHandler = new TimingInvocationHandler(new OperateImpl());
        Operate operate = (Operate)(Proxy.newProxyInstance(Operate.class.getClassLoader(), new Class[] {Operate.class},
                timingInvocationHandler));

        // call method of proxy instance
        operate.operateMethod1();
        System.out.println();
        operate.operateMethod2();
        System.out.println();
        operate.operateMethod3();
    }
}
```

这里我们先将委托类对象new OperateImpl()作为TimingInvocationHandler构造函数入参创建timingInvocationHandler对象；
然后通过Proxy.newProxyInstance(…)函数新建了一个代理对象，实际代理类就是在这时候动态生成的。我们调用该代理对象的函数就会调用到timingInvocationHandler的invoke函数(是不是有点类似静态代理)，而invoke函数实现中调用委托类对象new OperateImpl()相应的 method(是不是有点类似静态代理)。

## 关于代理类

想要查看生成的代理类，需要借助系统api（ProxyGenerator.generateProxyClass(proxyClassName, interfaces)）来生成class，输出到本地，借助jd-gui查看接口。如何得到这些生成的代码请见[ProxyUtils](https://github.com/android-cn/android-open-project-demo/blob/master/java-dynamic-proxy/src/com/codekk/java/test/dynamicproxy/util/ProxyUtils.java)。

### 基本原理

* 先调用getProxyClass(loader, interfaces)得到动态代理类，然后将InvocationHandler作为代理类构造函数入参新建代理类对象。这里创建代理对象使用了反射， `Constructor cons = cl.getConstructor(constructorParams); (Object) cons.newInstance(new Object[] { h })`
* 入参 interfaces 检验，包含3部分，1）是否在入参指定的 ClassLoader 内；2）是否是 Interface；3）interfaces 中是否有重复
* 动态生成代理类的字节码，最终调用 sun.misc.ProxyGenerator.generateClassFile() 得到代理类相关信息，返回来class字节数组，可写入 DataOutputStream 实现
*  native 层实现，虚拟机加载代理类并返回其类对象

## 使用场景

例如：J2EE Web 开发中 Spring 的 AOP(面向切面编程) 特性

比如在 Dao 中，每次数据库操作都需要开启事务，而且在操作的时候需要关注权限。一般写法是在 Dao 的每个函数中添加相应逻辑，造成代码冗余，耦合度高。
使用动态代理前伪代码如下：

```java
Dao {
    insert() {
        判断是否有保存的权限；
        开启事务；
        插入；
        提交事务；
    }

    delete() {
        判断是否有删除的权限；
        开启事务；
        删除；
        提交事务；
    }
}
```

使用动态代理的伪代码如下：
```java
// 使用动态代理，组合每个切面的函数，而每个切面只需要关注自己的逻辑就行，达到减少代码，松耦合的效果
invoke(Object proxy, Method method, Object[] args)
                    throws Throwable {
    判断是否有权限；
    开启事务；
    Object ob = method.invoke(dao, args)；
    提交事务；
    return ob; 
}
```

## 参考

* [codeKK-公共技术点之 Java 动态代理](http://a.codekk.com/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)
* [本地生成代理类的函数，可使用jd-gui查看class](https://github.com/android-cn/android-open-project-demo/blob/master/java-dynamic-proxy/src/com/codekk/java/test/dynamicproxy/util/ProxyUtils.java)
* [InvocationHandler中invoke方法中的第一个参数proxy的用途](https://blog.csdn.net/bu2_int/article/details/60150319)

