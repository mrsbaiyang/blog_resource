---
title: Proxy  
date: 2018-09-08 11:13:14  
tags: java  
categories: 技术

---
### 先讲讲由来
最近项目中要写一个统一的异常拦截，并且打日志的功能，所以很自然的想到了切面，并且很自然想到了spring的aop。  

### 先讲一个spring的aop
spring有好几种方式使用apo：  

1. 使用MethodInterceptor
2. 使用声明式aop（<aop:config>）
3. 使用aop注解（@Aspect）

再具体的细节我就不讲了，有很多优秀的文章都讲了如何使用[https://www.cnblogs.com/jacksonshi/p/5863313.html](https://www.cnblogs.com/jacksonshi/p/5863313.html)，
看到这里的时候对aspect产生了兴趣。

### 再讲讲aspect
aspect翻译为切面，和面向对象编程一样有面向切面编程（AOP），目前有一个非常出名的面向切面编程的框架AspectJ，它定义了自己的切面语法，还有自己的织入工具。对于spring来说，其实也是用AspectJ的预发定义了切面的规则，但它背后使用了cglib的动态代理，没有使用AspectJ的织入代码。这里有一篇很不错的文章介绍了AspectJ[https://juejin.im/entry/5a40abb16fb9a0451e400886](https://juejin.im/entry/5a40abb16fb9a0451e400886)  

<!-- more -->

### 然后了解下cglib
cglib本身是一个代码生成库，可以生成字节码。所谓cglib的动态代理就是分析了aspect定义的规则，然后生成了对应的代理类，加载到内存中使用。  
网上总是拿jdk的动态代理和cglib的动态代理来比较，这里我们也简单的介绍下：  

**JDK动态代理只能对实现了接口的类生成代理，而不能针对类  
CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法（继承）**  

talk is cheap,let's code.下面是我写的一个jdk代理的demo：  

    public class DynamicProxyTest {

    public static void main(String[] args){
        //生成$Proxy0的class文件
//        System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");

        IHello iHello = (IHello) Proxy.newProxyInstance(IHello.class.getClassLoader(),
                new Class[]{IHello.class} ,
                new MyInvocationHandler(new Hello()));
        iHello.sayHello();
    }

    public static interface IHello{
        void sayHello();
    }

    public static class Hello implements IHello{

        @Override
        public void sayHello() {
            System.out.println("hello world");
        }
    }

    public static class MyInvocationHandler implements InvocationHandler {
        Object javaProxy;
        public MyInvocationHandler(Object javaProxy) {
            this.javaProxy = javaProxy;
        }
        private void aopMethod() {
            System.out.println("before method");
        }
        //继承方法，代理时实际执行的犯法，如果要实现原方法，则需要调用method.invoke(javaProxy, args)，这里还调用了一个aopMethod(),可以类比于Spring中的切面before注解。
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            aopMethod();
            return method.invoke(javaProxy, args);
        }
    }
	}
这是下面是jdk生成的代理类反编译后的结果：

    public class DynamicProxyTest$MyInvocationHandler
        implements InvocationHandler
	{
    Object javaProxy;

    public DynamicProxyTest$MyInvocationHandler(Object javaProxy)
    {
        this.javaProxy = javaProxy;
    }

    private void aopMethod()
    {
        System.out.println("before method");
    }

    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable
    {
        aopMethod();
        return method.invoke(this.javaProxy, args);
    }
	}
可以看到InvocationHandler的写法还是最终生成的代理类的代码其实和我们写的静态代理的方式是一样的。

接下来我们看看cglib的：  
    public class DynamicProxyCglibTest {

    public static void main(String[] args){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(HelloImpl.class);
        enhancer.setCallback(new HelloMethodIntercepter());
        HelloImpl hello = (HelloImpl)enhancer.create();
        hello.sayHello();
    }


    public static class HelloImpl {

        public void sayHello() {
            System.out.println("hello world");
        }
    }

    public static class HelloMethodIntercepter implements MethodInterceptor {
        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            System.out.println("before:" + method.getName());
            Object object = methodProxy.invokeSuper(o, objects);
            System.out.println("after:" + method.getName());
            return object;
        }
    }
	}

这是下面是jdk生成的代理类反编译后的结果：  

    public class DynamicProxyCglibTest$HelloMethodIntercepter
        implements MethodInterceptor
	{
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy)
            throws Throwable
    {
        System.out.println("before:" + method.getName());
        Object object = methodProxy.invokeSuper(o, objects);
        System.out.println("after:" + method.getName());
        return object;
    }
	}
可以看到其实都是大同小异的。我们成天总是将动态代理、静态代理的区别什么的，其实从最根本上说是没什么区别的，都是生成了另一个代理实例，来调用我们真正的实例。只不过动态代理带给我们的方便是不需要我们自己写这个实例而已，由代码生成器去生成罢了。如果是统一的规则应用在很多的类方法，这个时候所谓动态代理就显示了威力，不需要我们去一个个写代理类了，反而言之，如果只有一个类的话，动态代理也就没什么必要了。  

这里顺便说一下，经常讨论的动态代理的性能不好什么的，我原来的以为是动态代理每次去做都需要生成字节码的，今天了解后才知道不是，根本就是编译的时候就生成放在jvm中了。那么问题来了，如果不是字节码的性能损耗，对于代码来说就跟我们平时写个包装类，也只是多了一层调用而已，不应该有什么损耗的吧。所以所谓动态代理的性能损耗不是指的是代理本身，而是指的动态代理用的反射的损耗。但是一般在spring管理下的实例也是单例的，生成的代理实例应该也是单例，所以也不存在newInstance的损耗，那最终应该就是invoke的损耗了吧。  

上面是我的推断，后续还要确认，如果异议欢迎来议。

最后说一下，jdk从1.7之后代理的性能已经超过了cglib了，所以网上说的什么cglib完爆jdk那是很多年前的事情了。