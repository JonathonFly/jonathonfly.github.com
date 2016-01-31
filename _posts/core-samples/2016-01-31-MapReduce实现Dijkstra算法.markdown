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

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal.netcare</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">Node</span> {
    <span class="pl-k">private</span> <span class="pl-smi">String</span> id;
    <span class="pl-k">private</span> <span class="pl-smi">String</span> dis;
    <span class="pl-k">private</span> <span class="pl-smi">String</span> adjacencyList;
    <span class="pl-k">private</span> <span class="pl-smi">String</span> preId;
    <span class="pl-k">private</span> <span class="pl-smi">String</span> color<span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">"</span>w<span class="pl-pds">"</span></span>;
    <span class="pl-k">public</span> <span class="pl-en">Node</span>(){
        <span class="pl-v">this</span><span class="pl-k">.</span>color<span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">"</span>w<span class="pl-pds">"</span></span>;
    }

    <span class="pl-k">public</span> <span class="pl-en">Node</span>(<span class="pl-smi">String</span> <span class="pl-v">nodeString</span>){
        <span class="pl-k">String</span>[] arr<span class="pl-k">=</span>nodeString<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>#|<span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
        <span class="pl-v">this</span><span class="pl-k">.</span>id<span class="pl-k">=</span>arr[<span class="pl-c1">0</span>];
        <span class="pl-v">this</span><span class="pl-k">.</span>dis<span class="pl-k">=</span>arr[<span class="pl-c1">1</span>];
        <span class="pl-v">this</span><span class="pl-k">.</span>adjacencyList<span class="pl-k">=</span>arr[<span class="pl-c1">2</span>];
        <span class="pl-v">this</span><span class="pl-k">.</span>preId<span class="pl-k">=</span>arr[<span class="pl-c1">3</span>];
        <span class="pl-v">this</span><span class="pl-k">.</span>color<span class="pl-k">=</span>arr[<span class="pl-c1">4</span>];
    }
    <span class="pl-k">public</span> <span class="pl-smi">String</span> <span class="pl-en">getId</span>() {
        <span class="pl-k">return</span> id;
    }

    <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">setId</span>(<span class="pl-smi">String</span> <span class="pl-v">id</span>) {
        <span class="pl-v">this</span><span class="pl-k">.</span>id <span class="pl-k">=</span> id;
    }

    <span class="pl-k">public</span> <span class="pl-smi">String</span> <span class="pl-en">getPreId</span>() {
        <span class="pl-k">return</span> preId;
    }

    <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">setPreId</span>(<span class="pl-smi">String</span> <span class="pl-v">preId</span>) {
        <span class="pl-v">this</span><span class="pl-k">.</span>preId <span class="pl-k">=</span> preId;
    }

    <span class="pl-k">public</span> <span class="pl-smi">String</span> <span class="pl-en">getDis</span>() {
        <span class="pl-k">return</span> dis;
    }

    <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">setDis</span>(<span class="pl-smi">String</span> <span class="pl-v">dis</span>) {
        <span class="pl-v">this</span><span class="pl-k">.</span>dis <span class="pl-k">=</span> dis;
    }

    <span class="pl-k">public</span> <span class="pl-smi">String</span> <span class="pl-en">getAdjacencyList</span>() {
        <span class="pl-k">return</span> adjacencyList;
    }

    <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">setAdjacencyList</span>(<span class="pl-smi">String</span> <span class="pl-v">adjacencyList</span>) {
        <span class="pl-v">this</span><span class="pl-k">.</span>adjacencyList <span class="pl-k">=</span> adjacencyList;
    }

    <span class="pl-k">public</span> <span class="pl-smi">String</span> <span class="pl-en">getColor</span>() {
        <span class="pl-k">return</span> color;
    }

    <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">setColor</span>(<span class="pl-smi">String</span> <span class="pl-v">color</span>) {
        <span class="pl-v">this</span><span class="pl-k">.</span>color <span class="pl-k">=</span> color;
    }
}
</pre></div>

计算邻接表
AdjListWithPath.java

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal.netcare</span>;

<span class="pl-k">import</span> <span class="pl-smi">java.io.IOException</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.util.Scanner</span>;

<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.conf.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.Path</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.Text</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Job</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Mapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Reducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileInputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.output.FileOutputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.util.GenericOptionsParser</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">AdjListWithPath</span> {

    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-smi">Scanner</span> sc;

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">AdjListWithPathMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-c">// edge A#B#10</span>
            <span class="pl-c">// vertex A</span>
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (arr<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">2</span>)
                <span class="pl-k">return</span>;
            <span class="pl-k">if</span> (arr[<span class="pl-c1">0</span>]<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>edge<span class="pl-pds">"</span></span>)) {
                <span class="pl-k">String</span>[] edge <span class="pl-k">=</span> arr[<span class="pl-c1">1</span>]<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span>);
                <span class="pl-c">// A#B#10</span>
                <span class="pl-k">if</span> (edge<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">3</span>)
                    <span class="pl-k">return</span>;
                <span class="pl-c">// A</span>
                mapKey<span class="pl-k">.</span>set(edge[<span class="pl-c1">0</span>]);
                <span class="pl-c">// B-10</span>
                mapValue<span class="pl-k">.</span>set(edge[<span class="pl-c1">1</span>] <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span> <span class="pl-k">+</span> edge[<span class="pl-c1">2</span>]);
                context<span class="pl-k">.</span>write(mapKey, mapValue);
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">AdjListWithPathReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-c">// A B-10</span>
            <span class="pl-smi">Configuration</span> conf <span class="pl-k">=</span> context<span class="pl-k">.</span>getConfiguration();
            <span class="pl-smi">String</span> startVertex <span class="pl-k">=</span> conf<span class="pl-k">.</span>get(<span class="pl-s"><span class="pl-pds">"</span>startVertex<span class="pl-pds">"</span></span>);
            <span class="pl-smi">String</span> adjList <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>;
            <span class="pl-k">if</span> (startVertex<span class="pl-k">.</span>equals(key<span class="pl-k">.</span>toString())) {
                <span class="pl-c">// begin vertex</span>
                <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                    <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                        <span class="pl-k">return</span>;
                    adjList <span class="pl-k">+=</span> val<span class="pl-k">.</span>toString() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>,<span class="pl-pds">"</span></span>;
                }
                <span class="pl-c">// delete the end ','</span>
                <span class="pl-k">if</span> (adjList<span class="pl-k">.</span>length() <span class="pl-k">&gt;</span> <span class="pl-c1">0</span>)
                    adjList <span class="pl-k">=</span> adjList<span class="pl-k">.</span>substring(<span class="pl-c1">0</span>, adjList<span class="pl-k">.</span>length() <span class="pl-k">-</span> <span class="pl-c1">1</span>);
                <span class="pl-smi">String</span> res <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>0#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> adjList <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span><span class="pl-k">+</span>startVertex<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span>#g<span class="pl-pds">"</span></span>;
                result<span class="pl-k">.</span>set(res);
                context<span class="pl-k">.</span>write(key, result);
            } <span class="pl-k">else</span> {
                <span class="pl-c">// other vertexes</span>
                <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                    <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                        <span class="pl-k">return</span>;
                    adjList <span class="pl-k">+=</span> val<span class="pl-k">.</span>toString() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>,<span class="pl-pds">"</span></span>;
                }
                <span class="pl-c">// delete the end ','</span>
                <span class="pl-k">if</span> (adjList<span class="pl-k">.</span>length() <span class="pl-k">&gt;</span> <span class="pl-c1">0</span>)
                    adjList <span class="pl-k">=</span> adjList<span class="pl-k">.</span>substring(<span class="pl-c1">0</span>, adjList<span class="pl-k">.</span>length() <span class="pl-k">-</span> <span class="pl-c1">1</span>);
                <span class="pl-smi">String</span> res <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> adjList<span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span><span class="pl-k">+</span>startVertex <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#w<span class="pl-pds">"</span></span>;
                result<span class="pl-k">.</span>set(res);
                context<span class="pl-k">.</span>write(key, result);
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">void</span> <span class="pl-en">main</span>(<span class="pl-k">String</span>[] <span class="pl-v">args</span>) <span class="pl-k">throws</span> <span class="pl-smi">Exception</span> {
    }

}
</pre></div>

计算最短路径
DijkstraWithPath.java

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal.netcare</span>;

<span class="pl-k">import</span> <span class="pl-smi">java.io.BufferedReader</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.io.IOException</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.io.InputStreamReader</span>;

<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.conf.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.FileSystem</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.Path</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.IntWritable</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.Text</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Job</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Mapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Reducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileInputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.output.FileOutputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.util.GenericOptionsParser</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">DijkstraWithPath</span> {
    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">DijkstraWithPathMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-c">// A 0#C-5,B-10#A#g</span>
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-smi">Node</span> node <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Node</span>(value<span class="pl-k">.</span>toString());
            <span class="pl-k">if</span> (node<span class="pl-k">.</span>getColor()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>g<span class="pl-pds">"</span></span>)) {
                <span class="pl-k">if</span> (node<span class="pl-k">.</span>getAdjacencyList() <span class="pl-k">!=</span> <span class="pl-c1">null</span> <span class="pl-k">&amp;&amp;</span> <span class="pl-k">!</span>node<span class="pl-k">.</span>getAdjacencyList()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>)) {
                    <span class="pl-c">// C-5 B-10</span>
                    <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> node<span class="pl-k">.</span>getAdjacencyList()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>,<span class="pl-pds">"</span></span>);
                    <span class="pl-k">if</span> (arr<span class="pl-k">.</span>length <span class="pl-k">&gt;</span> <span class="pl-c1">0</span>) {
                        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> arr<span class="pl-k">.</span>length; i<span class="pl-k">++</span>) {
                            <span class="pl-k">String</span>[] edge <span class="pl-k">=</span> arr[i]<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span>);
                            <span class="pl-c">// C 5</span>
                            <span class="pl-k">if</span> (edge<span class="pl-k">.</span>length <span class="pl-k">==</span> <span class="pl-c1">2</span>) {
                                <span class="pl-smi">String</span> id <span class="pl-k">=</span> edge[<span class="pl-c1">0</span>];
                                <span class="pl-smi">String</span> length <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>;
                                <span class="pl-k">if</span> (node<span class="pl-k">.</span>getDis()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;<span class="pl-pds">"</span></span>)) {
                                    length <span class="pl-k">=</span> node<span class="pl-k">.</span>getDis();
                                } <span class="pl-k">else</span> {
                                    <span class="pl-c">//currentDis+adjEdgeLength</span>
                                    length <span class="pl-k">=</span> (<span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(node<span class="pl-k">.</span>getDis()) <span class="pl-k">+</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(edge[<span class="pl-c1">1</span>])) <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>;
                                }
                                mapKey<span class="pl-k">.</span>set(id);
                                <span class="pl-c">// B 10##A#w C 5##A#w</span>
                                mapValue<span class="pl-k">.</span>set(length <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>##<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getId() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#w<span class="pl-pds">"</span></span>);
                                context<span class="pl-k">.</span>write(mapKey, mapValue);
                            }
                        }
                    }
                    <span class="pl-c">// make the grey node black</span>
                    mapKey<span class="pl-k">.</span>set(node<span class="pl-k">.</span>getId());
                    mapValue<span class="pl-k">.</span>set(node<span class="pl-k">.</span>getDis() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getAdjacencyList() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getPreId() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#b<span class="pl-pds">"</span></span>);
                    context<span class="pl-k">.</span>write(mapKey, mapValue);
                }
            } <span class="pl-k">else</span> {
                mapKey<span class="pl-k">.</span>set(node<span class="pl-k">.</span>getId());
                mapValue<span class="pl-k">.</span>set(node<span class="pl-k">.</span>getDis() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getAdjacencyList() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getPreId() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getColor());
                context<span class="pl-k">.</span>write(mapKey, mapValue);
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">DijkstraWithPathReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-c">// find the minDistance of the same key</span>

            <span class="pl-c">// results</span>
            <span class="pl-k">int</span> mindis<span class="pl-k">=</span><span class="pl-c1">0</span>;
            <span class="pl-smi">String</span> adj<span class="pl-k">=</span><span class="pl-c1">null</span>;
            <span class="pl-smi">String</span> pre<span class="pl-k">=</span><span class="pl-c1">null</span>;
            <span class="pl-smi">String</span> colr<span class="pl-k">=</span><span class="pl-c1">null</span>;
            <span class="pl-k">boolean</span> init <span class="pl-k">=</span> <span class="pl-c1">true</span>;

            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                    <span class="pl-k">return</span>;
                <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span>);
                <span class="pl-k">if</span> (arr<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">4</span>)
                    <span class="pl-k">return</span>;
                <span class="pl-smi">String</span> dis <span class="pl-k">=</span> arr[<span class="pl-c1">0</span>];
                <span class="pl-smi">String</span> adjList <span class="pl-k">=</span> arr[<span class="pl-c1">1</span>];
                <span class="pl-smi">String</span> preId <span class="pl-k">=</span> arr[<span class="pl-c1">2</span>];
                <span class="pl-smi">String</span> color <span class="pl-k">=</span> arr[<span class="pl-c1">3</span>];
                <span class="pl-c">// init the results</span>
                <span class="pl-k">if</span> (init) {
                    <span class="pl-k">if</span> (<span class="pl-k">!</span>dis<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;<span class="pl-pds">"</span></span>))
                        mindis <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(dis);
                    <span class="pl-k">else</span>
                        mindis <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-c1"><span class="pl-k">.</span>MAX_VALUE</span>;
                    adj <span class="pl-k">=</span> adjList;
                    pre <span class="pl-k">=</span> preId;
                    colr <span class="pl-k">=</span> color;
                    <span class="pl-c">//has inited</span>
                    init <span class="pl-k">=</span> <span class="pl-c1">false</span>;
                } <span class="pl-k">else</span> {
                    <span class="pl-c">// if color is black ,break the loop</span>
                    <span class="pl-k">if</span> (color<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>b<span class="pl-pds">"</span></span>)) {
                        <span class="pl-k">if</span> (<span class="pl-k">!</span>dis<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;<span class="pl-pds">"</span></span>)) {
                            mindis <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(dis);
                        }
                        <span class="pl-c">// adjList</span>
                        adj <span class="pl-k">=</span> adjList;
                        <span class="pl-c">// Pre node id</span>
                        pre <span class="pl-k">=</span> preId;
                        <span class="pl-c">// color</span>
                        colr <span class="pl-k">=</span> color;
                        <span class="pl-k">break</span>;
                    } <span class="pl-k">else</span> {
                        <span class="pl-k">if</span> (<span class="pl-k">!</span>dis<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;<span class="pl-pds">"</span></span>)) {
                            <span class="pl-k">if</span> (<span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(dis) <span class="pl-k">&lt;</span> mindis) {
                                <span class="pl-c">// the min dis</span>
                                mindis <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(dis);
                                <span class="pl-c">// the pre node id</span>
                                pre <span class="pl-k">=</span> preId;
                            }
                        }
                        <span class="pl-c">// the adjlist</span>
                        <span class="pl-k">if</span> (adjList <span class="pl-k">!=</span> <span class="pl-c1">null</span> <span class="pl-k">&amp;&amp;</span> <span class="pl-k">!</span>adjList<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>)) {
                            adj <span class="pl-k">=</span> adjList;
                        }
                    }
                }
            }
            <span class="pl-k">if</span> (mindis <span class="pl-k">!=</span> <span class="pl-smi">Integer</span><span class="pl-c1"><span class="pl-k">.</span>MAX_VALUE</span>) {
                result<span class="pl-k">.</span>set(mindis <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> adj <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> pre <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> colr);
            } <span class="pl-k">else</span> {
                result<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> adj <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> pre <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> colr);
            }
            context<span class="pl-k">.</span>write(key, result);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">MinDisSortWithPathMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">IntWritable</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">IntWritable</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">IntWritable</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-smi">Node</span> node <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Node</span>(value<span class="pl-k">.</span>toString());
            <span class="pl-k">int</span> dis <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-c1"><span class="pl-k">.</span>MAX_VALUE</span>;
            <span class="pl-k">if</span> (<span class="pl-k">!</span>node<span class="pl-k">.</span>getDis()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;<span class="pl-pds">"</span></span>)) {
                dis <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(node<span class="pl-k">.</span>getDis());
            }
            mapKey<span class="pl-k">.</span>set(dis);
            mapValue<span class="pl-k">.</span>set(node<span class="pl-k">.</span>getId() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getAdjacencyList() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span><span class="pl-k">+</span>node<span class="pl-k">.</span>getPreId()<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getColor());
            <span class="pl-c">//0 A#B-10,C-5#A#b</span>
            context<span class="pl-k">.</span>write(mapKey, mapValue);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">MinDisSortWithPathReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">IntWritable</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> key1 <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-c">// find the mindis of white color node</span>
        <span class="pl-k">private</span> <span class="pl-k">boolean</span> flag <span class="pl-k">=</span> <span class="pl-c1">true</span>;

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">IntWritable</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                <span class="pl-c">//A     B-10,C-5        A       b</span>
                <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span>);
                <span class="pl-k">if</span> (arr<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">4</span>)
                    <span class="pl-k">return</span>;
                <span class="pl-c">// has sort distance esc, find the first white node</span>
                <span class="pl-k">if</span> (flag <span class="pl-k">&amp;&amp;</span> arr[<span class="pl-c1">3</span>]<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>w<span class="pl-pds">"</span></span>)) {
                    key1<span class="pl-k">.</span>set(arr[<span class="pl-c1">0</span>]);
                    <span class="pl-c">// make its color grey</span>
                    <span class="pl-k">if</span> (key<span class="pl-k">.</span>get() <span class="pl-k">==</span> <span class="pl-smi">Integer</span><span class="pl-c1"><span class="pl-k">.</span>MAX_VALUE</span>)
                        result<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> arr[<span class="pl-c1">1</span>] <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span><span class="pl-k">+</span>arr[<span class="pl-c1">2</span>]<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span>#g<span class="pl-pds">"</span></span>);
                    <span class="pl-k">else</span>
                        result<span class="pl-k">.</span>set(key<span class="pl-k">.</span>get() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> arr[<span class="pl-c1">1</span>]<span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span><span class="pl-k">+</span>arr[<span class="pl-c1">2</span>] <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#g<span class="pl-pds">"</span></span>);
                    flag <span class="pl-k">=</span> <span class="pl-c1">false</span>;
                    context<span class="pl-k">.</span>write(key1, result);
                } <span class="pl-k">else</span> {
                    <span class="pl-c">// other nodes don't change</span>
                    key1<span class="pl-k">.</span>set(arr[<span class="pl-c1">0</span>]);
                    <span class="pl-k">if</span> (key<span class="pl-k">.</span>get() <span class="pl-k">==</span> <span class="pl-smi">Integer</span><span class="pl-c1"><span class="pl-k">.</span>MAX_VALUE</span>)
                        result<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>&lt;&gt;#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> arr[<span class="pl-c1">1</span>] <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span><span class="pl-k">+</span>arr[<span class="pl-c1">2</span>]<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> arr[<span class="pl-c1">3</span>]);
                    <span class="pl-k">else</span>
                        result<span class="pl-k">.</span>set(key<span class="pl-k">.</span>get() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> arr[<span class="pl-c1">1</span>] <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span><span class="pl-k">+</span>arr[<span class="pl-c1">2</span>]<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> arr[<span class="pl-c1">3</span>]);
                    context<span class="pl-k">.</span>write(key1, result);
                }
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">void</span> <span class="pl-en">main</span>(<span class="pl-k">String</span>[] <span class="pl-v">args</span>) <span class="pl-k">throws</span> <span class="pl-smi">Exception</span> {

    }
}
</pre></div>

计算最短路径上的所有顶点
PathResult.java

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal.netcare</span>;

<span class="pl-k">import</span> <span class="pl-smi">java.io.BufferedReader</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.io.IOException</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.io.InputStreamReader</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.util.ArrayList</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.util.List</span>;

<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.conf.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.FileSystem</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.Path</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.Text</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Job</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Mapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Reducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileInputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.output.FileOutputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.util.GenericOptionsParser</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">PathResult</span> {

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">PathPreMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-c">// the bigin vertex</span>
            <span class="pl-smi">Configuration</span> conf <span class="pl-k">=</span> context<span class="pl-k">.</span>getConfiguration();
            <span class="pl-smi">String</span> startVertex <span class="pl-k">=</span> conf<span class="pl-k">.</span>get(<span class="pl-s"><span class="pl-pds">"</span>startVertex<span class="pl-pds">"</span></span>);

            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-smi">Node</span> node <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Node</span>(value<span class="pl-k">.</span>toString());

            <span class="pl-k">if</span> (node<span class="pl-k">.</span>getPreId()<span class="pl-k">.</span>equals(startVertex)) {
                mapKey<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>b<span class="pl-pds">"</span></span>);
                mapValue<span class="pl-k">.</span>set(node<span class="pl-k">.</span>getPreId() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getId());
            } <span class="pl-k">else</span> {
                mapKey<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>w<span class="pl-pds">"</span></span>);
                mapValue<span class="pl-k">.</span>set(node<span class="pl-k">.</span>getPreId() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span> <span class="pl-k">+</span> node<span class="pl-k">.</span>getId());
            }
            context<span class="pl-k">.</span>write(mapKey, mapValue);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">PathPreReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                    <span class="pl-k">return</span>;
                result<span class="pl-k">.</span>set(val<span class="pl-k">.</span>toString());
                context<span class="pl-k">.</span>write(key, result);
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">PathMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;

            <span class="pl-c">// b A-C</span>
            <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (arr[<span class="pl-c1">0</span>]<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>b<span class="pl-pds">"</span></span>)) {
                <span class="pl-c">// has find the begin vertex ,get the string after the last "-" to be key</span>
                <span class="pl-smi">String</span> last <span class="pl-k">=</span> arr[<span class="pl-c1">1</span>]<span class="pl-k">.</span>substring(arr[<span class="pl-c1">1</span>]<span class="pl-k">.</span>lastIndexOf(<span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span>) <span class="pl-k">+</span> <span class="pl-c1">1</span>);
                <span class="pl-c">// C b#A-C</span>
                mapKey<span class="pl-k">.</span>set(last);
                mapValue<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>b#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> arr[<span class="pl-c1">1</span>]);
            } <span class="pl-k">else</span> {
                <span class="pl-c">// w C-E</span>
                <span class="pl-c">// haven't find the begin vertex ,get the string before the first "-" to be key</span>
                <span class="pl-smi">String</span> first <span class="pl-k">=</span> arr[<span class="pl-c1">1</span>]<span class="pl-k">.</span>substring(<span class="pl-c1">0</span>, arr[<span class="pl-c1">1</span>]<span class="pl-k">.</span>indexOf(<span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span>));
                <span class="pl-c">// C w#C-E</span>
                mapKey<span class="pl-k">.</span>set(first);
                mapValue<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>w#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> arr[<span class="pl-c1">1</span>]);
            }
            context<span class="pl-k">.</span>write(mapKey, mapValue);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">PathReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> newKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> newValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">List&lt;<span class="pl-smi">String</span>&gt;</span> preList <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-k">ArrayList&lt;<span class="pl-smi">String</span>&gt;</span>();
            <span class="pl-k">List&lt;<span class="pl-smi">String</span>&gt;</span> afterList <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-k">ArrayList&lt;<span class="pl-smi">String</span>&gt;</span>();
            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                    <span class="pl-k">return</span>;
                <span class="pl-c">// C b#A-C</span>
                <span class="pl-c">// C w#C-E</span>
                <span class="pl-c">// C w#C-B</span>
                <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span>);
                <span class="pl-k">if</span> (arr[<span class="pl-c1">0</span>]<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>b<span class="pl-pds">"</span></span>)) {
                    newKey<span class="pl-k">.</span>set(arr[<span class="pl-c1">0</span>]);
                    newValue<span class="pl-k">.</span>set(arr[<span class="pl-c1">1</span>]);
                    preList<span class="pl-k">.</span>add(arr[<span class="pl-c1">1</span>]);
                    context<span class="pl-k">.</span>write(newKey, newValue);
                } <span class="pl-k">else</span> {
                    afterList<span class="pl-k">.</span>add(arr[<span class="pl-c1">1</span>]);
                }
            }
            <span class="pl-k">if</span> (preList<span class="pl-k">.</span>size() <span class="pl-k">&gt;</span> <span class="pl-c1">0</span>) {
                <span class="pl-c">// has the black color</span>
                <span class="pl-c">//preList has the begin vertex, afterList doesn't</span>
                <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> preList<span class="pl-k">.</span>size(); i<span class="pl-k">++</span>) {
                    <span class="pl-k">for</span> (<span class="pl-k">int</span> j <span class="pl-k">=</span> <span class="pl-c1">0</span>; afterList<span class="pl-k">.</span>size() <span class="pl-k">&gt;</span> <span class="pl-c1">0</span> <span class="pl-k">&amp;&amp;</span> j <span class="pl-k">&lt;</span> afterList<span class="pl-k">.</span>size(); j<span class="pl-k">++</span>) {
                        newKey<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>b<span class="pl-pds">"</span></span>);
                        <span class="pl-c">// "A-C" and "C-E" link and delete one "C"</span>
                        newValue<span class="pl-k">.</span>set(preList<span class="pl-k">.</span>get(i) <span class="pl-k">+</span> afterList<span class="pl-k">.</span>get(j)<span class="pl-k">.</span>substring(afterList<span class="pl-k">.</span>get(j)<span class="pl-k">.</span>indexOf(<span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span>)));
                        context<span class="pl-k">.</span>write(newKey, newValue);
                    }
                }
            } <span class="pl-k">else</span> {
                <span class="pl-c">// only has white color nodes</span>
                <span class="pl-k">for</span> (<span class="pl-k">int</span> j <span class="pl-k">=</span> <span class="pl-c1">0</span>; afterList<span class="pl-k">.</span>size() <span class="pl-k">&gt;</span> <span class="pl-c1">0</span> <span class="pl-k">&amp;&amp;</span> j <span class="pl-k">&lt;</span> afterList<span class="pl-k">.</span>size(); j<span class="pl-k">++</span>) {
                    newKey<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>w<span class="pl-pds">"</span></span>);
                    newValue<span class="pl-k">.</span>set(afterList<span class="pl-k">.</span>get(j));
                    context<span class="pl-k">.</span>write(newKey, newValue);
                }
            }
            preList<span class="pl-k">.</span>clear();
            afterList<span class="pl-k">.</span>clear();
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">void</span> <span class="pl-en">main</span>(<span class="pl-k">String</span>[] <span class="pl-v">args</span>) <span class="pl-k">throws</span> <span class="pl-smi">Exception</span> {

    }

}
</pre></div>

整理最短路径结果
Result.java 

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal.netcare</span>;

<span class="pl-k">import</span> <span class="pl-smi">java.io.IOException</span>;

<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.conf.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.Path</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.Text</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Job</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Mapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Reducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileInputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.output.FileOutputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.util.GenericOptionsParser</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">Result</span> {

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">ResultValueMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (arr<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">2</span>)
                <span class="pl-k">return</span>;
            mapKey<span class="pl-k">.</span>set(arr[<span class="pl-c1">0</span>]);
            mapValue<span class="pl-k">.</span>set(arr[<span class="pl-c1">1</span>]);
            context<span class="pl-k">.</span>write(mapKey, mapValue);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">ResultValueReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                    <span class="pl-k">return</span>;
                <span class="pl-smi">String</span> res<span class="pl-k">=</span>val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>substring(<span class="pl-c1">0</span>, val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>indexOf(<span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span>));
                result<span class="pl-k">.</span>set(res);
                context<span class="pl-k">.</span>write(key, result);
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">ResultPathMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (arr<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">2</span>)
                <span class="pl-k">return</span>;
            <span class="pl-smi">String</span> keyarr<span class="pl-k">=</span>arr[<span class="pl-c1">1</span>]<span class="pl-k">.</span>substring(arr[<span class="pl-c1">1</span>]<span class="pl-k">.</span>lastIndexOf(<span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span>)<span class="pl-k">+</span><span class="pl-c1">1</span>);
            mapKey<span class="pl-k">.</span>set(keyarr);
            mapValue<span class="pl-k">.</span>set(arr[<span class="pl-c1">1</span>]);
            context<span class="pl-k">.</span>write(mapKey, mapValue);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">ResultPathReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                    <span class="pl-k">return</span>;
                result<span class="pl-k">.</span>set(val<span class="pl-k">.</span>toString());
                context<span class="pl-k">.</span>write(key, result);
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">ResultMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-k">String</span>[] arr <span class="pl-k">=</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (arr<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">2</span>)
                <span class="pl-k">return</span>;
            mapKey<span class="pl-k">.</span>set(arr[<span class="pl-c1">0</span>]);
            mapValue<span class="pl-k">.</span>set(arr[<span class="pl-c1">1</span>]);
            context<span class="pl-k">.</span>write(mapKey, mapValue);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">ResultReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-smi">String</span> res<span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>;
            <span class="pl-smi">String</span> path<span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>;
            <span class="pl-smi">String</span> value<span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>;
            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                    <span class="pl-k">return</span>;
                <span class="pl-k">if</span>(val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>contains(<span class="pl-s"><span class="pl-pds">"</span>-<span class="pl-pds">"</span></span>))
                    path<span class="pl-k">=</span>val<span class="pl-k">.</span>toString();
                <span class="pl-k">else</span>
                    value<span class="pl-k">=</span>val<span class="pl-k">.</span>toString();
            }
            res<span class="pl-k">=</span>path<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span>, <span class="pl-pds">"</span></span><span class="pl-k">+</span>value;
            result<span class="pl-k">.</span>set(res);
            context<span class="pl-k">.</span>write(key, result);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">void</span> <span class="pl-en">main</span>(<span class="pl-k">String</span>[] <span class="pl-v">args</span>) <span class="pl-k">throws</span> <span class="pl-smi">Exception</span> {

    }

}
</pre></div>

总的调用main函数：
AdjListAndDijkstraWithPath.java

<pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal.netcare</span>;

<span class="pl-k">import</span> <span class="pl-smi">java.io.BufferedReader</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.io.InputStreamReader</span>;

<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.conf.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.FileSystem</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.Path</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.IntWritable</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.Text</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Job</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileInputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.output.FileOutputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.util.GenericOptionsParser</span>;

<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.AdjListWithPath.AdjListWithPathMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.AdjListWithPath.AdjListWithPathReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.DijkstraWithPath.DijkstraWithPathMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.DijkstraWithPath.DijkstraWithPathReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.DijkstraWithPath.MinDisSortWithPathMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.DijkstraWithPath.MinDisSortWithPathReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.PathResult.PathMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.PathResult.PathPreMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.PathResult.PathPreReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.PathResult.PathReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.Result.ResultMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.Result.ResultPathMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.Result.ResultPathReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.Result.ResultReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.Result.ResultValueMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.netcare.Result.ResultValueReducer</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">AdjListAndDijkstraWithPath</span> {

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">void</span> <span class="pl-en">main</span>(<span class="pl-k">String</span>[] <span class="pl-v">args</span>) <span class="pl-k">throws</span> <span class="pl-smi">Exception</span> {
        <span class="pl-smi">Configuration</span> conf <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Configuration</span>();
        conf<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>mapred.jar<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>Dijkstra.jar<span class="pl-pds">"</span></span>);
        conf<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>startVertex<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>A<span class="pl-pds">"</span></span>);

        <span class="pl-c">// find the adjlist</span>
        <span class="pl-k">String</span>[] ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/adjlist-withpath/input<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>/syf/adjlist-withpath/output<span class="pl-pds">"</span></span> };
        <span class="pl-k">String</span>[] otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
        <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
            <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: adjList-withpath &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
            <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
        }
        <span class="pl-smi">Job</span> job0 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>adjList-withpath<span class="pl-pds">"</span></span>);
        job0<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">AdjListWithPath</span><span class="pl-k">.</span>class);
        job0<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">AdjListWithPathMapper</span><span class="pl-k">.</span>class);
        job0<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">AdjListWithPathReducer</span><span class="pl-k">.</span>class);
        job0<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        job0<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);

        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
            <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job0, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
        }
        <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job0, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
        job0<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

        <span class="pl-c">// find the shortest distance</span>
        <span class="pl-k">boolean</span> isdone <span class="pl-k">=</span> <span class="pl-c1">false</span>;
        <span class="pl-k">int</span> num <span class="pl-k">=</span> <span class="pl-c1">0</span>;
        <span class="pl-k">while</span> (<span class="pl-k">!</span>isdone) {
            <span class="pl-k">if</span> (num <span class="pl-k">==</span> <span class="pl-c1">0</span>)
                ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/adjlist-withpath/output<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_dijkstra<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num };
            <span class="pl-k">else</span>
                ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_mindis<span class="pl-pds">"</span></span> <span class="pl-k">+</span> (num <span class="pl-k">-</span> <span class="pl-c1">1</span>), <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_dijkstra<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num };
            otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
            <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
                <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: dijkstra<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span> &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
                <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
            }
            <span class="pl-smi">Job</span> job1 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>dijkstra-withpath<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num);
            job1<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">Dijkstra</span><span class="pl-k">.</span>class);
            job1<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">DijkstraWithPathMapper</span><span class="pl-k">.</span>class);
            job1<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">DijkstraWithPathReducer</span><span class="pl-k">.</span>class);
            job1<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
            job1<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);

            <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
                <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job1, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
            }
            <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job1, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
            job1<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

            <span class="pl-c">//</span>
            ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_dijkstra<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num, <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_mindis<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num };
            otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
            <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
                <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: mindis-withpath &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
                <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
            }
            <span class="pl-smi">Job</span> job2 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>mindis-withpath<span class="pl-pds">"</span></span>);
            job2<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">Dijkstra</span><span class="pl-k">.</span>class);
            job2<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">MinDisSortWithPathMapper</span><span class="pl-k">.</span>class);
            job2<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">MinDisSortWithPathReducer</span><span class="pl-k">.</span>class);

            job2<span class="pl-k">.</span>setMapOutputKeyClass(<span class="pl-smi">IntWritable</span><span class="pl-k">.</span>class);
            job2<span class="pl-k">.</span>setMapOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
            job2<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
            job2<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
            <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
                <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job2, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
            }
            <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job2, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
            job2<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

            <span class="pl-c">// check is end?</span>
            isdone <span class="pl-k">=</span> <span class="pl-c1">true</span>;
            <span class="pl-smi">Path</span> file <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Path</span>(<span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_mindis<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>/part-r-00000<span class="pl-pds">"</span></span>);
            <span class="pl-smi">FileSystem</span> fs <span class="pl-k">=</span> <span class="pl-smi">FileSystem</span><span class="pl-k">.</span>get(conf);
            <span class="pl-smi">BufferedReader</span> br <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">BufferedReader</span>(<span class="pl-k">new</span> <span class="pl-smi">InputStreamReader</span>(fs<span class="pl-k">.</span>open(file)));

            <span class="pl-smi">String</span> line <span class="pl-k">=</span> br<span class="pl-k">.</span>readLine();
            <span class="pl-k">while</span> (line <span class="pl-k">!=</span> <span class="pl-c1">null</span>) {
                <span class="pl-smi">String</span> color <span class="pl-k">=</span> line<span class="pl-k">.</span>substring(line<span class="pl-k">.</span>toString()<span class="pl-k">.</span>lastIndexOf(<span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span>) <span class="pl-k">+</span> <span class="pl-c1">1</span>);
                <span class="pl-smi">System</span><span class="pl-k">.</span>out<span class="pl-k">.</span>println(color);
                <span class="pl-k">if</span> (color<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>g<span class="pl-pds">"</span></span>)) {
                    isdone <span class="pl-k">=</span> <span class="pl-c1">false</span>;
                    <span class="pl-k">break</span>;
                }
                line <span class="pl-k">=</span> br<span class="pl-k">.</span>readLine();
            }

            num<span class="pl-k">++</span>;
        }

        <span class="pl-c">// style shortest distance result</span>
        ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_dijkstra<span class="pl-pds">"</span></span> <span class="pl-k">+</span> (num <span class="pl-k">-</span> <span class="pl-c1">1</span>), <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/result_value<span class="pl-pds">"</span></span> };
        otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
        <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
            <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: result_value-withpath &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
            <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
        }
        <span class="pl-smi">Job</span> job3 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>result_value-withpath<span class="pl-pds">"</span></span>);
        job3<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">Result</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">ResultValueMapper</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">ResultValueReducer</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);

        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
            <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job3, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
        }
        <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job3, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
        job3<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

        <span class="pl-c">// first path job</span>
        ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_dijkstra<span class="pl-pds">"</span></span> <span class="pl-k">+</span> (num <span class="pl-k">-</span> <span class="pl-c1">1</span>), <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_path0<span class="pl-pds">"</span></span> };
        otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
        <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
            <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: path0 &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
            <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
        }
        <span class="pl-smi">Job</span> job4 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>path0<span class="pl-pds">"</span></span>);
        job4<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">PathResult</span><span class="pl-k">.</span>class);
        job4<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">PathPreMapper</span><span class="pl-k">.</span>class);
        job4<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">PathPreReducer</span><span class="pl-k">.</span>class);
        job4<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        job4<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);

        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
            <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job4, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
        }
        <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job4, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
        job4<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

        <span class="pl-c">// other path jobs</span>
        <span class="pl-k">boolean</span> isdone1 <span class="pl-k">=</span> <span class="pl-c1">false</span>;
        <span class="pl-k">int</span> num1 <span class="pl-k">=</span> <span class="pl-c1">1</span>;
        <span class="pl-k">while</span> (<span class="pl-k">!</span>isdone1) {
            ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_path<span class="pl-pds">"</span></span> <span class="pl-k">+</span> (num1 <span class="pl-k">-</span> <span class="pl-c1">1</span>), <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_path<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num1 };
            otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
            <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
                <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: path<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num1 <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span> &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
                <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
            }
            <span class="pl-smi">Job</span> job5 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>path<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num1);
            job5<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">PathResult</span><span class="pl-k">.</span>class);
            job5<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">PathMapper</span><span class="pl-k">.</span>class);
            job5<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">PathReducer</span><span class="pl-k">.</span>class);
            job5<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
            job5<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);

            <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
                <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job5, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
            }
            <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job5, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
            job5<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

            <span class="pl-c">// check is end?</span>
            isdone1 <span class="pl-k">=</span> <span class="pl-c1">true</span>;
            <span class="pl-smi">Path</span> file <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Path</span>(<span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_path<span class="pl-pds">"</span></span> <span class="pl-k">+</span> num1 <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>/part-r-00000<span class="pl-pds">"</span></span>);
            <span class="pl-smi">FileSystem</span> fs <span class="pl-k">=</span> <span class="pl-smi">FileSystem</span><span class="pl-k">.</span>get(conf);
            <span class="pl-smi">BufferedReader</span> br <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">BufferedReader</span>(<span class="pl-k">new</span> <span class="pl-smi">InputStreamReader</span>(fs<span class="pl-k">.</span>open(file)));

            <span class="pl-smi">String</span> line <span class="pl-k">=</span> br<span class="pl-k">.</span>readLine();
            <span class="pl-k">while</span> (line <span class="pl-k">!=</span> <span class="pl-c1">null</span>) {
                <span class="pl-smi">String</span> color <span class="pl-k">=</span> line<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>)[<span class="pl-c1">0</span>];
                <span class="pl-smi">System</span><span class="pl-k">.</span>out<span class="pl-k">.</span>println(color);
                <span class="pl-k">if</span> (color<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span>w<span class="pl-pds">"</span></span>)) {
                    isdone1 <span class="pl-k">=</span> <span class="pl-c1">false</span>;
                    <span class="pl-k">break</span>;
                }
                line <span class="pl-k">=</span> br<span class="pl-k">.</span>readLine();
            }

            num1<span class="pl-k">++</span>;
        }

        <span class="pl-c">// style the total result</span>
        ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/output_path<span class="pl-pds">"</span></span> <span class="pl-k">+</span> (num1 <span class="pl-k">-</span> <span class="pl-c1">1</span>), <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/result_path<span class="pl-pds">"</span></span> };
        otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
        <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
            <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: result_path-withpath &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
            <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
        }
        <span class="pl-smi">Job</span> job6 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>result_path-withpath<span class="pl-pds">"</span></span>);
        job6<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">Result</span><span class="pl-k">.</span>class);
        job6<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">ResultPathMapper</span><span class="pl-k">.</span>class);
        job6<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">ResultPathReducer</span><span class="pl-k">.</span>class);
        job6<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        job6<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);

        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
            <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job6, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
        }
        <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job6, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
        job6<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

        <span class="pl-c">//</span>
        ars <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">String</span>[] { <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/result_value<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/result_path<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>/syf/dijkstra-withpath/output/result<span class="pl-pds">"</span></span> };
        otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
        <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">3</span>) {
            <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: result-withpath &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
            <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
        }
        <span class="pl-smi">Job</span> job7 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>result-withpath<span class="pl-pds">"</span></span>);
        job7<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">Result</span><span class="pl-k">.</span>class);
        job7<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">ResultMapper</span><span class="pl-k">.</span>class);
        job7<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">ResultReducer</span><span class="pl-k">.</span>class);
        job7<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        job7<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);

        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
            <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job7, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
        }
        <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job7, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
        job7<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

    }

}
</pre>

最后，迭代计算并不适合用MapReduce框架来计算，因为读写HDFS过于频繁，效率低下。还是适合用专门的图计算框架，如spark的Graphx或者GraphLab等。

