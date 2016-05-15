---
layout:     post
title:      "\"为什么我的程序抛Null Point Exception\""
subtitle:   " \"why NPE?\""
date:       2016-05-15 17:36:00
author:     "jeanlyn"
header-img: "img/post-bg-2015.jpg"
tags:
    - spark
---
## 起因

前些天,一个同事问我,发了一段spark的业务代码给我,问, "为什么这段代码抛出了Null Point Exception", 分析了一下后,觉得也挺有趣的,就发出来给大家吧.

## 代码以及分析
大概的代码如下

```java
List<scala.collection.immutable.HashMap> retList = row.getList(0)
if(null == retList || retList.isEmpty()){ 
 ....
}

```
同事说`retList.isEmpty()`这里抛出了异常

我在看了一下具体的异常

```java
java.lang.NullPointerException	at scala.collection.convert.Wrappers$IterableWrapperTrait$class.isEmpty(Wrappers.scala:25)	at scala.collection.convert.Wrappers$SeqWrapper.isEmpty(Wrappers.scala:64)	at com.gf.spark.module.CustomerLoginProcess$2.call(CustomerLoginProcess.java:77)	at com.gf.spark.module.CustomerLoginProcess$2.call(CustomerLoginProcess.java:70)	at org.apache.spark.api.java.JavaPairRDD$$anonfun$toScalaFunction$1.apply(JavaPairRDD.scala:1027)	at scala.collection.Iterator$$anon$11.next(Iterator.scala:328)	at scala.collection.Iterator$$anon$14.hasNext(Iterator.scala:389)	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)	at org.apache.spark.rdd.PairRDDFunctions$$anonfun$saveAsHadoopDataset$1$$anonfun$13$$anonfun$apply$6.apply$mcV$sp(PairRDDFunctions.scala:1108)	at org.apache.spark.rdd.PairRDDFunctions$$anonfun$saveAsHadoopDataset$1$$anonfun$13$$anonfun$apply$6.apply(PairRDDFunctions.scala:1108)	at org.apache.spark.rdd.PairRDDFunctions$$anonfun$saveAsHadoopDataset$1$$anonfun$13$$anonfun$apply$6.apply(PairRDDFunctions.scala:1108)	at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1285)	at org.apache.spark.rdd.PairRDDFunctions$$anonfun$saveAsHadoopDataset$1$$anonfun$13.apply(PairRDDFunctions.scala:1116)	at org.apache.spark.rdd.PairRDDFunctions$$anonfun$saveAsHadoopDataset$1$$anonfun$13.apply(PairRDDFunctions.scala:1095)	at org.apache.spark.scheduler.ResultTask.runTask(ResultTask.scala:63)	at org.apache.spark.scheduler.Task.run(Task.scala:70)	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:213)	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)	at java.lang.Thread.run(Thread.java:745)
```
其实真正抛出异常的是scala类库里面的isEmpty抛出来的.这时候我觉得需要看下spark里面的`Row.getList`是什么东西(spark-1.5)

```scala
def getList[T](i: Int): java.util.List[T] = {
  scala.collection.JavaConversions.seqAsJavaList(getSeq[T](i))
}
```

看到getList调用`JavaConversions.seqAsJavaList`返回`new SeqWrapper(seq)`,所以当`getSeq[T](i)`返回为null的时候,getList返回的也并不是空(为SeqWrapper(null)),在调用`isEmpty`方法的时候就相当于null.isEmpty,这样抛出null point Exception也就不奇怪了.那么,这段代码的正确姿势是

```java
if(!row.isNullAt(0)) {
    List<scala.collection.immutable.HashMap> restList = row.getList(0)
} else {
    .....
}

```
