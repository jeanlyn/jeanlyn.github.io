---
layout:     post
title:      "\"为什么我的程序抛Null Point Exception\""
subtitle:   " \"why NPE?\""
date:       2015-01-29 12:00:00
author:     "Hux"
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
java.lang.NullPointerException
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