---
title: 由使用mybatis批量插入功能联想到的  
date: 2018-09-13 13:59:14  
tags: orm  
categories: 技术

---
### 先说说由来
新项目中有批量插入的功能，然而在其他项目中没有找到批量插入的写法，我们都知道，我们使用jdbc的时候是可以指定批量模式的，然而在这里并没有看到我想要的有关批量的设置，所以决定去找找批量使用方式。

### 自己对mybatis的认识过程
查找了不少资料，有细心网友总结的三种方式[https://blog.csdn.net/m0_37981235/article/details/79131493](https://blog.csdn.net/m0_37981235/article/details/79131493)。不过我认为只有其中的第二种，指定批量模式的才是真正的**批量**操作。  

例子中用的SqlSession，SqlSessionTemplate是SqlSession的一个实现，我们可以使用SqlSessionTemplate 来代理以往的DefailtSqlSession完成对数据库的操作，相比于DefaultSqlSession，SqlSessionTemplate是线程安全的，可以被设置成单例的。

<!-- more -->

SqlSession可以理解成jdbc的connection，平时我们写jdbc的使用是使用connection来操作事务的：  
	
	try{
		con.setAutoCommit(false);
		preparedStatement.executeUpdate();
		con.commit();
	}finally{
		con.rollback();
	}

但是在mybatis中我是不建议这样使用的事务的，当然DefaultSqlSession也支持这种类似的操作，查看mybatis的官方文档中，使用TransactionManager管理事务：  

    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
	def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

	TransactionStatus status = txManager.getTransaction(def);
	try {
	  userMapper.insertUser(user);
	}
	catch (MyException ex) {
	  txManager.rollback(status);
	  throw ex;
	}
	txManager.commit(status);

但上述方式还是不够优雅，mybatis如果和IOC容器集成的话，事务都托管给IOC管理，是不支持自己手动管理事务的，看SqlSessionTemplate的源码也能看出来也是这样。
![](https://i.imgur.com/lWYX93K.png)
所以我们在spring的项目中会有类似如下的代码：  

    transactionTemplate.execute(transactionStatus -> {
            supplierStoreSkuList.forEach(supplierStoreSku -> {
                VenderStoreSkuEntity venderStoreSkuEntity = SupplierStoreSkuBizConvertUtils.convertToVenderStoreSkuEntityFromSupplierStoreSku(supplierStoreSku);
                venderStoreSkuMapper.saveSupplierStoreSku(venderStoreSkuEntity);
            });
            return null;
        });

这样的代码就会好看很多了，TransactionTemplate帮我们管理了事务，包括了事务的开启，事务的提交，事务的回滚。我们看execute内部源码其实也是使用的transactionManager只不过做了一层封装而已：  

![](https://i.imgur.com/YCXYI8W.png)

我们再来看看TransactionManager是如何管理事务的，  

![](https://i.imgur.com/NiRlOmG.png)

可以看到，其实内部就是使用的connection，所谓的管理事务，也就是通过connection来做的。

#### 然后我们再聊聊mapper

我们回头再看一下jdbc写的事务：  

![](https://i.imgur.com/GNtIXtS.png)

我们可以看到红线圈起来的部分都是transctionManager做了，那么preparedStatement的事情谁做了呢？很容易就想到这个是mapper做的事情嘛。代码中获取mapper的方式是这样的：  

	UserMapper userMapper = sqlSessionTemplate.getMapper(UserMapper.class);

从这种使用方式可以窥见，肯定是通过sqlSession来实现的，我们来看看sqlSession的实现DefaultSqlSession，  

![](https://i.imgur.com/i1TULEd.png)

里面有Excetor，再看看Excetor是什么样的，  

![](https://i.imgur.com/dfQ94GG.png)

我们又闻到了熟悉的味道，Statement、prepareStatement。至此可以知道了mybatis是如何映射jdbc的了。

#### 再看看我项目中的配置

![](https://i.imgur.com/3F4NAns.png)

其实最开始就已经指定了一个SqlSessionTemplate，所以我们才可以再项目中如下使用而没有问题，  

![](https://i.imgur.com/qKfV4d9.png)

### 最后说一下mybatis batch是如何做的

	applyTransactionTemplate.execute(transactionStatus -> {
		// 指定batch模型的sqlSessionTemplate
        SqlSessionTemplate sqlSessionTemplate =  new SqlSessionTemplate(applySqlSessionFactory, ExecutorType.BATCH);
        VenderSkuApplyMapper venderSkuApplyBatchMapper = sqlSessionTemplate.getMapper(VenderSkuApplyMapper.class);
        for(VenderSkuApplyEntity venderSkuApplyEntity : venderSkuApplyEntities){
            venderSkuApplyBatchMapper.insert(venderSkuApplyEntity);
        }
        sqlSessionTemplate.flushStatements();
        return null;
    });

其实就是指定了单独batch模式的SqlSessionTemplate。

#### 还有一点关于flushStatements的疑问
查看mybatis官方文档，看到了flushStatements这个方法，

![](https://i.imgur.com/wVycNik.png)

我们先看一下jdbc的batch是怎么写的，  

	String sql = "insert into t_user(name, mobile, email) values(?,?,?)";

    try (Connection conn = dataSource.getConnection(); PreparedStatement pstmt = conn.prepareStatement(sql);) {

        List<User> users = this.getUsers();

        for (User user : users) {

           pstmt.setString(1, user.getName());

           pstmt.setString(2, user.getMobile());

           pstmt.setString(3, user.getEmail());

           pstmt.addBatch();

        }

        pstmt.executeBatch();

        conn.commit();

      }

可以看到，其实是有一个addBatch和executeBatch的操作，mybatis的话batch操作会调用BatchExecutor的doUpdate方法，  

![](https://i.imgur.com/rPnJ7oN.png)

handler.batch内部是这样的，  

![](https://i.imgur.com/JpHdxxt.png)

其实内部也是调用的statement的addBatch操作。  

然后我在BatchExecutor的doFlushStatements找到了executeBatch()，  

![](https://i.imgur.com/SIZ5MxD.png)

在BaseExecutor的源码，发现commit中有这个调用，  

![](https://i.imgur.com/kM4u1Hu.png)  

而使用IOC容器管理事务最终会调用commit方法，所以最终结论是如果不是我们自己手动管理事务，是我们不需要自己调用flushStatements的，over。  

还找到了其他人对这个方法使用的解释，[https://blog.csdn.net/y41992910/article/details/53641825](https://blog.csdn.net/y41992910/article/details/53641825)，这个解释也有点奇怪，我没有尝试里面所说的方式，但是感觉这本身就是在一个事务中所做的事情吧，跟flushStatements本身没有关系的吧。