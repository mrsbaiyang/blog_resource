---
title: 闭包的理解
date: 2018-04-15 16:53:11
tags: 语言
categories: 技术

---
&emsp;&emsp;今天被小伙伴问到了闭包，虽然自己原来也看到一些关于闭包的文章，但是发现自己什么都说不出，所以又到了学习的时刻了。
&emsp;&emsp;首先先去百科了下闭包的概念：
```
闭包就是能够读取其他函数内部变量的函数。由于在javascript中，只有函数内部的子函数才能读取局部变量，所以闭包可以理解成“定义在一个函数内部的函数“。在本质上，闭包是将函数内部和函数外部连接起来的桥梁。
```
咱们将这段话拆分一下：
1. 能到读取到其他函数内部变量的函数。 
首先闭包也是一个“函数”，然后才是有“能够读取到其他函数变量”的功能。
2. javascript中理解成“定义在一个函数内部的函数”
3. 将函数内部和外部连接起来的桥梁

----------

将话分成了三部分，果然理解起来就容易多了。我在网上也找到了很多介绍闭包的文章，例如[http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)，但无奈本人是java程序员理解起来还是不够清晰，所以还是谢了自己的java理解版本，如下：
<!-- more -->
```
package com.xstore.tms.center.web;

import java.util.function.Function;

/**
 * @author yang.bai on 2017/12/10.
 */
public class Test1 {

    class Obj1{
        String str = "123";

        public String getStr() {
            return str;
        }

        public void setStr(String str) {
            this.str = str;
        }

        @Override
        public String toString() {
            return "Obj1{" +
                    "str='" + str + '\'' +
                    '}';
        }
    }

    Function<Integer, Integer> functionA = null;

    public void test1(){
        Obj1 b = new Obj1();
        System.out.println(b);
        functionA = (Integer a) ->{
            b.setStr("456");
            System.out.println(b);
            return a++;
        };
    }


    public static void main(String[] args){
        Test1 test1 = new Test1();
        test1.test1();
        test1.functionA.apply(3);

    }
}
```
首先在我们分子这个demo之前我们先思考一个问题：**如果我想在函数A里面访问函数B的局部变量我该如何做到呢？？？**其实我们经常做这种事情，只不过我们不知道它叫闭包罢了。
然后让我们来分析这个demo： 
这个demo里有两个函数，main函数和test1函数，test1函数里面内部函数functionA，functionA访问了局部变量b（我们这里甚至做到了修改）。
```
 public void test1(){
        Obj1 b = new Obj1();
        System.out.println(b);
        functionA = (Integer a) ->{
            b.setStr("456");
            System.out.println(b);
            return a++;
        };
    }
```
所以，按照闭包概念理解的话，functionA就是闭包，起到了连接函数内部和函数外部的作用。回想下，在一个方法体内的匿名函数里访问（甚至修改）函数外部的变量，是不是经常做呢？只不过我们不知道其实这个匿名函数就叫闭包罢了。
我们一开始理解闭包的时候，总是容易被“闭包”这两个字带偏，容易联想到其他关于这两个字的想象，我觉得只需要记住一点就好了。