---
title: PUBG External Support for 3.7.20.1
date: 2017-12-29
tags: "hack"
categories: "hack"
---


今天执行hive任务：

```
ALTER TABLE supplychain.test_log DROP IF EXISTS PARTITION (date=2018122);
```

遇到一个神奇的错误

```
Failed with java.sql.SQLException: Error while compiling statement: FAILED: ParseException line 1:33 mismatched input 'DROP' expecting KW_EXCHANGE near 'test_log' in alter exchange partition
  at org.apache.hive.jdbc.Utils.verifySuccess(Utils.java:120)
  at org.apache.hive.jdbc.Utils.verifySuccessWithInfo(Utils.java:108)
  at org.apache.hive.jdbc.HiveStatement.execute(HiveStatement.java:233)
  at com.xiaomi.data.arbiter.worker.hive.Executor$$anonfun$execute$1$$anonfun$apply$mcV$sp$4.apply(Executor.scala:131)
  at com.xiaomi.data.arbiter.worker.hive.Executor$$anonfun$execute$1$$anonfun$apply$mcV$sp$4.apply(Executor.scala:126)
  at scala.collection.immutable.List.foreach(List.scala:383)
  at com.xiaomi.data.arbiter.worker.hive.Executor$$anonfun$execute$1.apply$mcV$sp(Executor.scala:126)
  at com.xiaomi.data.arbiter.worker.hive.Executor$$anonfun$execute$1.apply(Executor.scala:106)
  at com.xiaomi.data.arbiter.worker.hive.Executor$$anonfun$execute$1.apply(Executor.scala:106)
  at scala.concurrent.impl.Future$PromiseCompletingRunnable.liftedTree1$1(Future.scala:24)
  at scala.concurrent.impl.Future$PromiseCompletingRunnable.run(Future.scala:24)
  at scala.concurrent.impl.ExecutionContextImpl$AdaptedForkJoinTask.exec(ExecutionContextImpl.scala:121)
  at scala.concurrent.forkjoin.ForkJoinTask.doExec(ForkJoinTask.java:260)
  at scala.concurrent.forkjoin.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1339)
  at scala.concurrent.forkjoin.ForkJoinPool.runWorker(ForkJoinPool.java:1979)
  at scala.concurrent.forkjoin.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:107)
```

然后把任务改成

```
USE supplychain; ALTER TABLE test_log DROP IF EXISTS PARTITION (date=2018122);
```


就正常了，真是神奇。。。
