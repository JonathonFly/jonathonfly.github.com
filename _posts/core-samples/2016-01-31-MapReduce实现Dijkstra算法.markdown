---
layout: post
category : Hadoop
tagline: 
tags : [hadoop, mapreduce, dijkstra]
---
{% include JB/setup %}

#Dijkstra算法简介#

首先，介绍一个Dijkstra算法，具体计算方法：
颜色为"b"的点是已经确定最短路径的点，颜色为"g"的点是要通过该点进行最短路径更新的点，即重新计算起点经过颜色为"g"的点到各个颜色为"w"的距离L1，与原来颜色为"w"的最短路径L0比较，如果L1小于L0，则用L1取代L0最为新的最短路径。持续上述步骤，直到图中没有颜色为"g"的点。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt1.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt1.JPG" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt2.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt2.JPG" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt3.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt3.JPG" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt4.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt4.JPG" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt5.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt5.JPG" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt6.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt6.JPG" style="max-width:100%;"></a>

#Dijkstra算法在Mapreduce中的实现#
了解了Dijkstra算法具体是干什么的，那么我们来考虑一下如何在Mapreduce程序中实现它。

<1>输入文件的形式：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt7.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt7.JPG" style="max-width:100%;"></a>

<2>计算邻接表，即计算各个点的从该点所能直接到达的点及其边长，如能从A点经过一条有向边到达B点和C点，则A的邻接点为B和C，变成分别为10和5。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt8.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt8.JPG" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt9.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt9.JPG" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt10.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt10.JPG" style="max-width:100%;"></a>

<3>计算最短路径的mapreduce

1.第一个MapReduce，重新计算各颜色为"w"的点的最短距离，原来的距离与经过颜色为"g"的点的距离比较，取较小的。如果新的距离短，则同时更新前置节点为颜色为"g"的点。

第一轮迭代，第一个MapReduce的Map阶段：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt11.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt11.JPG" style="max-width:100%;"></a>

第一轮迭代，第一个MapReduce的Reduce阶段：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt12.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt12.JPG" style="max-width:100%;"></a>


2.第二个MapReduce，将所有点按距离排序，取颜色为"w"的并且距离最短的点，将其颜色改为"g"，其余的点不变。

第一轮迭代，第二个MapReduce的Map阶段：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt13.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt13.JPG" style="max-width:100%;"></a>


第一轮迭代，第二个MapReduce的Reduce阶段：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt14.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt14.JPG" style="max-width:100%;"></a>


3.迭代计算的结束标志判断：没有颜色为g的点为止。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt15.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt15.JPG" style="max-width:100%;"></a>


4.最短路径各个步骤的计算结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt16.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt16.JPG" style="max-width:100%;"></a>


<4>根据前置节点，计算出最短路径经过的所有节点

第一次Mapreduce，用于初始化计算数据：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt17.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt17.JPG" style="max-width:100%;"></a>


1.后续的迭代的Map阶段：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt18.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt18.JPG" style="max-width:100%;"></a>


2.后续的迭代的Reduce阶段：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt19.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt19.JPG" style="max-width:100%;"></a>


3.各个步骤的计算结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt20.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/ppt20.JPG" style="max-width:100%;"></a>


#在HDFS中计算过程的阶段结果#

输入的文件在HDFS中的形式如下图，其实vertex在计算中没有用到，为了方便画图，我还是将其写在了文件中。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/1.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/1.png" style="max-width:100%;"></a>

初始化：除了起始点A的距离为0，其余距离均为无穷，前置节点均为起始节点A。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/2.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/2.png" style="max-width:100%;"></a>

<1> 第一轮迭代计算：
第一个mapreduce：计算通过当前颜色为"g"的点V1，到各个颜色为"w"的点的距离S，如果该距离S小于原来的距离L，则更新L为S，并记录前置节点为V1。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/3.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/3.png" style="max-width:100%;"></a>


第二个mapreduce：将第一个mapreduce的计算结果作为输入，map阶段将距离作为key，即可在reudeuce阶段自动从小到大排序。取距离最小的并且颜色为“w”的节点，将其颜色改为“g”，其余的点数据不变，再按原始格式输出。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/8.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/8.png" style="max-width:100%;"></a>

<2> 第二轮迭代计算：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/4.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/4.png" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/9.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/9.png" style="max-width:100%;"></a>

<3> 第三轮迭代计算：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/5.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/5.png" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/10.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/10.png" style="max-width:100%;"></a>

<4> 第四轮迭代计算：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/6.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/6.png" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/11.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/11.png" style="max-width:100%;"></a>

<4> 第五轮迭代计算：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/7.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/7.png" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/12.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/12.png" style="max-width:100%;"></a>

整理后最短路径结果如下：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/16.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/16.png" style="max-width:100%;"></a>

根据前置节点结果，找出最短路径的上的所有节点，各个步骤的计算结果如下：

输入为计算最短路径的输出结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/12.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/12.png" style="max-width:100%;"></a>

第一次计算结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/13.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/13.png" style="max-width:100%;"></a>

第二次阶段结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/14.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/14.png" style="max-width:100%;"></a>

第三次计算结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/15.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/15.png" style="max-width:100%;"></a>

目录结果如下：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/17.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/17.png" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/18.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-31/18.png" style="max-width:100%;"></a>

总的代码如下：

每一个节点数据的结构
Node.java

```java
package com.ideal.netcare;

public class Node {
	private String id;
	private String dis;
	private String adjacencyList;
	private String preId;
	private String color="w";
	public Node(){
		this.color="w";
	}
	
	public Node(String nodeString){
		String[] arr=nodeString.split("#|\\s+");
		this.id=arr[0];
		this.dis=arr[1];
		this.adjacencyList=arr[2];
		this.preId=arr[3];
		this.color=arr[4];
	}
	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}
	
	public String getPreId() {
		return preId;
	}

	public void setPreId(String preId) {
		this.preId = preId;
	}

	public String getDis() {
		return dis;
	}

	public void setDis(String dis) {
		this.dis = dis;
	}

	public String getAdjacencyList() {
		return adjacencyList;
	}

	public void setAdjacencyList(String adjacencyList) {
		this.adjacencyList = adjacencyList;
	}

	public String getColor() {
		return color;
	}

	public void setColor(String color) {
		this.color = color;
	}
}

```

计算邻接表
AdjListWithPath.java

```java
package com.ideal.netcare;

import java.io.IOException;
import java.util.Scanner;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class AdjListWithPath {

	private static Scanner sc;

	public static class AdjListWithPathMapper extends Mapper<Object, Text, Text, Text> {

		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			// edge A#B#10
			// vertex A
			if (value.toString() == null || value.toString().equals(""))
				return;
			String[] arr = value.toString().split("\\s+");
			if (arr.length < 2)
				return;
			if (arr[0].equals("edge")) {
				String[] edge = arr[1].split("#");
				// A#B#10
				if (edge.length < 3)
					return;
				// A
				mapKey.set(edge[0]);
				// B-10
				mapValue.set(edge[1] + "-" + edge[2]);
				context.write(mapKey, mapValue);
			}
		}
	}

	public static class AdjListWithPathReducer extends Reducer<Text, Text, Text, Text> {
		private Text result = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			// A B-10
			Configuration conf = context.getConfiguration();
			String startVertex = conf.get("startVertex");
			String adjList = "";
			if (startVertex.equals(key.toString())) {
				// begin vertex
				for (Text val : values) {
					if (val.toString() == null || val.toString().equals(""))
						return;
					adjList += val.toString() + ",";
				}
				// delete the end ','
				if (adjList.length() > 0)
					adjList = adjList.substring(0, adjList.length() - 1);
				String res = "0#" + adjList + "#"+startVertex+"#g";
				result.set(res);
				context.write(key, result);
			} else {
				// other vertexes
				for (Text val : values) {
					if (val.toString() == null || val.toString().equals(""))
						return;
					adjList += val.toString() + ",";
				}
				// delete the end ','
				if (adjList.length() > 0)
					adjList = adjList.substring(0, adjList.length() - 1);
				String res = "<>#" + adjList+ "#"+startVertex + "#w";
				result.set(res);
				context.write(key, result);
			}
		}
	}

	public static void main(String[] args) throws Exception {
	}

}

```

计算最短路径
DijkstraWithPath.java

```java
package com.ideal.netcare;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class DijkstraWithPath {
	public static class DijkstraWithPathMapper extends Mapper<Object, Text, Text, Text> {

		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			// A 0#C-5,B-10#A#g
			if (value.toString() == null || value.toString().equals(""))
				return;
			Node node = new Node(value.toString());
			if (node.getColor().equals("g")) {
				if (node.getAdjacencyList() != null && !node.getAdjacencyList().equals("")) {
					// C-5 B-10
					String[] arr = node.getAdjacencyList().split(",");
					if (arr.length > 0) {
						for (int i = 0; i < arr.length; i++) {
							String[] edge = arr[i].split("-");
							// C 5
							if (edge.length == 2) {
								String id = edge[0];
								String length = "";
								if (node.getDis().equals("<>")) {
									length = node.getDis();
								} else {
									//currentDis+adjEdgeLength
									length = (Integer.parseInt(node.getDis()) + Integer.parseInt(edge[1])) + "";
								}
								mapKey.set(id);
								// B 10##A#w C 5##A#w
								mapValue.set(length + "##" + node.getId() + "#w");
								context.write(mapKey, mapValue);
							}
						}
					}
					// make the grey node black
					mapKey.set(node.getId());
					mapValue.set(node.getDis() + "#" + node.getAdjacencyList() + "#" + node.getPreId() + "#b");
					context.write(mapKey, mapValue);
				}
			} else {
				mapKey.set(node.getId());
				mapValue.set(node.getDis() + "#" + node.getAdjacencyList() + "#" + node.getPreId() + "#" + node.getColor());
				context.write(mapKey, mapValue);
			}
		}
	}

	public static class DijkstraWithPathReducer extends Reducer<Text, Text, Text, Text> {
		private Text result = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			// find the minDistance of the same key

			// results
			int mindis=0;
			String adj=null;
			String pre=null;
			String colr=null;
			boolean init = true;

			for (Text val : values) {
				if (val.toString() == null || val.toString().equals(""))
					return;
				String[] arr = val.toString().split("#");
				if (arr.length < 4)
					return;
				String dis = arr[0];
				String adjList = arr[1];
				String preId = arr[2];
				String color = arr[3];
				// init the results
				if (init) {
					if (!dis.equals("<>"))
						mindis = Integer.parseInt(dis);
					else
						mindis = Integer.MAX_VALUE;
					adj = adjList;
					pre = preId;
					colr = color;
					//has inited
					init = false;
				} else {
					// if color is black ,break the loop
					if (color.equals("b")) {
						if (!dis.equals("<>")) {
							mindis = Integer.parseInt(dis);
						}
						// adjList
						adj = adjList;
						// Pre node id
						pre = preId;
						// color
						colr = color;
						break;
					} else {
						if (!dis.equals("<>")) {
							if (Integer.parseInt(dis) < mindis) {
								// the min dis
								mindis = Integer.parseInt(dis);
								// the pre node id
								pre = preId;
							}
						}
						// the adjlist
						if (adjList != null && !adjList.equals("")) {
							adj = adjList;
						}
					}
				}
			}
			if (mindis != Integer.MAX_VALUE) {
				result.set(mindis + "#" + adj + "#" + pre + "#" + colr);
			} else {
				result.set("<>#" + adj + "#" + pre + "#" + colr);
			}
			context.write(key, result);
		}
	}

	public static class MinDisSortWithPathMapper extends Mapper<Object, Text, IntWritable, Text> {

		private IntWritable mapKey = new IntWritable();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			if (value.toString() == null || value.toString().equals(""))
				return;
			Node node = new Node(value.toString());
			int dis = Integer.MAX_VALUE;
			if (!node.getDis().equals("<>")) {
				dis = Integer.parseInt(node.getDis());
			}
			mapKey.set(dis);
			mapValue.set(node.getId() + "#" + node.getAdjacencyList() + "#"+node.getPreId()+"#" + node.getColor());
			//0 A#B-10,C-5#A#b
			context.write(mapKey, mapValue);
		}
	}

	public static class MinDisSortWithPathReducer extends Reducer<IntWritable, Text, Text, Text> {
		private Text key1 = new Text();
		private Text result = new Text();
		// find the mindis of white color node
		private boolean flag = true;

		public void reduce(IntWritable key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			for (Text val : values) {
				//A		B-10,C-5		A		b
				String[] arr = val.toString().split("#");
				if (arr.length != 4)
					return;
				// has sort distance esc, find the first white node
				if (flag && arr[3].equals("w")) {
					key1.set(arr[0]);
					// make its color grey
					if (key.get() == Integer.MAX_VALUE)
						result.set("<>#" + arr[1] + "#"+arr[2]+"#g");
					else
						result.set(key.get() + "#" + arr[1]+ "#"+arr[2] + "#g");
					flag = false;
					context.write(key1, result);
				} else {
					// other nodes don't change
					key1.set(arr[0]);
					if (key.get() == Integer.MAX_VALUE)
						result.set("<>#" + arr[1] + "#"+arr[2]+"#" + arr[3]);
					else
						result.set(key.get() + "#" + arr[1] + "#"+arr[2]+"#" + arr[3]);
					context.write(key1, result);
				}
			}
		}
	}

	public static void main(String[] args) throws Exception {
		
	}
}

```

计算最短路径上的所有顶点
PathResult.java

```java
package com.ideal.netcare;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class PathResult {

	public static class PathPreMapper extends Mapper<Object, Text, Text, Text> {
		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			// the bigin vertex
			Configuration conf = context.getConfiguration();
			String startVertex = conf.get("startVertex");

			if (value.toString() == null || value.toString().equals(""))
				return;
			Node node = new Node(value.toString());

			if (node.getPreId().equals(startVertex)) {
				mapKey.set("b");
				mapValue.set(node.getPreId() + "-" + node.getId());
			} else {
				mapKey.set("w");
				mapValue.set(node.getPreId() + "-" + node.getId());
			}
			context.write(mapKey, mapValue);
		}
	}

	public static class PathPreReducer extends Reducer<Text, Text, Text, Text> {
		private Text result = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			for (Text val : values) {
				if (val.toString() == null || val.toString().equals(""))
					return;
				result.set(val.toString());
				context.write(key, result);
			}
		}
	}

	public static class PathMapper extends Mapper<Object, Text, Text, Text> {

		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			if (value.toString() == null || value.toString().equals(""))
				return;

			// b A-C
			String[] arr = value.toString().split("\\s+");
			if (arr[0].equals("b")) {
				// has find the begin vertex ,get the string after the last "-" to be key
				String last = arr[1].substring(arr[1].lastIndexOf("-") + 1);
				// C b#A-C
				mapKey.set(last);
				mapValue.set("b#" + arr[1]);
			} else {
				// w C-E
				// haven't find the begin vertex ,get the string before the first "-" to be key
				String first = arr[1].substring(0, arr[1].indexOf("-"));
				// C w#C-E
				mapKey.set(first);
				mapValue.set("w#" + arr[1]);
			}
			context.write(mapKey, mapValue);
		}
	}

	public static class PathReducer extends Reducer<Text, Text, Text, Text> {
		private Text newKey = new Text();
		private Text newValue = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			List<String> preList = new ArrayList<String>();
			List<String> afterList = new ArrayList<String>();
			for (Text val : values) {
				if (val.toString() == null || val.toString().equals(""))
					return;
				// C b#A-C
				// C w#C-E
				// C w#C-B
				String[] arr = val.toString().split("#");
				if (arr[0].equals("b")) {
					newKey.set(arr[0]);
					newValue.set(arr[1]);
					preList.add(arr[1]);
					context.write(newKey, newValue);
				} else {
					afterList.add(arr[1]);
				}
			}
			if (preList.size() > 0) {
				// has the black color
				//preList has the begin vertex, afterList doesn't
				for (int i = 0; i < preList.size(); i++) {
					for (int j = 0; afterList.size() > 0 && j < afterList.size(); j++) {
						newKey.set("b");
						// "A-C" and "C-E" link and delete one "C"
						newValue.set(preList.get(i) + afterList.get(j).substring(afterList.get(j).indexOf("-")));
						context.write(newKey, newValue);
					}
				}
			} else {
				// only has white color nodes
				for (int j = 0; afterList.size() > 0 && j < afterList.size(); j++) {
					newKey.set("w");
					newValue.set(afterList.get(j));
					context.write(newKey, newValue);
				}
			}
			preList.clear();
			afterList.clear();
		}
	}

	public static void main(String[] args) throws Exception {
		
	}

}

```

整理最短路径结果
Result.java 

```java
package com.ideal.netcare;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class Result {

	public static class ResultValueMapper extends Mapper<Object, Text, Text, Text> {

		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			if (value.toString() == null || value.toString().equals(""))
				return;
			String[] arr = value.toString().split("\\s+");
			if (arr.length < 2)
				return;
			mapKey.set(arr[0]);
			mapValue.set(arr[1]);
			context.write(mapKey, mapValue);
		}
	}

	public static class ResultValueReducer extends Reducer<Text, Text, Text, Text> {
		private Text result = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			for (Text val : values) {
				if (val.toString() == null || val.toString().equals(""))
					return;
				String res=val.toString().substring(0, val.toString().indexOf("#"));
				result.set(res);
				context.write(key, result);
			}
		}
	}
	
	public static class ResultPathMapper extends Mapper<Object, Text, Text, Text> {

		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			if (value.toString() == null || value.toString().equals(""))
				return;
			String[] arr = value.toString().split("\\s+");
			if (arr.length < 2)
				return;
			String keyarr=arr[1].substring(arr[1].lastIndexOf("-")+1);
			mapKey.set(keyarr);
			mapValue.set(arr[1]);
			context.write(mapKey, mapValue);
		}
	}

	public static class ResultPathReducer extends Reducer<Text, Text, Text, Text> {
		private Text result = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			for (Text val : values) {
				if (val.toString() == null || val.toString().equals(""))
					return;
				result.set(val.toString());
				context.write(key, result);
			}
		}
	}
	
	public static class ResultMapper extends Mapper<Object, Text, Text, Text> {

		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			if (value.toString() == null || value.toString().equals(""))
				return;
			String[] arr = value.toString().split("\\s+");
			if (arr.length < 2)
				return;
			mapKey.set(arr[0]);
			mapValue.set(arr[1]);
			context.write(mapKey, mapValue);
		}
	}

	public static class ResultReducer extends Reducer<Text, Text, Text, Text> {
		private Text result = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			String res="";
			String path="";
			String value="";
			for (Text val : values) {
				if (val.toString() == null || val.toString().equals(""))
					return;
				if(val.toString().contains("-"))
					path=val.toString();
				else
					value=val.toString();
			}
			res=path+", "+value;
			result.set(res);
			context.write(key, result);
		}
	}

	public static void main(String[] args) throws Exception {
		
	}

}

```

总的调用main函数：
AdjListAndDijkstraWithPath.java

```java
package com.ideal.netcare;

import java.io.BufferedReader;
import java.io.InputStreamReader;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.ideal.netcare.AdjListWithPath.AdjListWithPathMapper;
import com.ideal.netcare.AdjListWithPath.AdjListWithPathReducer;
import com.ideal.netcare.DijkstraWithPath.DijkstraWithPathMapper;
import com.ideal.netcare.DijkstraWithPath.DijkstraWithPathReducer;
import com.ideal.netcare.DijkstraWithPath.MinDisSortWithPathMapper;
import com.ideal.netcare.DijkstraWithPath.MinDisSortWithPathReducer;
import com.ideal.netcare.PathResult.PathMapper;
import com.ideal.netcare.PathResult.PathPreMapper;
import com.ideal.netcare.PathResult.PathPreReducer;
import com.ideal.netcare.PathResult.PathReducer;
import com.ideal.netcare.Result.ResultMapper;
import com.ideal.netcare.Result.ResultPathMapper;
import com.ideal.netcare.Result.ResultPathReducer;
import com.ideal.netcare.Result.ResultReducer;
import com.ideal.netcare.Result.ResultValueMapper;
import com.ideal.netcare.Result.ResultValueReducer;

public class AdjListAndDijkstraWithPath {

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		conf.set("mapred.jar", "Dijkstra.jar");
		conf.set("startVertex", "A");

		// find the adjlist
		String[] ars = new String[] { "/syf/adjlist-withpath/input", "/syf/adjlist-withpath/output" };
		String[] otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("usage: adjList-withpath <in> [<in>...] <out> ");
			System.exit(2);
		}
		Job job0 = Job.getInstance(conf, "adjList-withpath");
		job0.setJarByClass(AdjListWithPath.class);
		job0.setMapperClass(AdjListWithPathMapper.class);
		job0.setReducerClass(AdjListWithPathReducer.class);
		job0.setOutputKeyClass(Text.class);
		job0.setOutputValueClass(Text.class);

		for (int i = 0; i < otherArgs.length - 1; ++i) {
			FileInputFormat.addInputPath(job0, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job0, new Path(otherArgs[otherArgs.length - 1]));
		job0.waitForCompletion(true);

		// find the shortest distance
		boolean isdone = false;
		int num = 0;
		while (!isdone) {
			if (num == 0)
				ars = new String[] { "/syf/adjlist-withpath/output", "/syf/dijkstra-withpath/output/output_dijkstra" + num };
			else
				ars = new String[] { "/syf/dijkstra-withpath/output/output_mindis" + (num - 1), "/syf/dijkstra-withpath/output/output_dijkstra" + num };
			otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
			if (otherArgs.length != 2) {
				System.err.println("usage: dijkstra" + num + " <in> [<in>...] <out> ");
				System.exit(2);
			}
			Job job1 = Job.getInstance(conf, "dijkstra-withpath" + num);
			job1.setJarByClass(Dijkstra.class);
			job1.setMapperClass(DijkstraWithPathMapper.class);
			job1.setReducerClass(DijkstraWithPathReducer.class);
			job1.setOutputKeyClass(Text.class);
			job1.setOutputValueClass(Text.class);

			for (int i = 0; i < otherArgs.length - 1; ++i) {
				FileInputFormat.addInputPath(job1, new Path(otherArgs[i]));
			}
			FileOutputFormat.setOutputPath(job1, new Path(otherArgs[otherArgs.length - 1]));
			job1.waitForCompletion(true);

			//
			ars = new String[] { "/syf/dijkstra-withpath/output/output_dijkstra" + num, "/syf/dijkstra-withpath/output/output_mindis" + num };
			otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
			if (otherArgs.length != 2) {
				System.err.println("usage: mindis-withpath <in> [<in>...] <out> ");
				System.exit(2);
			}
			Job job2 = Job.getInstance(conf, "mindis-withpath");
			job2.setJarByClass(Dijkstra.class);
			job2.setMapperClass(MinDisSortWithPathMapper.class);
			job2.setReducerClass(MinDisSortWithPathReducer.class);

			job2.setMapOutputKeyClass(IntWritable.class);
			job2.setMapOutputValueClass(Text.class);
			job2.setOutputKeyClass(Text.class);
			job2.setOutputValueClass(Text.class);
			for (int i = 0; i < otherArgs.length - 1; ++i) {
				FileInputFormat.addInputPath(job2, new Path(otherArgs[i]));
			}
			FileOutputFormat.setOutputPath(job2, new Path(otherArgs[otherArgs.length - 1]));
			job2.waitForCompletion(true);

			// check is end?
			isdone = true;
			Path file = new Path("/syf/dijkstra-withpath/output/output_mindis" + num + "/part-r-00000");
			FileSystem fs = FileSystem.get(conf);
			BufferedReader br = new BufferedReader(new InputStreamReader(fs.open(file)));

			String line = br.readLine();
			while (line != null) {
				String color = line.substring(line.toString().lastIndexOf("#") + 1);
				System.out.println(color);
				if (color.equals("g")) {
					isdone = false;
					break;
				}
				line = br.readLine();
			}

			num++;
		}

		// style shortest distance result
		ars = new String[] { "/syf/dijkstra-withpath/output/output_dijkstra" + (num - 1), "/syf/dijkstra-withpath/output/result_value" };
		otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("usage: result_value-withpath <in> [<in>...] <out> ");
			System.exit(2);
		}
		Job job3 = Job.getInstance(conf, "result_value-withpath");
		job3.setJarByClass(Result.class);
		job3.setMapperClass(ResultValueMapper.class);
		job3.setReducerClass(ResultValueReducer.class);
		job3.setOutputKeyClass(Text.class);
		job3.setOutputValueClass(Text.class);

		for (int i = 0; i < otherArgs.length - 1; ++i) {
			FileInputFormat.addInputPath(job3, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job3, new Path(otherArgs[otherArgs.length - 1]));
		job3.waitForCompletion(true);

		// first path job
		ars = new String[] { "/syf/dijkstra-withpath/output/output_dijkstra" + (num - 1), "/syf/dijkstra-withpath/output/output_path0" };
		otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("usage: path0 <in> [<in>...] <out> ");
			System.exit(2);
		}
		Job job4 = Job.getInstance(conf, "path0");
		job4.setJarByClass(PathResult.class);
		job4.setMapperClass(PathPreMapper.class);
		job4.setReducerClass(PathPreReducer.class);
		job4.setOutputKeyClass(Text.class);
		job4.setOutputValueClass(Text.class);

		for (int i = 0; i < otherArgs.length - 1; ++i) {
			FileInputFormat.addInputPath(job4, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job4, new Path(otherArgs[otherArgs.length - 1]));
		job4.waitForCompletion(true);

		// other path jobs
		boolean isdone1 = false;
		int num1 = 1;
		while (!isdone1) {
			ars = new String[] { "/syf/dijkstra-withpath/output/output_path" + (num1 - 1), "/syf/dijkstra-withpath/output/output_path" + num1 };
			otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
			if (otherArgs.length != 2) {
				System.err.println("usage: path" + num1 + " <in> [<in>...] <out> ");
				System.exit(2);
			}
			Job job5 = Job.getInstance(conf, "path" + num1);
			job5.setJarByClass(PathResult.class);
			job5.setMapperClass(PathMapper.class);
			job5.setReducerClass(PathReducer.class);
			job5.setOutputKeyClass(Text.class);
			job5.setOutputValueClass(Text.class);

			for (int i = 0; i < otherArgs.length - 1; ++i) {
				FileInputFormat.addInputPath(job5, new Path(otherArgs[i]));
			}
			FileOutputFormat.setOutputPath(job5, new Path(otherArgs[otherArgs.length - 1]));
			job5.waitForCompletion(true);

			// check is end?
			isdone1 = true;
			Path file = new Path("/syf/dijkstra-withpath/output/output_path" + num1 + "/part-r-00000");
			FileSystem fs = FileSystem.get(conf);
			BufferedReader br = new BufferedReader(new InputStreamReader(fs.open(file)));

			String line = br.readLine();
			while (line != null) {
				String color = line.split("\\s+")[0];
				System.out.println(color);
				if (color.equals("w")) {
					isdone1 = false;
					break;
				}
				line = br.readLine();
			}

			num1++;
		}

		// style the total result
		ars = new String[] { "/syf/dijkstra-withpath/output/output_path" + (num1 - 1), "/syf/dijkstra-withpath/output/result_path" };
		otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("usage: result_path-withpath <in> [<in>...] <out> ");
			System.exit(2);
		}
		Job job6 = Job.getInstance(conf, "result_path-withpath");
		job6.setJarByClass(Result.class);
		job6.setMapperClass(ResultPathMapper.class);
		job6.setReducerClass(ResultPathReducer.class);
		job6.setOutputKeyClass(Text.class);
		job6.setOutputValueClass(Text.class);

		for (int i = 0; i < otherArgs.length - 1; ++i) {
			FileInputFormat.addInputPath(job6, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job6, new Path(otherArgs[otherArgs.length - 1]));
		job6.waitForCompletion(true);

		//
		ars = new String[] { "/syf/dijkstra-withpath/output/result_value", "/syf/dijkstra-withpath/output/result_path", "/syf/dijkstra-withpath/output/result" };
		otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
		if (otherArgs.length != 3) {
			System.err.println("usage: result-withpath <in> [<in>...] <out> ");
			System.exit(2);
		}
		Job job7 = Job.getInstance(conf, "result-withpath");
		job7.setJarByClass(Result.class);
		job7.setMapperClass(ResultMapper.class);
		job7.setReducerClass(ResultReducer.class);
		job7.setOutputKeyClass(Text.class);
		job7.setOutputValueClass(Text.class);

		for (int i = 0; i < otherArgs.length - 1; ++i) {
			FileInputFormat.addInputPath(job7, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job7, new Path(otherArgs[otherArgs.length - 1]));
		job7.waitForCompletion(true);

	}

}

```

最后，迭代计算并不适合用MapReduce框架来计算，因为读写HDFS过于频繁，效率低下。还是适合用专门的图计算框架，如spark的Graphx或者GraphLab等。

