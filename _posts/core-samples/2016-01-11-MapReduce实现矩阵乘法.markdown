---
layout: post
category : Hadoop
tagline: 
tags : [hadoop, mapreduce, matrix multiply]
---
{% include JB/setup %}

由于项目需要，前段时间在研究Hadoop下的MapReduce，主要是将普通的算法转化为可在Hadoop集群下运行的MapReduce算法。

MapReduce程序一般是将HDFS文件系统中的文件作为输入和输出，本例的输入形式如下：
![ppt1.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt1.JPG "")

输入文件A矩阵：matrix_A.txt，B矩阵文件matrix_B.txt，在HDFS中的形式如下：

![1.PNG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/1.PNG "")

![2.PNG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/2.PNG "")

要实现如何计算mxn的矩阵与nxp的矩阵相乘，首先，我们先来看一下3x3的矩阵相乘，以便有一个直观的印象。

![ppt2.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt2.JPG "")

左侧矩阵的每一行的元素与右侧矩阵的每一列的对应元素相乘，然后将相乘结果相加，得到结果矩阵每个位置的计算结果。由于MapReduce中的数据只能遍历一次，第二次读的时候就会变为空，所以，首先我们要计算出矩阵中每一个元素需要复制多少份，才能满足A，B各个元素相乘的需求。我们可以看到，A矩阵第1行，第1列的元素A1,1分别与B1,1、B1,2、B1,3三个元素相乘。即A1,1需要复制3份，而该份数正好是B的列数。同理，B中的每个元素需要复制的份数为A的行数。

![ppt3.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt3.JPG "")

那么，知道一个元素需复制的份数后，下一个问题就是：这些元素以何种形式进行进行存储呢？
如A1,1这个元素，需要分别与B1,1、B1,2、B1,3三个元素相乘，那么我们可以用如下形式表示：
(1#1#1 A1,1)(1#2#1 A1,1)(1#3#1 A1,1)，其中一个括号代表一行数据，A1,1代表矩阵第一行第一列的值。1#2#1中第一个数字1代表结果矩阵第1行，即A1,1的行号；第二个数字2代表结果矩阵的第2列，即B1,2的列号；第3个数字1代表A1,1的列号和B1,2的行号。

![ppt4.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt4.JPG "")

这样的存储形式的优势是什么？同一个key，如1#2#1，只会有2个元素，即一个A1,1和一个B1,2。可以直接将这两个元素相乘。

![ppt5.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt5.JPG "")

同理，可以算出B的元素复制和元素表示。如B1,2可以表示为：(1#2#1 B1,2)(2#2#1 B1,2)(3#2#1 B1,2)

![ppt6.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt6.JPG "")

通用公式如下：

![ppt7.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt7.JPG "")

在reduce的时候，相同的key的值在同一个节点计算，可以直接相乘。如key为1#2#1，结果为A1,1 X B1,2

![ppt8.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt8.JPG "")

通用的在第一个reduce阶段进行乘法计算。

![ppt9.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt9.JPG "")

第二个阶段进行加法计算。

![ppt10.JPG](E:/MyRepository/jonathonfly.github.com/_posts/core-samples/pictures/2016-01-11/ppt10.JPG "")

总的代码如下：

MatrixMultiply.java

```java
package com.ideal;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.log4j.Logger;

public class MatrixMultiply {
	private static Logger logger = Logger.getLogger(MatrixMultiply.class);
	private static Integer ARowNum = 0;
	private static Integer AColNum = 0;
	private static Integer BRowNum = 0;
	private static Integer BColNum = 0;

	public static class MatrixMultiplyPreMapper extends Mapper<Object, Text, Text, Text> {

		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			Configuration conf = context.getConfiguration();
			MatrixMultiply.ARowNum = conf.getInt("ARowNum", 0);
			MatrixMultiply.AColNum = conf.getInt("AColNum", 0);
			MatrixMultiply.BRowNum = conf.getInt("BRowNum", 0);
			MatrixMultiply.BColNum = conf.getInt("BColNum", 0);
			if (MatrixMultiply.BRowNum != MatrixMultiply.AColNum)
				return;
			String pathName = ((FileSplit) context.getInputSplit()).getPath().toString();
			logger.info(pathName);
			// get matrix value
			if (value.toString() == null || value.toString().equals(""))
				return;
			String[] rowVal = value.toString().split("#|\\s+");
			if (rowVal.length < 3)
				return;

			if (pathName.contains("matrix_A")) {
				int m = Integer.parseInt(rowVal[0]);
				int n = Integer.parseInt(rowVal[1]);
				int numVal = Integer.parseInt(rowVal[2]);
				for (int i = 1; i <= MatrixMultiply.BColNum; i++) {
					mapKey.set(m + "#" + i + "#" + n);
					mapValue.set(numVal+"");
					context.write(mapKey, mapValue);
				}
			} else if (pathName.contains("matrix_B")) {
				int n = Integer.parseInt(rowVal[0]);
				int k = Integer.parseInt(rowVal[1]);
				int numVal = Integer.parseInt(rowVal[2]);

				for (int i = 1; i <= MatrixMultiply.ARowNum; i++) {
					mapKey.set(i + "#" + k + "#" + n);
					mapValue.set(numVal+"");
					context.write(mapKey, mapValue);
				}
			}
		}
	}

	public static class MatrixMultiplyPreReducer extends Reducer<Text, Text, Text, Text> {
		private Text resKey = new Text();
		private Text result = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			int res=1;
			for (Text val : values) {
				if (val.toString() == null || val.toString().equals(""))
					return;
				res*= Integer.parseInt(val.toString());
			}
			result.set(res + "");
			
			String newKey=key.toString().substring(0,key.toString().lastIndexOf("#"));
			resKey.set(newKey);
			context.write(resKey, result);
		}
	}
	
	public static class MatrixMultiplyMapper extends Mapper<Object, Text, Text, Text> {

		private Text mapKey = new Text();
		private Text mapValue = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			if (value.toString() == null || value.toString().equals(""))
				return;
			String[] rowVal = value.toString().split("\\s+");
			if (rowVal.length < 2)
				return;
			mapKey.set(rowVal[0]);
			mapValue.set(rowVal[1]);
			context.write(mapKey, mapValue);
		}
	}

	public static class MatrixMultiplyReducer extends Reducer<Text, Text, Text, Text> {
		private Text result = new Text();

		public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
			int sum=0;
			for (Text val : values) {
				sum+=Integer.valueOf(val.toString());
			}
			result.set(sum+"");
			context.write(key, result);
		}
	}

	public static void main(String[] args) throws Exception {
		
	}

}

```

将计算结果排个序：

MatrixSort.java

```java
package com.ideal;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.log4j.Logger;

public class MatrixSort {
	private static Logger logger = Logger.getLogger(MatrixSort.class);

	public static class IntPair implements WritableComparable<IntPair>
    {
        int first;
        int second;

        public void set(int left, int right)
        {
            first = left;
            second = right;
        }
        public int getFirst()
        {
            return first;
        }
        public int getSecond()
        {
            return second;
        }
        @Override
        
        public void readFields(DataInput in) throws IOException
        {
            // TODO Auto-generated method stub
            first = in.readInt();
            second = in.readInt();
        }
        @Override
        
        public void write(DataOutput out) throws IOException
        {
            // TODO Auto-generated method stub
            out.writeInt(first);
            out.writeInt(second);
        }
        @Override
        
        public int compareTo(IntPair o)
        {
            // TODO Auto-generated method stub
            if (first != o.first)
            {
                return first < o.first ? -1 : 1;
            }
            else if (second != o.second)
            {
                return second < o.second ? -1 : 1;
            }
            else
            {
                return 0;
            }
        }

        @Override
        //The hashCode() method is used by the HashPartitioner (the default partitioner in MapReduce)
        public int hashCode()
        {
            return first * 157 + second;
        }
        @Override
        public boolean equals(Object right)
        {
            if (right == null)
                return false;
            if (this == right)
                return true;
            if (right instanceof IntPair)
            {
                IntPair r = (IntPair) right;
                return r.first == first && r.second == second;
            }
            else
            {
                return false;
            }
        }
    }
    
    public static class FirstPartitioner extends Partitioner<IntPair, IntWritable>
    {
        @Override
        public int getPartition(IntPair key, IntWritable value,int numPartitions)
        {
            return Math.abs(key.getFirst() * 127) % numPartitions;
        }
    }

    
    public static class GroupingComparator extends WritableComparator
    {
        protected GroupingComparator()
        {
            super(IntPair.class, true);
        }
        @SuppressWarnings("rawtypes")
		@Override
        //Compare two WritableComparables.
        public int compare(WritableComparable w1, WritableComparable w2)
        {
            IntPair ip1 = (IntPair) w1;
            IntPair ip2 = (IntPair) w2;
            int l = ip1.getSecond();
            int r = ip2.getSecond();
            return l == r ? 0 : (l < r ? -1 : 1);
        }
    }


    
    public static class MatrixSortMapper extends Mapper<Object, Text, IntPair, IntWritable>
    {
        private IntPair intkey = new IntPair();
        private IntWritable intvalue = new IntWritable();
        public void map(Object key, Text value, Context context) throws IOException, InterruptedException
        {
        	logger.info("map start");
        	if (value.toString() == null || value.toString().equals(""))
				return;
			String[] rowVal = value.toString().split("#|\\s+");
			if (rowVal.length < 3)
				return;
			
			int left = Integer.parseInt(rowVal[0]);
			int right = Integer.parseInt(rowVal[1]);
			int num=Integer.parseInt(rowVal[2]);
            intkey.set(left, right);
			intvalue.set(num);
			logger.info(left+" "+right+" "+num);
            context.write(intkey, intvalue);
        }
    }
    
    //
    public static class MatrixSortReducer extends Reducer<IntPair, IntWritable, Text, IntWritable>
    {
        private Text left = new Text();
        
        public void reduce(IntPair key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException
        {
            left.set(key.getFirst()+"#"+key.getSecond());
            for (IntWritable val : values)
            {
                context.write(left, val);
                logger.info(left+" "+val);
            }
        }
    }

	public static void main(String[] args) throws Exception {
	
	}

}

```

总的调用main方法：

MatrixMultiplyAndSort.java

```java
package com.ideal;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.ideal.MatrixMultiply.MatrixMultiplyMapper;
import com.ideal.MatrixMultiply.MatrixMultiplyPreMapper;
import com.ideal.MatrixMultiply.MatrixMultiplyPreReducer;
import com.ideal.MatrixMultiply.MatrixMultiplyReducer;
import com.ideal.MatrixSort.FirstPartitioner;
import com.ideal.MatrixSort.GroupingComparator;
import com.ideal.MatrixSort.IntPair;
import com.ideal.MatrixSort.MatrixSortMapper;
import com.ideal.MatrixSort.MatrixSortReducer;

public class MatrixMultiplyAndSort {
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		conf.set("mapred.jar", "MatrixMultiply.jar");
		
		conf.setInt("ARowNum", 3);
		conf.setInt("AColNum", 3);
		conf.setInt("BRowNum", 3);
		conf.setInt("BColNum", 3);

		String[] ars=new String[]{"/syf/matrix_test/input/input1","/syf/matrix_test/output/output_multiply_1"};  
		String[] otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("usage: matrix multiply1 <in> [<in>...] <out> ");
			System.exit(2);
		}
		Job job1 = Job.getInstance(conf, "matrix multiply1");
		job1.setJarByClass(MatrixMultiply.class);
		job1.setMapperClass(MatrixMultiplyPreMapper.class);
		job1.setReducerClass(MatrixMultiplyPreReducer.class);
		job1.setOutputKeyClass(Text.class);
		job1.setOutputValueClass(Text.class);

		for (int i = 0; i < otherArgs.length - 1; ++i) {
			FileInputFormat.addInputPath(job1, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job1, new Path(otherArgs[otherArgs.length - 1]));
		job1.waitForCompletion(true);

		//
		ars=new String[]{"/syf/matrix_test/output/output_multiply_1","/syf/matrix_test/output/output_multiply_2"}; 
		otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("usage: matrix multiply2 <in> [<in>...] <out> ");
			System.exit(2);
		}
		Job job2 = Job.getInstance(conf, "matrix multiply2");
		job2.setJarByClass(MatrixMultiply.class);
		job2.setMapperClass(MatrixMultiplyMapper.class);
		job2.setReducerClass(MatrixMultiplyReducer.class);
		
        job2.setOutputKeyClass(Text.class);
        job2.setOutputValueClass(Text.class);
        for (int i = 0; i < otherArgs.length - 1; ++i) {
			FileInputFormat.addInputPath(job2, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job2, new Path(otherArgs[otherArgs.length - 1]));
		job2.waitForCompletion(true);
		
		//
		ars=new String[]{"/syf/matrix_test/output/output_multiply_2","/syf/matrix_test/output/output_sort"}; 
		otherArgs = new GenericOptionsParser(conf, ars).getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("usage: matrix sort <in> [<in>...] <out> ");
			System.exit(2);
		}
		Job job3 = Job.getInstance(conf, "matrix sort");
		job3.setJarByClass(MatrixSort.class);
		job3.setMapperClass(MatrixSortMapper.class);
		job3.setReducerClass(MatrixSortReducer.class);
        job3.setPartitionerClass(FirstPartitioner.class);
        job3.setGroupingComparatorClass(GroupingComparator.class);
		
        job3.setMapOutputKeyClass(IntPair.class);
        job3.setMapOutputValueClass(IntWritable.class);
        job3.setOutputKeyClass(Text.class);
        job3.setOutputValueClass(IntWritable.class);
        for (int i = 0; i < otherArgs.length - 1; ++i) {
			FileInputFormat.addInputPath(job3, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job3, new Path(otherArgs[otherArgs.length - 1]));
		job3.waitForCompletion(true);
	}

}

```