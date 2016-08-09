---
layout: post
category : Spark
tagline: 
tags : [dijkstra, graphx, scala]
---
{% include JB/setup %}

还是老东西，Dijkstra算法在spark下的实现，这次使用了scala语言（spark的原生语言）。主要是因为使用Java开发spark程序写法过于吃力，而且相关的example比起scala来相对较少。权衡之下，最终还是决定稍微学一下scala，以后还是用scala来开发spark。这样一方面可以找到更多的资源来解决问题，另一方面方便排错。

主要用到了graphx中构建图的方法和Pregel方法这两个主要内容，好了，话不多说，展示结果。

输入：(A B 10)表示从A指向B的路径长度为10

    A B 10
    A C 5
    B C 2
    B D 1
    C B 3
    C D 9
    C E 2
    D E 4
    E A 7
    E D 6
    
<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/3.png" target="_blank"> 
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/3.png" style="max-width:100%;"></a>   

为了方便构造图，将输入改为
1、vertex.txt，点的构造，(顶点id,顶点名称):

    1 A
    2 B
    3 C
    4 D
    5 E
    
<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/1.png" target="_blank"> 
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/1.png" style="max-width:100%;"></a>   

2、edge.txt，边的构造，(起点id,终点id,边的权值):

    1 2 10
    1 3 5
    2 3 2
    2 4 1
    3 2 3
    3 4 9
    3 5 2
    4 5 4
    5 1 7
    5 4 6
    
<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/2.png" target="_blank"> 
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/2.png" style="max-width:100%;"></a>   

构造图后的结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/4.png" target="_blank"> 
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/4.png" style="max-width:100%;"></a>   

计算出最短路径的结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/5.png" target="_blank"> 
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-07-04/5.png" style="max-width:100%;"></a>   

最后，代码如下：

```java
package com.ideal.netcare.test

import org.apache.spark.graphx._
import org.apache.spark.rdd.RDD
import org.apache.spark.{Logging, SparkConf, SparkContext}

/**
 * Created by syf on 2016/6/24.
 */
object Dijkstra extends Logging {

  def main(args: Array[String]) {
    val conf = new SparkConf()
    GraphXUtils.registerKryoClasses(conf)
    conf.setAppName("Spark Dijkstra").setMaster("spark://spark-master:7077").set("spark.executor.memory", "512m")
    val sc = new SparkContext(conf)
    //本地打包的jar的位置  必备
    sc.addJar("target/scala-2.10/spark-test_2.10-1.0.jar")

    val fileV = sc.textFile("hdfs://spark-master:9000/syf/spark/data/dijkstra/vertex.txt")
    //(Double,String,String)  指的是  (最短路径长度, 顶点的名称, 最短路径上的点)
    val vertexes: RDD[(VertexId, (Double, String, String))] = fileV.map { line =>
      val parts = line.split("\\s+")
      (parts(0).toLong, (Double.PositiveInfinity, parts(1), ""))
    }.cache()

    val fileE = sc.textFile("hdfs://spark-master:9000/syf/spark/data/dijkstra/edge.txt")
    val edges: RDD[Edge[String]] = fileE.map { line =>
      val parts = line.split("\\s+")
      Edge(parts(0).toLong, parts(1).toLong, parts(2))
    }

    val graph = Graph(vertexes, edges)

    // 输出Graph的信息
    //    graph.vertices.collect().foreach(println(_))
    //    graph.triplets.map(triplet => triplet.srcAttr._2 + "----->" + triplet.dstAttr._2 + "    attr:" + triplet.attr).collect().foreach(println(_))

    val sourceId: VertexId = 1 // The ultimate source
    var sourceName = ""
    vertexes.filter(_._1==sourceId).collect().foreach(rdd=> sourceName=rdd._2._2)

    println("sourceName=" + sourceName)

    val initialGraph = graph.mapVertices((id, attr) => if (id == sourceId) (0.0, attr._2, sourceName) else (Double.PositiveInfinity, attr._2, sourceName))
    initialGraph.cache()

    //(Double.PositiveInfinity, "", "")为初始消息，用于调用第一次Vertex Program，以实现初始化。
    val sssp = initialGraph.pregel((Double.PositiveInfinity, "", ""))(
      (id, dist, newDist) => {
        if (dist._1 < newDist._1) dist else (newDist._1, dist._2, newDist._3)
      }, // Vertex Program
      triplet => {
        // Send Message
        if (triplet.srcAttr._1 + triplet.attr.toDouble < triplet.dstAttr._1) {
          Iterator((triplet.dstId, ((triplet.srcAttr._1 + triplet.attr.toDouble), triplet.dstAttr._2, triplet.srcAttr._3 + "->" + triplet.dstAttr._2)))
        } else {
          Iterator.empty
        }
      },
      (a, b) => {
        if (a._1 < b._1) a else b
      } // Merge Message
    )
    println(sssp.vertices.map(_._2).collect.mkString("\n"))
  }
}

```