---
layout: post
title: InnoDB 事务隔离级别（Mysql篇）
date: 2017-05-02
tag: Mysql
---

> 前言：
	    Mysql支持MyISAM和InnoDB两种存储引擎，区别在此就不详细说明。此篇是讲述事务，所以切记自己的table是InnDB。此处大坑！

在Mysql InnoDB 中，事务主要有四种隔离级别 

 - Read uncommitted (未提交读)
 -  Read committed (已提交读)  
 - Repeatable read (可重复读)
 - Serializable (可串行化)

在理解四种隔离级别之前，我们需要先了解另外三个名词：

 1. 脏读
 2. 不可重复读
 3. 幻读


#### 脏读
	另一个事务修改了数据，但尚未提交，而本事务中的SELECT会读到这些未被提交的数据。
#### 不重复读
	解决了脏读后，会遇到，同一个事务执行过程中，另外一个事务提交了新数据，因此本事务先后两次读到的数据结果会不一致。
#### 幻读
	解决了不重复读，保证了同一个事务里，查询的结果都是事务开始时的状态（一致性）。但是，如果另一个事务同时提交了新数据，本事务再更新时，就会“惊奇的”发现了这些新数据，貌似之前读到的数据是“鬼影”一样的幻觉。


下面我们就直接来通过实验来看，Mysql Innodb中，不同的事务隔离级别，会出现怎么样的结果。

首先我们开启两个终端，查询当前MySQL的默认隔离级别:

```
SELECT @@global.tx_isolation; //查询全局事务
SELECT @@session.tx_isolation; //查询当前会话事务
```

![这里写图片描述](http://img.blog.csdn.net/20170507213530385?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到，默认的隔离级别是：`REPEATABLE-READ`

### 实验Read uncommitted

我们将会话事务设置为：`Read uncommitted`

```
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
//测试可以不用设置全局事务
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;(这个可以不用设，只设置上面一行就可以了进行测试了)
```

更改完之后，重新查询事务：


![这里写图片描述](http://img.blog.csdn.net/20170507213603388?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到，全局事务已经更改为`Read uncommitted`

然后，我们首先创建一个测试的数据库test_tx，并插入了2条测试数据，如下图：

![这里写图片描述](http://img.blog.csdn.net/20170507213624123?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
然后我们分别开启事务，然后我们在B终端中，插入一条数据，但是不提交，然后在A终端进行数据查询。

![这里写图片描述](http://img.blog.csdn.net/20170507213642166?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到，我们在B终端insert一条数据，但是未进行提交操作(commit)，但是在A事务中，却查询到了。我们称这种现象叫做脏读，在实际开发过程中，我们一般较少使用Read uncommitted隔离级别，这种隔离级别对任何的数据操作都不会进行加锁。

### 实验Read committed

首先我们将会话的事务隔离级别设置为`read committed`

```
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

然后我们用上面相同的方式，进行测试。首先同时将2个终端的事务开启：begin;，然后在B终端中插入一条新的数据insert into test_tx values(4,"Lee");，但是不提交事务(commit)，然后在A终端中，查询数据，如图，我们在A终端中，没有查询到刚才插入的这条数据。

![这里写图片描述](http://img.blog.csdn.net/20170507213702385?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

所以，实验表明，在`Read committed`隔离级别，不会出现脏读的问题。

然后我们继续做实验，看看在`Read committed`隔离级别中，会不会出现不可重复读、幻读的现象。

我们同时打开两个终端的事务，然后在A终端中，查询当前的数据，然后我们在B终端中，将ID为3的数据，name修改为Jeff。然后将B终端的事务提交(commit)，但是A终端不提交事务，在一个事务的生命周期内，然后查询数据，我们查询到了刚才B终端修改过的数据。也就是说，我们在A终端的一个事务周期内(事务未commit)，两次查询，得到的结果是不一样的。



实验表明，在Read committed隔离级别中，存在不可重复读的现象。

我们继续做实验，因为刚才B终端已经将事务提交，所以我们重新打开B终端的事务，然后我们在B终端中，插入(insert)一条ID为5的新数据，并提交事务。然后我们回到A终端，查询数据，我们同样可以查询到刚才B终端新插入的数据。也就是说我们在A终端中，三次查询，得到的结果都是不一样的。
![这里写图片描述](http://img.blog.csdn.net/20170507213816906?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


实验表明，在Read committed隔离级别中，存在幻读的现象。

总结，在Read committed隔离级别中，可以有效解决脏读问题，但是有不可重复读、幻读问题，而不可重复读和幻读的差异主要是，不可重复读主要是针对修改和删除操作、幻读针对插入数据操作。

### 实验Repeatable read

首先我们将隔离级别更改为`Repeatable read`

```
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

然后我们首先实验，在Repeatable read级别中是否存在脏读问题，我们首先同时开启A,B两个终端的事务(begin;)，然后在B终端中，插入一条ID为6的数据，但是不提交事务。然后在A终端中进行数据查询，结果是我们未查询到刚才插入的数据，所以在`Repeatable read`级别中，没有脏读现象。
![这里写图片描述](http://img.blog.csdn.net/20170507213840281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

接着，我们顺着刚才的新插入的数据，然后将B终端的事务进行提交，然后再回到A终端查询数据，依然没有查询到B终端刚才插入的ID为6的数据，以此也就表明，目前Mysql 5.6以上的版本中，Repeatable read级别已经不存在幻读的问题，而之前的版本我并未做测试，后面有时间会在去查一下，mysql是在哪个版本开始解决了幻读问题。

![这里写图片描述](http://img.blog.csdn.net/20170507213858136?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
由于刚才B终端已经提交了事务，所以为了实验是否存在不可重复读的现象，我们重新开启B终端的事务，然后我们将ID为5的name修改为Joy：`update test_tx set name = "Joy" where id = 5;`，同时B终端的事务commit;，然后我们回到A终端进行查询，三次的查询结果都是一致的。所以实验表明，在Repeatable read级别中，不存在不可重复读现象。

![这里写图片描述](http://img.blog.csdn.net/20170507213923324?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
总结，在Repeatable read级别中，脏读、不可重复读、幻读现象都没有。在mysql中，该级别也是默认的事务隔离级别，我们日常在开发中，也是主要使用该隔离级别。

### Serializable

Serializable完全串行化的读，每次读都需要获得表级共享锁，读写相互会相互互斥，这样可以更好的解决数据一致性的问题，但是同样会大大的降低数据库的实际吞吐性能。所以该隔离级别因为损耗太大，一般很少在开发中使用。


### 总结：
| 隔离级别| 脏读| 不可重复读|	幻读|
| ------------- |:-------------:| -----:|
| 未提交读（Read uncommitted）| 可能| 可能|可能|
| 已提交读（Read committed）| 不可能| 可能|可能|
| 可重复读（Repeatable read）| 不可能	| 不可能	|不可能	|
| 可串行化（Serializable ）| 不可能	| 不可能	|不可能	|

