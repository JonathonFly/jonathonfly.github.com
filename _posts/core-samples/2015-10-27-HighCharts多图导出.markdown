---
layout: post
category : Java
tagline: 
tags : []
---
{% include JB/setup %}

因为自己在维护的项目的新需求，需要用到HighCharts做统计图，并导出为PDF文件，所以稍微研究了一下。

单个统计图HighCharts有导出功能，并可以选择格式为PDF、CVS、图片等。因为需求中只有要求导出为PDF，所以需要将其他功能隐藏一下。高版本的exporting.js默认是将这些功能共同放在一个下拉列表中，我想要的是一个button，点击可以直接导出为PDF。于是我选择了Highcharts JS v2.2.5 版本，这个版本支持一个图标按钮，可直接导出PDF。

另一个需求是天、周、月、年各个统计图不仅需要支持单个导出，还要支持一起导出。一开始没有在HighCharts教程中看到相关的介绍，经过查找，找到了解决方法。

另外，HighCharts支持多语言搭建导出服务器，将导出服务器的war包在web容器中跑一下，将下载地址修改一下就行。具体可查看HighCharts中文网，教程还是比较全的。

导出多张统计图的exportSeveralCharts.js

```javascript
Highcharts.getSVG = function(charts) {
			var svgArr = [], top = 0, width = 0;

			$.each(charts, function(i, chart) {
				var svg = chart.getSVG();
				svg = svg.replace('<svg', '<g transform="translate(0,' + top
						+ ')" ');
				svg = svg.replace('</svg>', '</g>');

				top += chart.chartHeight;
				width = Math.max(width, chart.chartWidth);

				svgArr.push(svg);
			});

			return '<svg height="'+ top +'" width="' + width + '" version="1.1" xmlns="http://www.w3.org/2000/svg">'
					+ svgArr.join('') + '</svg>';
		};

		/**
		 * Create a global exportCharts method that takes an array of charts as an argument,
		 * and exporting options as the second argument
		 */
		Highcharts.exportCharts = function(charts, options, url, type, filename) {
			var form
			svg = Highcharts.getSVG(charts);

			// merge the options
			options = Highcharts.merge(Highcharts.getOptions().exporting,
					options);
			options.url = url;
			options.type = type;
			options.filename = filename;
			// create the form
			form = Highcharts.createElement('form', {
				method : 'post',
				action : options.url
			}, {
				display : 'none'
			}, document.body);

			// add the values
			Highcharts.each([ 'filename', 'type', 'width', 'svg' ], function(
					name) {
				Highcharts.createElement('input', {
					type : 'hidden',
					name : name,
					value : {
						filename : options.filename || 'chart',
						type : options.type,
						width : options.width,
						svg : svg
					}[name]
				}, null, form);
			});
			form.submit();

			// clean up
			form.parentNode.removeChild(form);
		};
```

具体调用的js代码如下：

```javascript
var export_server_url="http://localhost:9090/highcharts-export";
    $('#export').click(
    function() {
        //导出服务器地址
        var url = export_server_url;
        //导出格式
        var type = "application/pdf";
        //导出的文件名
        var filename = "WAN Statistics";
        Highcharts.exportCharts([ chart1, chart2 ], null, url,
                type, filename);
    });
```

统计图页面详细代码如下：

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ include file="/include/taglibs.jsp"%>
<html>

<script src="<c:url value='/js/jquery/jquery-1.8.2.min.js'/>"
	type="text/javascript"></script>
<script type="text/javascript"
	src="<c:url value='/js/My97DatePicker/WdatePicker.js'/>"></script>
<script type="text/javascript"
	src="<c:url value='/js/highcharts/highcharts.js'/>"></script>
<script src="<c:url value='/js/highcharts/exporting.js'/>"
	type="text/javascript"></script>
<script src="<c:url value='./js/exportSeveralCharts.js'/>"
	type="text/javascript"></script>

<body>
	<div id="container1" style="height: 400px"></div>
	<div id="container2" style="height: 400px"></div>

	<button id="export">Export all</button>

	<script type="text/javascript">
	//var  export_server_url="http://export.highcharts.com";
	var export_server_url="http://localhost:9090/highcharts-export";
	
		var chart1 = new Highcharts.Chart({

			chart : {
				renderTo : 'container1',
				type : 'spline', //曲线图
				//type : 'line',   //折线图
				backgroundColor : '#FBF8D7',
				zoomType : 'xy',
				reflow : true
			},
			title : {
				text : 'Test chart1' //指定图表标题
			},
			xAxis : {
				type : 'datetime',
				lineColor : '#000000',
				lineWidth : 2,
				tickWidth : 0,
				tickColor : '#000000',
				startOnTick : true,
				maxPadding : 0.1,
				gridLineColor : "#E0E0E0",
				gridLineDashStyle : "solid",
				gridLineWidth : 1,
				tickInterval : 24 * 3600 * 1000,//X轴点间隔
				labels : {
					rotation : -55,
					y : 40,
					x : -20,
					formatter : function() {
						return Highcharts.dateFormat('%Y-%m-%d', this.value);
					}
				}
			},
			yAxis : {
				min : 0,
				startOnTick : true,
				lineColor : '#000000',
				lineWidth : 2,
				tickWidth : 0,
				tickColor : '#000000',
				max : 1,
				title : {
					text : 'WAN Traffic (M)' //指定y轴的标题
				},
				labels : {
					style : {
						color : '#000',
						font : '11px Trebuchet MS, Verdana, sans-serif'
					}
				},
				plotLines : [ {
					id : 'ymax',
					color : 'red', //线的颜色，定义为红色
					dashStyle : 'solid', //默认值，这里定义为实线
					value : 5, //表示线所在的值
					width : 3, //标示线的宽度
					label : {
						text : 'WAN', //标签的内容
						align : 'right', //标签的水平位置，水平居右,默认是水平居中center
						x : -10,
						y : -10
					}
				} ]
			},
			tooltip : {
				formatter : function() {
					return this.series.name
							+ '<br/><strong>Date:</strong>'
							+ Highcharts
									.dateFormat('%Y-%m-%d %H:%M:%S', this.x)
							+ '<br/><strong>value:</strong> ' + this.y;
				}
			},
			plotOptions : {
				series : {
					stickyTracking : false
				},
				turboThreshold : 0
			//不限制数据点个数
			},
			series : [],
			//导出按钮的tip
			lang : {
				exportButtonTitle : "Export to PDF"
			},
			//统计图导出为PDF
			exporting : {
				//url:"http://export.highcharts.com",
				url : export_server_url,
				buttons : {
					exportButton : {
						menuItems : null,
						onclick : function() {
							this.exportChart({
								type : 'application/pdf',
								filename : 'Day Chart of WAN Port Performance'
							}, null);
						}
					},
					printButton : {
						enabled : false
					}
				}
			}
		});

		var chart2 = new Highcharts.Chart({

			chart : {
				renderTo : 'container2',
				type : 'column',
				backgroundColor : '#FBF8D7',
				zoomType : 'xy',
				reflow : true
			},
			title : {
				text : 'Test chart2' //指定图表标题
			},
			xAxis : {
				lineColor : '#000000',
				lineWidth : 2,
				tickWidth : 0,
				tickColor : '#000000',
				startOnTick : true,
				maxPadding : 0.1,
				gridLineColor : "#E0E0E0",
				gridLineDashStyle : "solid",
				gridLineWidth : 1,
				categories : [ 'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul',
						'Aug', 'Sep', 'Oct', 'Nov', 'Dec' ]
			},
			yAxis : {
				min : 0,
				startOnTick : true,
				lineColor : '#000000',
				lineWidth : 2,
				tickWidth : 0,
				tickColor : '#000000',
				title : {
					text : 'WAN Traffic (M)' //指定y轴的标题
				},
			},
			plotOptions : {
				series : {
					stickyTracking : false
				},
				turboThreshold : 0
			//不限制数据点个数
			},
			series : [ {
				data : [ 176.0, 135.6, 148.5, 216.4, 194.1, 95.6, 54.4, 29.9,
						71.5, 106.4, 129.2, 144.0 ]
			} ],
			//统计图导出为PDF
			exporting : {
				//url:"http://export.highcharts.com",
				url : export_server_url,
				buttons : {
					exportButton : {
						menuItems : null,
						onclick : function() {
							this.exportChart({
								type : 'application/pdf',
								filename : 'aaa'
							}, null);
						}
					},
					printButton : {
						enabled : false
					}
				}
			}
		});

		$('#export').click(
				function() {
					//导出服务器地址
					
					var url = export_server_url;
					//导出格式
					var type = "application/pdf";
					//导出的文件名
					var filename = "WAN Statistics";
					Highcharts.exportCharts([ chart1, chart2 ], null, url,
							type, filename);
				});

		// 与后台进行数据交互  
		function getForm(neId) {
			$.ajax({
				type : "POST",
				url : "<c:url value='/buz/wanStatus.do'/>",
				data : {
					"method" : "getDayStatusChartsOptions",
					"neId" : neId
				},
				success : function(data) {
					var categoriesResult = data.categories;
					var chartTitleResult = data.chartTitle;
					var timeGranularityResult = data.timeGranularity;
					var seriesResult = data.series;
					//wan端口带宽值
					var wanValue = 0.5;

					//先清除series的值
					var series1 = chart1.series;
					while (series1.length > 0) {
						series1[0].remove(false);
					}
					var color = [];
					color.push('#0000ff');
					color.push('#00dd00');
					//添加series的值
					if (seriesResult != null) {
						for (var i = 0; seriesResult.length > 0
								&& i < seriesResult.length; i++) {
							var sdata = seriesResult[i].data;

							chart1.addSeries({
								name : seriesResult[i].name,
								color : color[i],
								lineWidth : 2,
								data : sdata
							}, false);
						}
					}

					//重新画WAN带宽标示线，先删除
					chart1.yAxis[0].removePlotLine('ymax');
					var plotLine = {
						id : 'ymax',
						color : 'red', //线的颜色，定义为红色
						dashStyle : 'solid', //默认值，这里定义为实线
						value : wanValue, //表示线所在的值
						width : 3, //标示线的宽度
						label : {
							text : 'WAN', //标签的内容
							style : {
								color : '#ff0000',
								fontWeight : 'bold'
							},
							align : 'right', //标签的水平位置，水平居右,默认是水平居中center
							x : -10,
							y : -10
						}
					};
					chart1.yAxis[0].addPlotLine(plotLine);

					//重新设y轴的最大最小值，将wan端口的值显示在图中
					chart1.yAxis[0].setExtremes(0, wanValue + wanValue / 20,
							true);

					//设置标题
					var title = {
						text : chartTitleResult,
						style : {
							fontWeight : "bold"
						}
					};
					chart1.setTitle(title);
					chart1.redraw();
				}
			});
		}

		//定时刷新 列表数据  
		$(function() {
			//每隔x秒自动调用方法，实现图表的实时更新   
			window.setInterval(getForm(158238), 30000);
			setInterval(function() {
				getForm(158238);
			}, 20000);
		});
	</script>
</body>
</html>
```

结果图如下：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-27/1.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-27/1.png" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-27/2.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-27/2.png" style="max-width:100%;"></a>