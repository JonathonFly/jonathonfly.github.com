---
layout: post
category : Spark
tagline: 
tags : [dijkstra]
---
{% include JB/setup %}

之前有一篇写到用Hadoop的MapReduce实现Dijkstra算法，现在，原理不变，将其翻译到Spark环境下（初学，暂时不用GraphX，只用类似MapReduce的操作）。由于原理已经在那一片blog中阐述过，这里不再赘述（PS：输入也是一致的）。简单粗暴，直接上代码。

```java
package com.ideal.netcare;

import java.util.ArrayList;
import java.util.List;
import java.util.regex.Pattern;

import org.apache.commons.lang3.StringUtils;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFlatMapFunction;
import org.apache.spark.api.java.function.PairFunction;

import scala.Tuple2;

public class Dijkstra {
	private static final Pattern SPLIT1 = Pattern.compile("#");
	private static final Pattern SPLIT2 = Pattern.compile(",");
	private static final Pattern SPLIT3 = Pattern.compile("-");
	private static final String startVertex = "A";

	@SuppressWarnings({ "serial", "resource" })
	public static void main(String[] args) {
		if (args.length < 3) {
			System.err.println("Usage: dijkstra <master> <inputFile> <outputFloder>");
			System.exit(1);
		}

		SparkConf sparkConf = new SparkConf().setAppName("dijkstra");
		sparkConf.setMaster(args[0]);
		sparkConf.set("spark.executor.memory", "512m");
		JavaSparkContext ctx = new JavaSparkContext(sparkConf);
		ctx.addJar("spark-test.jar");

		// A#B#10
		JavaRDD<String> lines = ctx.textFile(args[1], 1);

		// A B-10
		JavaPairRDD<String, String> trans1 = lines.mapToPair(new PairFunction<String, String, String>() {
			@Override
			public Tuple2<String, String> call(String s) {
				String[] parts = SPLIT1.split(s);
				return new Tuple2<String, String>(parts[0], parts[1] + "-" + parts[2]);
			}
		}).cache();

		// 计算邻接表
		JavaPairRDD<String, String> adjList = trans1.reduceByKey(new Function2<String, String, String>() {
			@Override
			public String call(String s1, String s2) {
				return s1 + "," + s2;
			}
		}).flatMapToPair(new PairFlatMapFunction<Tuple2<String, String>, String, String>() {
			@Override
			public Iterable<Tuple2<String, String>> call(Tuple2<String, String> s) {
				List<Tuple2<String, String>> results = new ArrayList<Tuple2<String, String>>();
				String res = "";
				String key = s._1();
				String value = s._2();
				if (startVertex.equals(s._1())) {
					res = "0#" + value + "#" + startVertex + "#g";
				} else {
					res = "<>#" + value + "#" + startVertex + "#w";
				}
				results.add(new Tuple2<String, String>(key, res));
				return results;
			}
		});

		JavaPairRDD<String, String> shortestDistance = null;
		JavaPairRDD<Integer, String> nextNodeMap = null;
		JavaPairRDD<String, String> nextNode = null;

		boolean isdone = false;
		int num = 0;
		while (!isdone) {
			// 第一次迭代，将邻接表计算结果作为输入
			if (num == 0)
				nextNode = adjList;

			// 计算最短路径
			shortestDistance = nextNode.flatMapToPair(new PairFlatMapFunction<Tuple2<String, String>, String, String>() {
				@Override
				public Iterable<Tuple2<String, String>> call(Tuple2<String, String> s) {
					List<Tuple2<String, String>> results = new ArrayList<Tuple2<String, String>>();
					// A
					String key = s._1();
					// 到起点的距离 # 邻接表 # 前置节点 # 颜色
					// 0#B-10,C-5#A#g
					String value = s._2();

					String[] parts = SPLIT1.split(value);
					if (parts[3].equals("g")) {
						// B-10 C-5
						String[] arr = SPLIT2.split(parts[1]);
						if (arr.length > 0) {
							for (int i = 0; i < arr.length; i++) {
								// B 10
								String[] edge = SPLIT3.split(arr[i]);
								String id = edge[0];
								String length = "";
								if (parts[0].equals("<>")) {
									length = parts[0];
								} else {
									// 当前节点的距离+邻接边长
									length = (Integer.parseInt(parts[0]) + Integer.parseInt(edge[1])) + "";
								}
								results.add(new Tuple2<String, String>(id, length + "##" + key + "#w"));
							}
						}
						// 将灰色节点置为黑色
						results.add(new Tuple2<String, String>(key, parts[0] + "#" + parts[1] + "#" + parts[2] + "#b"));
					} else {
						results.add(new Tuple2<String, String>(key, parts[0] + "#" + parts[1] + "#" + parts[2] + "#" + parts[3]));
					}
					return results;
				}
			}).reduceByKey(new Function2<String, String, String>() {
				@Override
				public String call(String s1, String s2) {
					String[] parts1 = SPLIT1.split(s1);
					String[] parts2 = SPLIT1.split(s2);
					int length1 = Integer.MAX_VALUE;
					int length2 = Integer.MAX_VALUE;
					if (parts1[3].equals("b"))
						return s1;
					if (parts2[3].equals("b"))
						return s2;
					if (!parts1[0].equals("<>"))
						length1 = Integer.parseInt(parts1[0]);
					if (!parts2[0].equals("<>"))
						length2 = Integer.parseInt(parts2[0]);
					if (length1 < length2) {
						return parts1[0] + "#" + (StringUtils.isNotEmpty(parts1[1]) ? parts1[1] : parts2[1]) + "#" + parts1[2] + "#" + parts1[3];
					} else {
						return parts2[0] + "#" + (StringUtils.isNotEmpty(parts2[1]) ? parts2[1] : parts1[1]) + "#" + parts2[2] + "#" + parts2[3];
					}
				}
			});

			// 排序选出颜色为w的最小距离节点，将其颜色改为g，进行下一轮迭代计算
			nextNodeMap = shortestDistance.flatMapToPair(new PairFlatMapFunction<Tuple2<String, String>, Integer, String>() {
				@Override
				public Iterable<Tuple2<Integer, String>> call(Tuple2<String, String> s) {
					List<Tuple2<Integer, String>> results = new ArrayList<Tuple2<Integer, String>>();
					// A
					String key = s._1();
					// 0#B-10,C-5#A#b
					String value = s._2();

					String[] parts = SPLIT1.split(value);
					int dis = Integer.MAX_VALUE;
					if (!parts[0].equals("<>"))
						dis = Integer.parseInt(parts[0]);
					results.add(new Tuple2<Integer, String>(dis, key + "#" + parts[1] + "#" + parts[2] + "#" + parts[3]));
					return results;
				}
			}).sortByKey(true);

			nextNode = nextNodeMap.flatMapToPair(new PairFlatMapFunction<Tuple2<Integer, String>, String, String>() {
				private boolean hasChangeColor = false;

				@Override
				public Iterable<Tuple2<String, String>> call(Tuple2<Integer, String> s) {
					List<Tuple2<String, String>> results = new ArrayList<Tuple2<String, String>>();
					// 0
					Integer key = s._1();
					// A#B-10,C-5#A#b
					String value = s._2();

					String dis = key.toString();
					if (key.equals(Integer.MAX_VALUE))
						dis = "<>";
					String[] parts = SPLIT1.split(value);
					if (hasChangeColor) {
						// 如果已经找到距离最小的w节点
						results.add(new Tuple2<String, String>(parts[0], dis + "#" + parts[1] + "#" + parts[2] + "#" + parts[3]));
					} else {
						if (parts[3].equals("w")) {
							// 第一个白色节点就是最小的
							results.add(new Tuple2<String, String>(parts[0], dis + "#" + parts[1] + "#" + parts[2] + "#g"));
							hasChangeColor = true;
						} else {
							// 不为白色节点
							results.add(new Tuple2<String, String>(parts[0], dis + "#" + parts[1] + "#" + parts[2] + "#" + parts[3]));
						}
					}

					return results;
				}
			});

			// 校验是否结束

			isdone = true;
			// 看有没有颜色为g的记录
			long count = nextNode.filter(new Function<Tuple2<String, String>, Boolean>() {
				@Override
				public Boolean call(Tuple2<String, String> tp) {
					String value = tp._2();
					String[] parts = SPLIT1.split(value);
					return parts[3].equals("g");
				}
			}).count();

			if (count > 0)
				isdone = false;

			num++;

		}

		// 查找最短路径上的每个点
		// 预处理，将格式变为 b A-A
		JavaPairRDD<String, String> pathPre = nextNode.flatMapToPair(new PairFlatMapFunction<Tuple2<String, String>, String, String>() {
			@Override
			public Iterable<Tuple2<String, String>> call(Tuple2<String, String> s) {
				List<Tuple2<String, String>> results = new ArrayList<Tuple2<String, String>>();
				// A
				String key = s._1();
				// 到起点的距离 # 邻接表 # 前置节点 # 颜色
				// 0#B-10,C-5#A#g
				String value = s._2();

				String[] parts = SPLIT1.split(value);

				if (parts[2].equals(startVertex)) {
					// 前置节点为起始点，则已找到路径上的所有点
					// b A-A
					results.add(new Tuple2<String, String>("b", parts[2] + "-" + key));
				} else {
					results.add(new Tuple2<String, String>("w", parts[2] + "-" + key));
				}
				return results;
			}
		});

		JavaPairRDD<String, String> pathMap = null;
		JavaPairRDD<String, Iterable<String>> pathGroup = null;
		JavaPairRDD<String, String> path = null;

		// 后续迭代查找
		boolean isdone1 = false;
		int num1 = 0;
		while (!isdone1) {

			// 第一次迭代，将邻接表计算结果作为输入
			if (num1 == 0)
				path = pathPre;

			// 将连接点作为key，重新组织键值对
			// key :连接点 value: color#path
			pathMap = path.flatMapToPair(new PairFlatMapFunction<Tuple2<String, String>, String, String>() {
				@Override
				public Iterable<Tuple2<String, String>> call(Tuple2<String, String> s) {
					List<Tuple2<String, String>> results = new ArrayList<Tuple2<String, String>>();
					// b
					String key = s._1();
					// A-A
					String value = s._2();

					if (key.equals("w")) {
						// 如果没有找到起点，则取第一个“-”前的值作为key
						String first = value.substring(0, value.indexOf("-"));
						// C w#C-E
						results.add(new Tuple2<String, String>(first, "w#" + value));
					} else {
						// 如果找到起点，则取最后一个“-”后的值作为key
						String last = value.substring(value.lastIndexOf("-") + 1);
						// A b#A-A
						results.add(new Tuple2<String, String>(last, "b#" + value));
					}
					return results;
				}
			});

			// 将需要组合的路径放在一起 如：C <w#C-B,b#A-C,w#C-E>
			pathGroup = pathMap.groupByKey();

			path = pathGroup.flatMapToPair(new PairFlatMapFunction<Tuple2<String, Iterable<String>>, String, String>() {
				@Override
				public Iterable<Tuple2<String, String>> call(Tuple2<String, Iterable<String>> s) {
					List<Tuple2<String, String>> results = new ArrayList<Tuple2<String, String>>();
					// value: <w#C-B,b#A-C,w#C-E>
					List<String> preList = new ArrayList<String>();
					List<String> afterList = new ArrayList<String>();

					for (String str : s._2()) {
						String[] parts = SPLIT1.split(str);
						if (parts[0].equals("w")) {
							afterList.add(parts[1]);
						}
						if (parts[0].equals("b")) {
							preList.add(parts[1]);
						}
					}

					if (preList != null && preList.size() > 0) {
						for (String n1 : preList) {
							//颜色为b的，原样输出
							results.add(new Tuple2<String, String>("b", n1));
							// 有颜色为b的值，将其路径与颜色为w的路径连接
							for (String n2 : afterList) {
								String res = n1 + n2.substring(n2.indexOf("-"));
								results.add(new Tuple2<String, String>("b", res));
							}
						}
					} else {
						// 只有颜色为w的值
						for (String n2 : afterList) {
							results.add(new Tuple2<String, String>("w", n2));
						}
					}
					return results;
				}
			});

			// 校验是否结束

			isdone1 = true;
			// 看有没有颜色为g的记录
			long count = path.filter(new Function<Tuple2<String, String>, Boolean>() {
				@Override
				public Boolean call(Tuple2<String, String> tp) {
					String key = tp._1();
					return key.equals("w");
				}
			}).count();

			if (count > 0)
				isdone1 = false;

			num1++;

		}
		
		//最短路径值
		nextNode=nextNode.mapToPair(new PairFunction<Tuple2<String,String>, String, String>() {

			@Override
			public Tuple2<String, String> call(Tuple2<String, String> s) throws Exception {
				String key=s._1();
				String value=s._2();
				String res=SPLIT1.split(value)[0];
				return new Tuple2<String, String>(key, res);
			}
		});

		//最短路径上的点
		
		path=path.mapToPair(new PairFunction<Tuple2<String,String>, String, String>() {

			@Override
			public Tuple2<String, String> call(Tuple2<String, String> s) throws Exception {
				String value=s._2();
				String[] parts=SPLIT3.split(value);
				return new Tuple2<String, String>(parts[parts.length-1], value);
			}
		});
		
		//合并结果整理输出
		JavaPairRDD<String,String> result=nextNode.join(path).mapValues(new Function<Tuple2<String,String>, String>() {

			@Override
			public String call(Tuple2<String, String> s) throws Exception {
				return s._1()+","+s._2();
			}
		});
		List<Tuple2<String, String>> output2 = result.sortByKey(true).collect();

		for (Tuple2<String, String> tuple : output2) {
			System.out.println(tuple._1() + ": " + tuple._2());
		}

		// counts.saveAsTextFile(args[2]);

		ctx.stop();
	}

}

```

计算结果：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-03-24/1.png" target="_blank">    
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2016-03-24/1.png" style="max-width:100%;"></a>

It's Done.