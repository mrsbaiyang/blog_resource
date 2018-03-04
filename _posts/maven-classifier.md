---
title: maven-classifier
date: 2018-02-09 12:17:11
tags: maven
categories: 技术
---
&emsp;&emsp;在一个大型公司当中，对项目的最最主要的要求就是稳定，任何的改动尽可能的是其影响的范围缩小到最小。改动其实是难免的，只要有需求的任何变化，项目或多或少都会有改动。maven作为一个项目的管理工具，他使用model来实现分层，理想的情况下每个模块内部的类只影响到自己个module本身。例如我们的项目，原则上至少会分成以下几个module，dao、biz、rpc、service、api。  
&emsp;&emsp;假设我们我们的业务中有一个student表，所以我们dao module会有一个StudentEntity的pojo，如果严格按照原则的话，那我们至少有三个pojo（service是api的一个实现算一个）来表达这个student业务，需要做至少两次convert才最终到service提供出去服务（暂时不包括rpc module），这样才做到了控制影响范围。但是这样带来的一个后果就是，我们的每个module里都会有大量的convert代码，然而其实绝大多数每个module 的pojo之间差别是很小的，甚至是没有任何差别的。所以我们一般做项目的时候都会做一些权衡，有一些东西是可以贯穿整个项目的，比如说自定义的BusinessException、一些constants、一些enum。但是项目module之间的依赖是顺序传递的，如下图:  
```
	+---------+     +-----+     +-----+
	| service | --> | biz | --> | dao |
	+---------+     +-----+     +-----+
	  |               |
	  |               |
	  v               v
	+---------+     +-----+
	|   api   |     | rpc |
	+---------+     +-----+
```
<!-- more -->
我们如何做到使最外层的依赖，让里层获取到呢？例如，api 里面的StudentTypeEnum如何让biz使用呢？  
&emsp;&emsp;在这里我们定下来回想一下，我们分层的目的是什么？是为了，当我们有任何改动的时候只影响到本层内，不会扩散到其他层。所以我们的依赖是顺序的，里层引用不到和它隔层的类，但是如果我们打破这个顺序，这样每个module之间都可以相互引用，这样其实module也失去了存在的意义。但是我们的需求又是实际存在的，这好像是个矛盾的命题，貌似是不能兼顾双方的。  
&emsp;&emsp;但maven给了我们解决方案，以往我们使用maven管理jar的时候，只是应用到版本号，但其实在版本号下面还有classifier的分级，同一个版本号但是classifier不同其jar包的内容也是不同的。所以我们如果要解决这个问题，就需要将api打包成两个不同的classifier给其引用，例如classifier分别为all和model，service可以引用all的，biz给其引用model的，这样既防止了biz层的乱用，又使其可以不用写大量重复的convert代码，不过至于哪些类应该给里层使用，这个应该考虑好，也不要过度放开。下面是一些打包的代码，可以借鉴：
```
<plugin>
                <artifactId>maven-jar-plugin</artifactId>
                <executions>
                    <execution>
                        <id>all</id>
                        <goals><goal>jar</goal></goals>
                        <phase>package</phase>
                        <configuration>
                            <classifier>all</classifier>
                            <includes>
                                <include>**</include>
                            </includes>
                        </configuration>
                    </execution>
                    <execution>
                        <id>model</id>
                        <goals><goal>jar</goal></goals>
                        <phase>package</phase>
                        <configuration>
                            <classifier>model</classifier>
                            <includes>
                                <include>**/apply/soa/api/exceptions/**</include>
                                <include>**/apply/soa/api/service/**/exceptions/**</include>
                                <include>**/common/**</include>
                            </includes>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```