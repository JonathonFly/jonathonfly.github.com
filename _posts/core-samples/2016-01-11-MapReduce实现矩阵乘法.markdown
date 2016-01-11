---
layout: post
category : Hadoop
tagline: 
tags : [hadoop, mapreduce, matrix multiply]
---
{% include JB/setup %}

由于项目需要，前段时间在研究Hadoop下的MapReduce，主要是将普通的算法转化为可在Hadoop集群下运行的MapReduce算法。

MapReduce程序一般是将HDFS文件系统中的文件作为输入和输出，本例的输入形式如下：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt1.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt1.JPG" style="max-width:100%;"></a>

输入文件A矩阵：matrix_A.txt，B矩阵文件matrix_B.txt，在HDFS中的形式如下：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/1.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/1.png" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/2.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/2.png" style="max-width:100%;"></a>

要实现如何计算mxn的矩阵与nxp的矩阵相乘，首先，我们先来看一下3x3的矩阵相乘，以便有一个直观的印象。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt2.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt2.JPG" style="max-width:100%;"></a>

左侧矩阵的每一行的元素与右侧矩阵的每一列的对应元素相乘，然后将相乘结果相加，得到结果矩阵每个位置的计算结果。由于MapReduce中的数据只能遍历一次，第二次读的时候就会变为空，所以，首先我们要计算出矩阵中每一个元素需要复制多少份，才能满足A，B各个元素相乘的需求。我们可以看到，A矩阵第1行，第1列的元素A1,1分别与B1,1、B1,2、B1,3三个元素相乘。即A1,1需要复制3份，而该份数正好是B的列数。同理，B中的每个元素需要复制的份数为A的行数。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt3.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt3.JPG" style="max-width:100%;"></a>

那么，知道一个元素需复制的份数后，下一个问题就是：这些元素以何种形式进行进行存储呢？
如A1,1这个元素，需要分别与B1,1、B1,2、B1,3三个元素相乘，那么我们可以用如下形式表示：
(1#1#1 A1,1)(1#2#1 A1,1)(1#3#1 A1,1)，其中一个括号代表一行数据，A1,1代表矩阵第一行第一列的值。1#2#1中第一个数字1代表结果矩阵第1行，即A1,1的行号；第二个数字2代表结果矩阵的第2列，即B1,2的列号；第3个数字1代表A1,1的列号和B1,2的行号。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt4.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt4.JPG" style="max-width:100%;"></a>

这样的存储形式的优势是什么？同一个key，如1#2#1，只会有2个元素，即一个A1,1和一个B1,2。可以直接将这两个元素相乘。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt5.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt5.JPG" style="max-width:100%;"></a>

同理，可以算出B的元素复制和元素表示。如B1,2可以表示为：(1#2#1 B1,2)(2#2#1 B1,2)(3#2#1 B1,2)

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt6.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt6.JPG" style="max-width:100%;"></a>

通用公式如下：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt7.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt7.JPG" style="max-width:100%;"></a>

在reduce的时候，相同的key的值在同一个节点计算，可以直接相乘。如key为1#2#1，结果为A1,1 * B1,2，输出结果为:1#2 A1,1 * B1,2 

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt8.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt8.JPG" style="max-width:100%;"></a>

通用的在第一个reduce阶段进行乘法计算。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt9.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt9.JPG" style="max-width:100%;"></a>

第二个阶段进行加法计算。

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt10.JPG" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/ppt10.JPG" style="max-width:100%;"></a>

一些HDFS计算步骤的结果：

第一次reduce结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/3.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/3.png" style="max-width:100%;"></a>

第二次reduce结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/4.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/4.png" style="max-width:100%;"></a>

Sort结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/5.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/5.png" style="max-width:100%;"></a>

目录结构：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/6.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-01-11/6.png" style="max-width:100%;"></a>

总的代码如下：

MatrixMultiply.java

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal</span>;

<span class="pl-k">import</span> <span class="pl-smi">java.io.IOException</span>;

<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.conf.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.Path</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.Text</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Job</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Mapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Reducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileInputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileSplit</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.output.FileOutputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.util.GenericOptionsParser</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.log4j.Logger</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">MatrixMultiply</span> {
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-smi">Logger</span> logger <span class="pl-k">=</span> <span class="pl-smi">Logger</span><span class="pl-k">.</span>getLogger(<span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span>class);
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-smi">Integer</span> ARowNum <span class="pl-k">=</span> <span class="pl-c1">0</span>;
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-smi">Integer</span> AColNum <span class="pl-k">=</span> <span class="pl-c1">0</span>;
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-smi">Integer</span> BRowNum <span class="pl-k">=</span> <span class="pl-c1">0</span>;
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-smi">Integer</span> BColNum <span class="pl-k">=</span> <span class="pl-c1">0</span>;

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">MatrixMultiplyPreMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-smi">Configuration</span> conf <span class="pl-k">=</span> context<span class="pl-k">.</span>getConfiguration();
            <span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span><span class="pl-smi">ARowNum</span> <span class="pl-k">=</span> conf<span class="pl-k">.</span>getInt(<span class="pl-s"><span class="pl-pds">"</span>ARowNum<span class="pl-pds">"</span></span>, <span class="pl-c1">0</span>);
            <span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span><span class="pl-smi">AColNum</span> <span class="pl-k">=</span> conf<span class="pl-k">.</span>getInt(<span class="pl-s"><span class="pl-pds">"</span>AColNum<span class="pl-pds">"</span></span>, <span class="pl-c1">0</span>);
            <span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span><span class="pl-smi">BRowNum</span> <span class="pl-k">=</span> conf<span class="pl-k">.</span>getInt(<span class="pl-s"><span class="pl-pds">"</span>BRowNum<span class="pl-pds">"</span></span>, <span class="pl-c1">0</span>);
            <span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span><span class="pl-smi">BColNum</span> <span class="pl-k">=</span> conf<span class="pl-k">.</span>getInt(<span class="pl-s"><span class="pl-pds">"</span>BColNum<span class="pl-pds">"</span></span>, <span class="pl-c1">0</span>);
            <span class="pl-k">if</span> (<span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span><span class="pl-smi">BRowNum</span> <span class="pl-k">!=</span> <span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span><span class="pl-smi">AColNum</span>)
                <span class="pl-k">return</span>;
            <span class="pl-smi">String</span> pathName <span class="pl-k">=</span> ((<span class="pl-smi">FileSplit</span>) context<span class="pl-k">.</span>getInputSplit())<span class="pl-k">.</span>getPath()<span class="pl-k">.</span>toString();
            logger<span class="pl-k">.</span>info(pathName);
            <span class="pl-c">// get matrix value</span>
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-k">String</span>[] rowVal <span class="pl-k">=</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>#|<span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (rowVal<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">3</span>)
                <span class="pl-k">return</span>;

            <span class="pl-k">if</span> (pathName<span class="pl-k">.</span>contains(<span class="pl-s"><span class="pl-pds">"</span>matrix_A<span class="pl-pds">"</span></span>)) {
                <span class="pl-k">int</span> m <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">0</span>]);
                <span class="pl-k">int</span> n <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">1</span>]);
                <span class="pl-k">int</span> numVal <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">2</span>]);
                <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">1</span>; i <span class="pl-k">&lt;=</span> <span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span><span class="pl-smi">BColNum</span>; i<span class="pl-k">++</span>) {
                    mapKey<span class="pl-k">.</span>set(m <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> i <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> n);
                    mapValue<span class="pl-k">.</span>set(numVal<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>);
                    context<span class="pl-k">.</span>write(mapKey, mapValue);
                }
            } <span class="pl-k">else</span> <span class="pl-k">if</span> (pathName<span class="pl-k">.</span>contains(<span class="pl-s"><span class="pl-pds">"</span>matrix_B<span class="pl-pds">"</span></span>)) {
                <span class="pl-k">int</span> n <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">0</span>]);
                <span class="pl-k">int</span> k <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">1</span>]);
                <span class="pl-k">int</span> numVal <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">2</span>]);

                <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">1</span>; i <span class="pl-k">&lt;=</span> <span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span><span class="pl-smi">ARowNum</span>; i<span class="pl-k">++</span>) {
                    mapKey<span class="pl-k">.</span>set(i <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> k <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span> <span class="pl-k">+</span> n);
                    mapValue<span class="pl-k">.</span>set(numVal<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>);
                    context<span class="pl-k">.</span>write(mapKey, mapValue);
                }
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">MatrixMultiplyPreReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> resKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">int</span> res<span class="pl-k">=</span><span class="pl-c1">1</span>;
            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                <span class="pl-k">if</span> (val<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> val<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                    <span class="pl-k">return</span>;
                res<span class="pl-k">*=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(val<span class="pl-k">.</span>toString());
            }
            result<span class="pl-k">.</span>set(res <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>);

            <span class="pl-smi">String</span> newKey<span class="pl-k">=</span>key<span class="pl-k">.</span>toString()<span class="pl-k">.</span>substring(<span class="pl-c1">0</span>,key<span class="pl-k">.</span>toString()<span class="pl-k">.</span>lastIndexOf(<span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span>));
            resKey<span class="pl-k">.</span>set(newKey);
            context<span class="pl-k">.</span>write(resKey, result);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">MatrixMultiplyMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {

        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapKey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> mapValue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-k">String</span>[] rowVal <span class="pl-k">=</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (rowVal<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">2</span>)
                <span class="pl-k">return</span>;
            mapKey<span class="pl-k">.</span>set(rowVal[<span class="pl-c1">0</span>]);
            mapValue<span class="pl-k">.</span>set(rowVal[<span class="pl-c1">1</span>]);
            context<span class="pl-k">.</span>write(mapKey, mapValue);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">MatrixMultiplyReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">Text</span>&gt;</span> {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> result <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">Text</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">Text</span>&gt;</span> <span class="pl-v">values</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span> {
            <span class="pl-k">int</span> sum<span class="pl-k">=</span><span class="pl-c1">0</span>;
            <span class="pl-k">for</span> (<span class="pl-smi">Text</span> val <span class="pl-k">:</span> values) {
                sum<span class="pl-k">+=</span><span class="pl-smi">Integer</span><span class="pl-k">.</span>valueOf(val<span class="pl-k">.</span>toString());
            }
            result<span class="pl-k">.</span>set(sum<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>);
            context<span class="pl-k">.</span>write(key, result);
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">void</span> <span class="pl-en">main</span>(<span class="pl-k">String</span>[] <span class="pl-v">args</span>) <span class="pl-k">throws</span> <span class="pl-smi">Exception</span> {

    }

}
</pre></div>

将计算结果排个序：

MatrixSort.java

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal</span>;

<span class="pl-k">import</span> <span class="pl-smi">java.io.DataInput</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.io.DataOutput</span>;
<span class="pl-k">import</span> <span class="pl-smi">java.io.IOException</span>;

<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.conf.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.Path</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.IntWritable</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.Text</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.WritableComparable</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.WritableComparator</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Job</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Mapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Partitioner</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Reducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileInputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.output.FileOutputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.util.GenericOptionsParser</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.log4j.Logger</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">MatrixSort</span> {
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-smi">Logger</span> logger <span class="pl-k">=</span> <span class="pl-smi">Logger</span><span class="pl-k">.</span>getLogger(<span class="pl-smi">MatrixSort</span><span class="pl-k">.</span>class);

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">IntPair</span> <span class="pl-k">implements</span> <span class="pl-e">WritableComparable&lt;<span class="pl-smi">IntPair</span>&gt;</span>
    {
        <span class="pl-k">int</span> first;
        <span class="pl-k">int</span> second;

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">set</span>(<span class="pl-k">int</span> <span class="pl-v">left</span>, <span class="pl-k">int</span> <span class="pl-v">right</span>)
        {
            first <span class="pl-k">=</span> left;
            second <span class="pl-k">=</span> right;
        }
        <span class="pl-k">public</span> <span class="pl-k">int</span> <span class="pl-en">getFirst</span>()
        {
            <span class="pl-k">return</span> first;
        }
        <span class="pl-k">public</span> <span class="pl-k">int</span> <span class="pl-en">getSecond</span>()
        {
            <span class="pl-k">return</span> second;
        }
        <span class="pl-k">@Override</span>

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">readFields</span>(<span class="pl-smi">DataInput</span> <span class="pl-v">in</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>
        {
            <span class="pl-c">// TODO Auto-generated method stub</span>
            first <span class="pl-k">=</span> in<span class="pl-k">.</span>readInt();
            second <span class="pl-k">=</span> in<span class="pl-k">.</span>readInt();
        }
        <span class="pl-k">@Override</span>

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">write</span>(<span class="pl-smi">DataOutput</span> <span class="pl-v">out</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>
        {
            <span class="pl-c">// TODO Auto-generated method stub</span>
            out<span class="pl-k">.</span>writeInt(first);
            out<span class="pl-k">.</span>writeInt(second);
        }
        <span class="pl-k">@Override</span>

        <span class="pl-k">public</span> <span class="pl-k">int</span> <span class="pl-en">compareTo</span>(<span class="pl-smi">IntPair</span> <span class="pl-v">o</span>)
        {
            <span class="pl-c">// TODO Auto-generated method stub</span>
            <span class="pl-k">if</span> (first <span class="pl-k">!=</span> o<span class="pl-k">.</span>first)
            {
                <span class="pl-k">return</span> first <span class="pl-k">&lt;</span> o<span class="pl-k">.</span>first <span class="pl-k">?</span> <span class="pl-k">-</span><span class="pl-c1">1</span> <span class="pl-k">:</span> <span class="pl-c1">1</span>;
            }
            <span class="pl-k">else</span> <span class="pl-k">if</span> (second <span class="pl-k">!=</span> o<span class="pl-k">.</span>second)
            {
                <span class="pl-k">return</span> second <span class="pl-k">&lt;</span> o<span class="pl-k">.</span>second <span class="pl-k">?</span> <span class="pl-k">-</span><span class="pl-c1">1</span> <span class="pl-k">:</span> <span class="pl-c1">1</span>;
            }
            <span class="pl-k">else</span>
            {
                <span class="pl-k">return</span> <span class="pl-c1">0</span>;
            }
        }

        <span class="pl-k">@Override</span>
        <span class="pl-c">//The hashCode() method is used by the HashPartitioner (the default partitioner in MapReduce)</span>
        <span class="pl-k">public</span> <span class="pl-k">int</span> <span class="pl-en">hashCode</span>()
        {
            <span class="pl-k">return</span> first <span class="pl-k">*</span> <span class="pl-c1">157</span> <span class="pl-k">+</span> second;
        }
        <span class="pl-k">@Override</span>
        <span class="pl-k">public</span> <span class="pl-k">boolean</span> <span class="pl-en">equals</span>(<span class="pl-smi">Object</span> <span class="pl-v">right</span>)
        {
            <span class="pl-k">if</span> (right <span class="pl-k">==</span> <span class="pl-c1">null</span>)
                <span class="pl-k">return</span> <span class="pl-c1">false</span>;
            <span class="pl-k">if</span> (<span class="pl-v">this</span> <span class="pl-k">==</span> right)
                <span class="pl-k">return</span> <span class="pl-c1">true</span>;
            <span class="pl-k">if</span> (right <span class="pl-k">instanceof</span> <span class="pl-smi">IntPair</span>)
            {
                <span class="pl-smi">IntPair</span> r <span class="pl-k">=</span> (<span class="pl-smi">IntPair</span>) right;
                <span class="pl-k">return</span> r<span class="pl-k">.</span>first <span class="pl-k">==</span> first <span class="pl-k">&amp;&amp;</span> r<span class="pl-k">.</span>second <span class="pl-k">==</span> second;
            }
            <span class="pl-k">else</span>
            {
                <span class="pl-k">return</span> <span class="pl-c1">false</span>;
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">FirstPartitioner</span> <span class="pl-k">extends</span> <span class="pl-e">Partitioner&lt;<span class="pl-smi">IntPair</span>, <span class="pl-smi">IntWritable</span>&gt;</span>
    {
        <span class="pl-k">@Override</span>
        <span class="pl-k">public</span> <span class="pl-k">int</span> <span class="pl-en">getPartition</span>(<span class="pl-smi">IntPair</span> <span class="pl-v">key</span>, <span class="pl-smi">IntWritable</span> <span class="pl-v">value</span>,<span class="pl-k">int</span> <span class="pl-v">numPartitions</span>)
        {
            <span class="pl-k">return</span> <span class="pl-smi">Math</span><span class="pl-k">.</span>abs(key<span class="pl-k">.</span>getFirst() <span class="pl-k">*</span> <span class="pl-c1">127</span>) <span class="pl-k">%</span> numPartitions;
        }
    }


    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">GroupingComparator</span> <span class="pl-k">extends</span> <span class="pl-e">WritableComparator</span>
    {
        <span class="pl-k">protected</span> <span class="pl-en">GroupingComparator</span>()
        {
            <span class="pl-v">super</span>(<span class="pl-smi">IntPair</span><span class="pl-k">.</span>class, <span class="pl-c1">true</span>);
        }
        <span class="pl-k">@SuppressWarnings</span>(<span class="pl-s"><span class="pl-pds">"</span>rawtypes<span class="pl-pds">"</span></span>)
        <span class="pl-k">@Override</span>
        <span class="pl-c">//Compare two WritableComparables.</span>
        <span class="pl-k">public</span> <span class="pl-k">int</span> <span class="pl-en">compare</span>(<span class="pl-smi">WritableComparable</span> <span class="pl-v">w1</span>, <span class="pl-smi">WritableComparable</span> <span class="pl-v">w2</span>)
        {
            <span class="pl-smi">IntPair</span> ip1 <span class="pl-k">=</span> (<span class="pl-smi">IntPair</span>) w1;
            <span class="pl-smi">IntPair</span> ip2 <span class="pl-k">=</span> (<span class="pl-smi">IntPair</span>) w2;
            <span class="pl-k">int</span> l <span class="pl-k">=</span> ip1<span class="pl-k">.</span>getSecond();
            <span class="pl-k">int</span> r <span class="pl-k">=</span> ip2<span class="pl-k">.</span>getSecond();
            <span class="pl-k">return</span> l <span class="pl-k">==</span> r <span class="pl-k">?</span> <span class="pl-c1">0</span> <span class="pl-k">:</span> (l <span class="pl-k">&lt;</span> r <span class="pl-k">?</span> <span class="pl-k">-</span><span class="pl-c1">1</span> <span class="pl-k">:</span> <span class="pl-c1">1</span>);
        }
    }



    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">MatrixSortMapper</span> <span class="pl-k">extends</span> <span class="pl-e">Mapper&lt;<span class="pl-smi">Object</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">IntPair</span>, <span class="pl-smi">IntWritable</span>&gt;</span>
    {
        <span class="pl-k">private</span> <span class="pl-smi">IntPair</span> intkey <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">IntPair</span>();
        <span class="pl-k">private</span> <span class="pl-smi">IntWritable</span> intvalue <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">IntWritable</span>();
        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">map</span>(<span class="pl-smi">Object</span> <span class="pl-v">key</span>, <span class="pl-smi">Text</span> <span class="pl-v">value</span>, <span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span>
        {
            logger<span class="pl-k">.</span>info(<span class="pl-s"><span class="pl-pds">"</span>map start<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (value<span class="pl-k">.</span>toString() <span class="pl-k">==</span> <span class="pl-c1">null</span> <span class="pl-k">||</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>equals(<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>))
                <span class="pl-k">return</span>;
            <span class="pl-k">String</span>[] rowVal <span class="pl-k">=</span> value<span class="pl-k">.</span>toString()<span class="pl-k">.</span>split(<span class="pl-s"><span class="pl-pds">"</span>#|<span class="pl-cce">\\</span>s+<span class="pl-pds">"</span></span>);
            <span class="pl-k">if</span> (rowVal<span class="pl-k">.</span>length <span class="pl-k">&lt;</span> <span class="pl-c1">3</span>)
                <span class="pl-k">return</span>;

            <span class="pl-k">int</span> left <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">0</span>]);
            <span class="pl-k">int</span> right <span class="pl-k">=</span> <span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">1</span>]);
            <span class="pl-k">int</span> num<span class="pl-k">=</span><span class="pl-smi">Integer</span><span class="pl-k">.</span>parseInt(rowVal[<span class="pl-c1">2</span>]);
            intkey<span class="pl-k">.</span>set(left, right);
            intvalue<span class="pl-k">.</span>set(num);
            logger<span class="pl-k">.</span>info(left<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span> <span class="pl-pds">"</span></span><span class="pl-k">+</span>right<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span> <span class="pl-pds">"</span></span><span class="pl-k">+</span>num);
            context<span class="pl-k">.</span>write(intkey, intvalue);
        }
    }

    <span class="pl-c">//</span>
    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">class</span> <span class="pl-en">MatrixSortReducer</span> <span class="pl-k">extends</span> <span class="pl-e">Reducer&lt;<span class="pl-smi">IntPair</span>, <span class="pl-smi">IntWritable</span>, <span class="pl-smi">Text</span>, <span class="pl-smi">IntWritable</span>&gt;</span>
    {
        <span class="pl-k">private</span> <span class="pl-smi">Text</span> left <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Text</span>();

        <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">reduce</span>(<span class="pl-smi">IntPair</span> <span class="pl-v">key</span>, <span class="pl-k">Iterable&lt;<span class="pl-smi">IntWritable</span>&gt;</span> <span class="pl-v">values</span>,<span class="pl-smi">Context</span> <span class="pl-v">context</span>) <span class="pl-k">throws</span> <span class="pl-smi">IOException</span>, <span class="pl-smi">InterruptedException</span>
        {
            left<span class="pl-k">.</span>set(key<span class="pl-k">.</span>getFirst()<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span>#<span class="pl-pds">"</span></span><span class="pl-k">+</span>key<span class="pl-k">.</span>getSecond());
            <span class="pl-k">for</span> (<span class="pl-smi">IntWritable</span> val <span class="pl-k">:</span> values)
            {
                context<span class="pl-k">.</span>write(left, val);
                logger<span class="pl-k">.</span>info(left<span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">"</span> <span class="pl-pds">"</span></span><span class="pl-k">+</span>val);
            }
        }
    }

    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">void</span> <span class="pl-en">main</span>(<span class="pl-k">String</span>[] <span class="pl-v">args</span>) <span class="pl-k">throws</span> <span class="pl-smi">Exception</span> {

    }

}
</pre></div>

总的调用main方法：

MatrixMultiplyAndSort.java

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.ideal</span>;

<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.conf.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.fs.Path</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.IntWritable</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.io.Text</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.Job</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.input.FileInputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.mapreduce.lib.output.FileOutputFormat</span>;
<span class="pl-k">import</span> <span class="pl-smi">org.apache.hadoop.util.GenericOptionsParser</span>;

<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixMultiply.MatrixMultiplyMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixMultiply.MatrixMultiplyPreMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixMultiply.MatrixMultiplyPreReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixMultiply.MatrixMultiplyReducer</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixSort.FirstPartitioner</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixSort.GroupingComparator</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixSort.IntPair</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixSort.MatrixSortMapper</span>;
<span class="pl-k">import</span> <span class="pl-smi">com.ideal.MatrixSort.MatrixSortReducer</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">MatrixMultiplyAndSort</span> {
    <span class="pl-k">public</span> <span class="pl-k">static</span> <span class="pl-k">void</span> <span class="pl-en">main</span>(<span class="pl-k">String</span>[] <span class="pl-v">args</span>) <span class="pl-k">throws</span> <span class="pl-smi">Exception</span> {
        <span class="pl-smi">Configuration</span> conf <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">Configuration</span>();
        conf<span class="pl-k">.</span>set(<span class="pl-s"><span class="pl-pds">"</span>mapred.jar<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>MatrixMultiply.jar<span class="pl-pds">"</span></span>);

        conf<span class="pl-k">.</span>setInt(<span class="pl-s"><span class="pl-pds">"</span>ARowNum<span class="pl-pds">"</span></span>, <span class="pl-c1">3</span>);
        conf<span class="pl-k">.</span>setInt(<span class="pl-s"><span class="pl-pds">"</span>AColNum<span class="pl-pds">"</span></span>, <span class="pl-c1">3</span>);
        conf<span class="pl-k">.</span>setInt(<span class="pl-s"><span class="pl-pds">"</span>BRowNum<span class="pl-pds">"</span></span>, <span class="pl-c1">3</span>);
        conf<span class="pl-k">.</span>setInt(<span class="pl-s"><span class="pl-pds">"</span>BColNum<span class="pl-pds">"</span></span>, <span class="pl-c1">3</span>);

        <span class="pl-k">String</span>[] ars<span class="pl-k">=</span><span class="pl-k">new</span> <span class="pl-smi">String</span>[]{<span class="pl-s"><span class="pl-pds">"</span>/syf/matrix_test/input/input1<span class="pl-pds">"</span></span>,<span class="pl-s"><span class="pl-pds">"</span>/syf/matrix_test/output/output_multiply_1<span class="pl-pds">"</span></span>};  
        <span class="pl-k">String</span>[] otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
        <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
            <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: matrix multiply1 &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
            <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
        }
        <span class="pl-smi">Job</span> job1 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>matrix multiply1<span class="pl-pds">"</span></span>);
        job1<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span>class);
        job1<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">MatrixMultiplyPreMapper</span><span class="pl-k">.</span>class);
        job1<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">MatrixMultiplyPreReducer</span><span class="pl-k">.</span>class);
        job1<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        job1<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);

        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
            <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job1, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
        }
        <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job1, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
        job1<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

        <span class="pl-c">//</span>
        ars<span class="pl-k">=</span><span class="pl-k">new</span> <span class="pl-smi">String</span>[]{<span class="pl-s"><span class="pl-pds">"</span>/syf/matrix_test/output/output_multiply_1<span class="pl-pds">"</span></span>,<span class="pl-s"><span class="pl-pds">"</span>/syf/matrix_test/output/output_multiply_2<span class="pl-pds">"</span></span>}; 
        otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
        <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
            <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: matrix multiply2 &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
            <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
        }
        <span class="pl-smi">Job</span> job2 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>matrix multiply2<span class="pl-pds">"</span></span>);
        job2<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">MatrixMultiply</span><span class="pl-k">.</span>class);
        job2<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">MatrixMultiplyMapper</span><span class="pl-k">.</span>class);
        job2<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">MatrixMultiplyReducer</span><span class="pl-k">.</span>class);

        job2<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        job2<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
            <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job2, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
        }
        <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job2, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
        job2<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);

        <span class="pl-c">//</span>
        ars<span class="pl-k">=</span><span class="pl-k">new</span> <span class="pl-smi">String</span>[]{<span class="pl-s"><span class="pl-pds">"</span>/syf/matrix_test/output/output_multiply_2<span class="pl-pds">"</span></span>,<span class="pl-s"><span class="pl-pds">"</span>/syf/matrix_test/output/output_sort<span class="pl-pds">"</span></span>}; 
        otherArgs <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">GenericOptionsParser</span>(conf, ars)<span class="pl-k">.</span>getRemainingArgs();
        <span class="pl-k">if</span> (otherArgs<span class="pl-k">.</span>length <span class="pl-k">!=</span> <span class="pl-c1">2</span>) {
            <span class="pl-smi">System</span><span class="pl-k">.</span>err<span class="pl-k">.</span>println(<span class="pl-s"><span class="pl-pds">"</span>usage: matrix sort &lt;in&gt; [&lt;in&gt;...] &lt;out&gt; <span class="pl-pds">"</span></span>);
            <span class="pl-smi">System</span><span class="pl-k">.</span>exit(<span class="pl-c1">2</span>);
        }
        <span class="pl-smi">Job</span> job3 <span class="pl-k">=</span> <span class="pl-smi">Job</span><span class="pl-k">.</span>getInstance(conf, <span class="pl-s"><span class="pl-pds">"</span>matrix sort<span class="pl-pds">"</span></span>);
        job3<span class="pl-k">.</span>setJarByClass(<span class="pl-smi">MatrixSort</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setMapperClass(<span class="pl-smi">MatrixSortMapper</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setReducerClass(<span class="pl-smi">MatrixSortReducer</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setPartitionerClass(<span class="pl-smi">FirstPartitioner</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setGroupingComparatorClass(<span class="pl-smi">GroupingComparator</span><span class="pl-k">.</span>class);

        job3<span class="pl-k">.</span>setMapOutputKeyClass(<span class="pl-smi">IntPair</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setMapOutputValueClass(<span class="pl-smi">IntWritable</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setOutputKeyClass(<span class="pl-smi">Text</span><span class="pl-k">.</span>class);
        job3<span class="pl-k">.</span>setOutputValueClass(<span class="pl-smi">IntWritable</span><span class="pl-k">.</span>class);
        <span class="pl-k">for</span> (<span class="pl-k">int</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>; <span class="pl-k">++</span>i) {
            <span class="pl-smi">FileInputFormat</span><span class="pl-k">.</span>addInputPath(job3, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[i]));
        }
        <span class="pl-smi">FileOutputFormat</span><span class="pl-k">.</span>setOutputPath(job3, <span class="pl-k">new</span> <span class="pl-smi">Path</span>(otherArgs[otherArgs<span class="pl-k">.</span>length <span class="pl-k">-</span> <span class="pl-c1">1</span>]));
        job3<span class="pl-k">.</span>waitForCompletion(<span class="pl-c1">true</span>);
    }

}
</pre></div>