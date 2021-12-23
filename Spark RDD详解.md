# Spark RDD详解

spark的RDD是spark的编程模型。其中RDD中采用了隐式转换，定义了大量算子。

隐式转换 :RDD对象如果是(k,v)类型可直接调用PairRDDFunctions中的reduceByKey方法，隐式转换只针对类型作转换，不对方法名作转换

转换代码：object RDD中

```scala
 implicit def rddToPairRDDFunctions[K, V](rdd: RDD[(K, V)])
    (implicit kt: ClassTag[K], vt: ClassTag[V], ord: Ordering[K] = null): PairRDDFunctions[K, V] = {
    new PairRDDFunctions(rdd)
  }
```

task中会执行RDD的iterator方法以及compute方法，实现正真的计算逻辑。

```scala
 final def iterator(split: Partition, context: TaskContext): Iterator[T] = {
    if (storageLevel != StorageLevel.NONE) {
      getOrCompute(split, context)
    } else {
      computeOrReadCheckpoint(split, context)
    }
  }
```

如map的compute

```scala
override def compute(split: Partition, context: TaskContext): Iterator[U] =
  f(context, split.index, firstParent[T].iterator(split, context))
```

```
protected[spark] def firstParent[U: ClassTag]: RDD[U] = {
  dependencies.head.rdd.asInstanceOf[RDD[U]]
} // 表示父rdd的数据
```

学习Rdd收获

知道RDD执行逻辑信息，能自定义算子，自定义RDD,实现性能优化

