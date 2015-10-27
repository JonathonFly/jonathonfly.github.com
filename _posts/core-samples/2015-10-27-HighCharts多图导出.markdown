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

<div class="highlight highlight-source-js"><pre><span class="pl-c1">Highcharts</span>.<span class="pl-en">getSVG</span> <span class="pl-k">=</span> <span class="pl-k">function</span>(<span class="pl-smi">charts</span>) {
            <span class="pl-k">var</span> svgArr <span class="pl-k">=</span> [], top <span class="pl-k">=</span> <span class="pl-c1">0</span>, width <span class="pl-k">=</span> <span class="pl-c1">0</span>;

            $.each(charts, <span class="pl-k">function</span>(<span class="pl-smi">i</span>, <span class="pl-smi">chart</span>) {
                <span class="pl-k">var</span> svg <span class="pl-k">=</span> chart.getSVG();
                svg <span class="pl-k">=</span> svg.<span class="pl-c1">replace</span>(<span class="pl-s"><span class="pl-pds">'</span>&lt;svg<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>&lt;g transform="translate(0,<span class="pl-pds">'</span></span> <span class="pl-k">+</span> top
                        <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">'</span>)" <span class="pl-pds">'</span></span>);
                svg <span class="pl-k">=</span> svg.<span class="pl-c1">replace</span>(<span class="pl-s"><span class="pl-pds">'</span>&lt;/svg&gt;<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>&lt;/g&gt;<span class="pl-pds">'</span></span>);

                top <span class="pl-k">+=</span> chart.chartHeight;
                width <span class="pl-k">=</span> <span class="pl-c1">Math</span>.<span class="pl-c1">max</span>(width, chart.chartWidth);

                svgArr.<span class="pl-c1">push</span>(svg);
            });

            <span class="pl-k">return</span> <span class="pl-s"><span class="pl-pds">'</span>&lt;svg height="<span class="pl-pds">'</span></span><span class="pl-k">+</span> top <span class="pl-k">+</span><span class="pl-s"><span class="pl-pds">'</span>" width="<span class="pl-pds">'</span></span> <span class="pl-k">+</span> width <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">'</span>" version="1.1" xmlns="http://www.w3.org/2000/svg"&gt;<span class="pl-pds">'</span></span>
                    <span class="pl-k">+</span> svgArr.<span class="pl-c1">join</span>(<span class="pl-s"><span class="pl-pds">'</span><span class="pl-pds">'</span></span>) <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">'</span>&lt;/svg&gt;<span class="pl-pds">'</span></span>;
        };

        <span class="pl-c">/**</span>
<span class="pl-c">         * Create a global exportCharts method that takes an array of charts as an argument,</span>
<span class="pl-c">         * and exporting options as the second argument</span>
<span class="pl-c">         */</span>
        <span class="pl-c1">Highcharts</span>.<span class="pl-en">exportCharts</span> <span class="pl-k">=</span> <span class="pl-k">function</span>(<span class="pl-smi">charts</span>, <span class="pl-smi">options</span>, <span class="pl-smi">url</span>, <span class="pl-smi">type</span>, <span class="pl-smi">filename</span>) {
            <span class="pl-k">var</span> form
            svg <span class="pl-k">=</span> Highcharts.getSVG(charts);

            <span class="pl-c">// merge the options</span>
            options <span class="pl-k">=</span> Highcharts.merge(Highcharts.getOptions().exporting,
                    options);
            options.url <span class="pl-k">=</span> url;
            options.<span class="pl-c1">type</span> <span class="pl-k">=</span> type;
            options.<span class="pl-c1">filename</span> <span class="pl-k">=</span> filename;
            <span class="pl-c">// create the form</span>
            form <span class="pl-k">=</span> Highcharts.<span class="pl-c1">createElement</span>(<span class="pl-s"><span class="pl-pds">'</span>form<span class="pl-pds">'</span></span>, {
                method <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>post<span class="pl-pds">'</span></span>,
                action <span class="pl-k">:</span> options.url
            }, {
                display <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>none<span class="pl-pds">'</span></span>
            }, <span class="pl-c1">document</span>.<span class="pl-c1">body</span>);

            <span class="pl-c">// add the values</span>
            Highcharts.each([ <span class="pl-s"><span class="pl-pds">'</span>filename<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>type<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>width<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>svg<span class="pl-pds">'</span></span> ], <span class="pl-k">function</span>(
                    <span class="pl-smi">name</span>) {
                Highcharts.<span class="pl-c1">createElement</span>(<span class="pl-s"><span class="pl-pds">'</span>input<span class="pl-pds">'</span></span>, {
                    type <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>hidden<span class="pl-pds">'</span></span>,
                    name <span class="pl-k">:</span> name,
                    value <span class="pl-k">:</span> {
                        filename <span class="pl-k">:</span> options.<span class="pl-c1">filename</span> <span class="pl-k">||</span> <span class="pl-s"><span class="pl-pds">'</span>chart<span class="pl-pds">'</span></span>,
                        type <span class="pl-k">:</span> options.<span class="pl-c1">type</span>,
                        width <span class="pl-k">:</span> options.<span class="pl-c1">width</span>,
                        svg <span class="pl-k">:</span> svg
                    }[name]
                }, <span class="pl-c1">null</span>, form);
            });
            form.<span class="pl-c1">submit</span>();

            <span class="pl-c">// clean up</span>
            form.<span class="pl-c1">parentNode</span>.removeChild(form);
        };</pre></div>

具体调用的js代码如下：

<div class="highlight highlight-source-js"><pre><span class="pl-k">var</span> export_server_url<span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">"</span>http://localhost:9090/highcharts-export<span class="pl-pds">"</span></span>;
    $(<span class="pl-s"><span class="pl-pds">'</span>#export<span class="pl-pds">'</span></span>).<span class="pl-c1">click</span>(
    <span class="pl-k">function</span>() {
        <span class="pl-c">//导出服务器地址</span>
        <span class="pl-k">var</span> url <span class="pl-k">=</span> export_server_url;
        <span class="pl-c">//导出格式</span>
        <span class="pl-k">var</span> type <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>application/pdf<span class="pl-pds">"</span></span>;
        <span class="pl-c">//导出的文件名</span>
        <span class="pl-k">var</span> filename <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>WAN Statistics<span class="pl-pds">"</span></span>;
        Highcharts.exportCharts([ chart1, chart2 ], <span class="pl-c1">null</span>, url,
                type, filename);
    });</pre></div>

统计图页面详细代码如下：

<div class="highlight highlight-text-html-basic"><pre>&lt;%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%&gt;
&lt;%@ include file="/include/taglibs.jsp"%&gt;
&lt;<span class="pl-ent">html</span>&gt;

&lt;<span class="pl-ent">script</span> <span class="pl-e">src</span>=<span class="pl-s"><span class="pl-pds">"</span>&lt;c:url value='/js/jquery/jquery-1.8.2.min.js'/&gt;<span class="pl-pds">"</span></span>
    <span class="pl-e">type</span>=<span class="pl-s"><span class="pl-pds">"</span>text/javascript<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">script</span>&gt;
<span class="pl-s1">&lt;<span class="pl-ent">script</span> <span class="pl-e">type</span>=<span class="pl-s"><span class="pl-pds">"</span>text/javascript<span class="pl-pds">"</span></span></span>
<span class="pl-s1">    <span class="pl-e">src</span>=<span class="pl-s"><span class="pl-pds">"</span>&lt;c:url value='/js/My97DatePicker/WdatePicker.js'/&gt;<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">script</span>&gt;</span>
<span class="pl-s1">&lt;<span class="pl-ent">script</span> <span class="pl-e">type</span>=<span class="pl-s"><span class="pl-pds">"</span>text/javascript<span class="pl-pds">"</span></span></span>
<span class="pl-s1">    <span class="pl-e">src</span>=<span class="pl-s"><span class="pl-pds">"</span>&lt;c:url value='/js/highcharts/highcharts.js'/&gt;<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">script</span>&gt;</span>
&lt;<span class="pl-ent">script</span> <span class="pl-e">src</span>=<span class="pl-s"><span class="pl-pds">"</span>&lt;c:url value='/js/highcharts/exporting.js'/&gt;<span class="pl-pds">"</span></span>
    <span class="pl-e">type</span>=<span class="pl-s"><span class="pl-pds">"</span>text/javascript<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">script</span>&gt;
&lt;<span class="pl-ent">script</span> <span class="pl-e">src</span>=<span class="pl-s"><span class="pl-pds">"</span>&lt;c:url value='./js/exportSeveralCharts.js'/&gt;<span class="pl-pds">"</span></span>
    <span class="pl-e">type</span>=<span class="pl-s"><span class="pl-pds">"</span>text/javascript<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">script</span>&gt;

&lt;<span class="pl-ent">body</span>&gt;
    &lt;<span class="pl-ent">div</span> <span class="pl-e">id</span>=<span class="pl-s"><span class="pl-pds">"</span>container1<span class="pl-pds">"</span></span> <span class="pl-e">style</span>=<span class="pl-s"><span class="pl-pds">"</span>height: 400px<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
    &lt;<span class="pl-ent">div</span> <span class="pl-e">id</span>=<span class="pl-s"><span class="pl-pds">"</span>container2<span class="pl-pds">"</span></span> <span class="pl-e">style</span>=<span class="pl-s"><span class="pl-pds">"</span>height: 400px<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;

    &lt;<span class="pl-ent">button</span> <span class="pl-e">id</span>=<span class="pl-s"><span class="pl-pds">"</span>export<span class="pl-pds">"</span></span>&gt;Export all&lt;/<span class="pl-ent">button</span>&gt;

<span class="pl-s1">    &lt;<span class="pl-ent">script</span> <span class="pl-e">type</span>=<span class="pl-s"><span class="pl-pds">"</span>text/javascript<span class="pl-pds">"</span></span>&gt;</span>
<span class="pl-s1">    <span class="pl-c">//var  export_server_url="http://export.highcharts.com";</span></span>
<span class="pl-s1">    <span class="pl-k">var</span> export_server_url<span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">"</span>http://localhost:9090/highcharts-export<span class="pl-pds">"</span></span>;</span>
<span class="pl-s1"></span>
<span class="pl-s1">        <span class="pl-k">var</span> chart1 <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-en">Highcharts.Chart</span>({</span>
<span class="pl-s1"></span>
<span class="pl-s1">            chart <span class="pl-k">:</span> {</span>
<span class="pl-s1">                renderTo <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>container1<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                type <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>spline<span class="pl-pds">'</span></span>, <span class="pl-c">//曲线图</span></span>
<span class="pl-s1">                <span class="pl-c">//type : 'line',   //折线图</span></span>
<span class="pl-s1">                backgroundColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#FBF8D7<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                zoomType <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>xy<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                reflow <span class="pl-k">:</span> <span class="pl-c1">true</span></span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            title <span class="pl-k">:</span> {</span>
<span class="pl-s1">                text <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>Test chart1<span class="pl-pds">'</span></span> <span class="pl-c">//指定图表标题</span></span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            xAxis <span class="pl-k">:</span> {</span>
<span class="pl-s1">                type <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>datetime<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                lineColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                lineWidth <span class="pl-k">:</span> <span class="pl-c1">2</span>,</span>
<span class="pl-s1">                tickWidth <span class="pl-k">:</span> <span class="pl-c1">0</span>,</span>
<span class="pl-s1">                tickColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                startOnTick <span class="pl-k">:</span> <span class="pl-c1">true</span>,</span>
<span class="pl-s1">                maxPadding <span class="pl-k">:</span> <span class="pl-c1">0.1</span>,</span>
<span class="pl-s1">                gridLineColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>#E0E0E0<span class="pl-pds">"</span></span>,</span>
<span class="pl-s1">                gridLineDashStyle <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>solid<span class="pl-pds">"</span></span>,</span>
<span class="pl-s1">                gridLineWidth <span class="pl-k">:</span> <span class="pl-c1">1</span>,</span>
<span class="pl-s1">                tickInterval <span class="pl-k">:</span> <span class="pl-c1">24</span> <span class="pl-k">*</span> <span class="pl-c1">3600</span> <span class="pl-k">*</span> <span class="pl-c1">1000</span>,<span class="pl-c">//X轴点间隔</span></span>
<span class="pl-s1">                labels <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    rotation <span class="pl-k">:</span> <span class="pl-k">-</span><span class="pl-c1">55</span>,</span>
<span class="pl-s1">                    y <span class="pl-k">:</span> <span class="pl-c1">40</span>,</span>
<span class="pl-s1">                    x <span class="pl-k">:</span> <span class="pl-k">-</span><span class="pl-c1">20</span>,</span>
<span class="pl-s1">                    <span class="pl-en">formatter</span> <span class="pl-k">:</span> <span class="pl-k">function</span>() {</span>
<span class="pl-s1">                        <span class="pl-k">return</span> Highcharts.dateFormat(<span class="pl-s"><span class="pl-pds">'</span>%Y-%m-%d<span class="pl-pds">'</span></span>, <span class="pl-v">this</span>.<span class="pl-c1">value</span>);</span>
<span class="pl-s1">                    }</span>
<span class="pl-s1">                }</span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            yAxis <span class="pl-k">:</span> {</span>
<span class="pl-s1">                min <span class="pl-k">:</span> <span class="pl-c1">0</span>,</span>
<span class="pl-s1">                startOnTick <span class="pl-k">:</span> <span class="pl-c1">true</span>,</span>
<span class="pl-s1">                lineColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                lineWidth <span class="pl-k">:</span> <span class="pl-c1">2</span>,</span>
<span class="pl-s1">                tickWidth <span class="pl-k">:</span> <span class="pl-c1">0</span>,</span>
<span class="pl-s1">                tickColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                max <span class="pl-k">:</span> <span class="pl-c1">1</span>,</span>
<span class="pl-s1">                title <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    text <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>WAN Traffic (M)<span class="pl-pds">'</span></span> <span class="pl-c">//指定y轴的标题</span></span>
<span class="pl-s1">                },</span>
<span class="pl-s1">                labels <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    style <span class="pl-k">:</span> {</span>
<span class="pl-s1">                        color <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                        font <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>11px Trebuchet MS, Verdana, sans-serif<span class="pl-pds">'</span></span></span>
<span class="pl-s1">                    }</span>
<span class="pl-s1">                },</span>
<span class="pl-s1">                plotLines <span class="pl-k">:</span> [ {</span>
<span class="pl-s1">                    id <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>ymax<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                    color <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>red<span class="pl-pds">'</span></span>, <span class="pl-c">//线的颜色，定义为红色</span></span>
<span class="pl-s1">                    dashStyle <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>solid<span class="pl-pds">'</span></span>, <span class="pl-c">//默认值，这里定义为实线</span></span>
<span class="pl-s1">                    value <span class="pl-k">:</span> <span class="pl-c1">5</span>, <span class="pl-c">//表示线所在的值</span></span>
<span class="pl-s1">                    width <span class="pl-k">:</span> <span class="pl-c1">3</span>, <span class="pl-c">//标示线的宽度</span></span>
<span class="pl-s1">                    label <span class="pl-k">:</span> {</span>
<span class="pl-s1">                        text <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>WAN<span class="pl-pds">'</span></span>, <span class="pl-c">//标签的内容</span></span>
<span class="pl-s1">                        align <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>right<span class="pl-pds">'</span></span>, <span class="pl-c">//标签的水平位置，水平居右,默认是水平居中center</span></span>
<span class="pl-s1">                        x <span class="pl-k">:</span> <span class="pl-k">-</span><span class="pl-c1">10</span>,</span>
<span class="pl-s1">                        y <span class="pl-k">:</span> <span class="pl-k">-</span><span class="pl-c1">10</span></span>
<span class="pl-s1">                    }</span>
<span class="pl-s1">                } ]</span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            tooltip <span class="pl-k">:</span> {</span>
<span class="pl-s1">                <span class="pl-en">formatter</span> <span class="pl-k">:</span> <span class="pl-k">function</span>() {</span>
<span class="pl-s1">                    <span class="pl-k">return</span> <span class="pl-v">this</span>.series.<span class="pl-c1">name</span></span>
<span class="pl-s1">                            <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">'</span>&lt;br/&gt;&lt;strong&gt;Date:&lt;/strong&gt;<span class="pl-pds">'</span></span></span>
<span class="pl-s1">                            <span class="pl-k">+</span> Highcharts</span>
<span class="pl-s1">                                    .dateFormat(<span class="pl-s"><span class="pl-pds">'</span>%Y-%m-%d %H:%M:%S<span class="pl-pds">'</span></span>, <span class="pl-v">this</span>.<span class="pl-c1">x</span>)</span>
<span class="pl-s1">                            <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">'</span>&lt;br/&gt;&lt;strong&gt;value:&lt;/strong&gt; <span class="pl-pds">'</span></span> <span class="pl-k">+</span> <span class="pl-v">this</span>.<span class="pl-c1">y</span>;</span>
<span class="pl-s1">                }</span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            plotOptions <span class="pl-k">:</span> {</span>
<span class="pl-s1">                series <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    stickyTracking <span class="pl-k">:</span> <span class="pl-c1">false</span></span>
<span class="pl-s1">                },</span>
<span class="pl-s1">                turboThreshold <span class="pl-k">:</span> <span class="pl-c1">0</span></span>
<span class="pl-s1">            <span class="pl-c">//不限制数据点个数</span></span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            series <span class="pl-k">:</span> [],</span>
<span class="pl-s1">            <span class="pl-c">//导出按钮的tip</span></span>
<span class="pl-s1">            lang <span class="pl-k">:</span> {</span>
<span class="pl-s1">                exportButtonTitle <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>Export to PDF<span class="pl-pds">"</span></span></span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            <span class="pl-c">//统计图导出为PDF</span></span>
<span class="pl-s1">            exporting <span class="pl-k">:</span> {</span>
<span class="pl-s1">                <span class="pl-c">//url:"http://export.highcharts.com",</span></span>
<span class="pl-s1">                url <span class="pl-k">:</span> export_server_url,</span>
<span class="pl-s1">                buttons <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    exportButton <span class="pl-k">:</span> {</span>
<span class="pl-s1">                        menuItems <span class="pl-k">:</span> <span class="pl-c1">null</span>,</span>
<span class="pl-s1">                        <span class="pl-en">onclick</span> <span class="pl-k">:</span> <span class="pl-k">function</span>() {</span>
<span class="pl-s1">                            <span class="pl-v">this</span>.exportChart({</span>
<span class="pl-s1">                                type <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>application/pdf<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                                filename <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>Day Chart of WAN Port Performance<span class="pl-pds">'</span></span></span>
<span class="pl-s1">                            }, <span class="pl-c1">null</span>);</span>
<span class="pl-s1">                        }</span>
<span class="pl-s1">                    },</span>
<span class="pl-s1">                    printButton <span class="pl-k">:</span> {</span>
<span class="pl-s1">                        enabled <span class="pl-k">:</span> <span class="pl-c1">false</span></span>
<span class="pl-s1">                    }</span>
<span class="pl-s1">                }</span>
<span class="pl-s1">            }</span>
<span class="pl-s1">        });</span>
<span class="pl-s1"></span>
<span class="pl-s1">        <span class="pl-k">var</span> chart2 <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-en">Highcharts.Chart</span>({</span>
<span class="pl-s1"></span>
<span class="pl-s1">            chart <span class="pl-k">:</span> {</span>
<span class="pl-s1">                renderTo <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>container2<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                type <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>column<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                backgroundColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#FBF8D7<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                zoomType <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>xy<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                reflow <span class="pl-k">:</span> <span class="pl-c1">true</span></span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            title <span class="pl-k">:</span> {</span>
<span class="pl-s1">                text <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>Test chart2<span class="pl-pds">'</span></span> <span class="pl-c">//指定图表标题</span></span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            xAxis <span class="pl-k">:</span> {</span>
<span class="pl-s1">                lineColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                lineWidth <span class="pl-k">:</span> <span class="pl-c1">2</span>,</span>
<span class="pl-s1">                tickWidth <span class="pl-k">:</span> <span class="pl-c1">0</span>,</span>
<span class="pl-s1">                tickColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                startOnTick <span class="pl-k">:</span> <span class="pl-c1">true</span>,</span>
<span class="pl-s1">                maxPadding <span class="pl-k">:</span> <span class="pl-c1">0.1</span>,</span>
<span class="pl-s1">                gridLineColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>#E0E0E0<span class="pl-pds">"</span></span>,</span>
<span class="pl-s1">                gridLineDashStyle <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>solid<span class="pl-pds">"</span></span>,</span>
<span class="pl-s1">                gridLineWidth <span class="pl-k">:</span> <span class="pl-c1">1</span>,</span>
<span class="pl-s1">                categories <span class="pl-k">:</span> [ <span class="pl-s"><span class="pl-pds">'</span>Jan<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Feb<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Mar<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Apr<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>May<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Jun<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Jul<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                        <span class="pl-s"><span class="pl-pds">'</span>Aug<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Sep<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Oct<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Nov<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>Dec<span class="pl-pds">'</span></span> ]</span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            yAxis <span class="pl-k">:</span> {</span>
<span class="pl-s1">                min <span class="pl-k">:</span> <span class="pl-c1">0</span>,</span>
<span class="pl-s1">                startOnTick <span class="pl-k">:</span> <span class="pl-c1">true</span>,</span>
<span class="pl-s1">                lineColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                lineWidth <span class="pl-k">:</span> <span class="pl-c1">2</span>,</span>
<span class="pl-s1">                tickWidth <span class="pl-k">:</span> <span class="pl-c1">0</span>,</span>
<span class="pl-s1">                tickColor <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#000000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                title <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    text <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>WAN Traffic (M)<span class="pl-pds">'</span></span> <span class="pl-c">//指定y轴的标题</span></span>
<span class="pl-s1">                },</span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            plotOptions <span class="pl-k">:</span> {</span>
<span class="pl-s1">                series <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    stickyTracking <span class="pl-k">:</span> <span class="pl-c1">false</span></span>
<span class="pl-s1">                },</span>
<span class="pl-s1">                turboThreshold <span class="pl-k">:</span> <span class="pl-c1">0</span></span>
<span class="pl-s1">            <span class="pl-c">//不限制数据点个数</span></span>
<span class="pl-s1">            },</span>
<span class="pl-s1">            series <span class="pl-k">:</span> [ {</span>
<span class="pl-s1">                data <span class="pl-k">:</span> [ <span class="pl-c1">176.0</span>, <span class="pl-c1">135.6</span>, <span class="pl-c1">148.5</span>, <span class="pl-c1">216.4</span>, <span class="pl-c1">194.1</span>, <span class="pl-c1">95.6</span>, <span class="pl-c1">54.4</span>, <span class="pl-c1">29.9</span>,</span>
<span class="pl-s1">                        <span class="pl-c1">71.5</span>, <span class="pl-c1">106.4</span>, <span class="pl-c1">129.2</span>, <span class="pl-c1">144.0</span> ]</span>
<span class="pl-s1">            } ],</span>
<span class="pl-s1">            <span class="pl-c">//统计图导出为PDF</span></span>
<span class="pl-s1">            exporting <span class="pl-k">:</span> {</span>
<span class="pl-s1">                <span class="pl-c">//url:"http://export.highcharts.com",</span></span>
<span class="pl-s1">                url <span class="pl-k">:</span> export_server_url,</span>
<span class="pl-s1">                buttons <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    exportButton <span class="pl-k">:</span> {</span>
<span class="pl-s1">                        menuItems <span class="pl-k">:</span> <span class="pl-c1">null</span>,</span>
<span class="pl-s1">                        <span class="pl-en">onclick</span> <span class="pl-k">:</span> <span class="pl-k">function</span>() {</span>
<span class="pl-s1">                            <span class="pl-v">this</span>.exportChart({</span>
<span class="pl-s1">                                type <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>application/pdf<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                                filename <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>aaa<span class="pl-pds">'</span></span></span>
<span class="pl-s1">                            }, <span class="pl-c1">null</span>);</span>
<span class="pl-s1">                        }</span>
<span class="pl-s1">                    },</span>
<span class="pl-s1">                    printButton <span class="pl-k">:</span> {</span>
<span class="pl-s1">                        enabled <span class="pl-k">:</span> <span class="pl-c1">false</span></span>
<span class="pl-s1">                    }</span>
<span class="pl-s1">                }</span>
<span class="pl-s1">            }</span>
<span class="pl-s1">        });</span>
<span class="pl-s1"></span>
<span class="pl-s1">        $(<span class="pl-s"><span class="pl-pds">'</span>#export<span class="pl-pds">'</span></span>).<span class="pl-c1">click</span>(</span>
<span class="pl-s1">                <span class="pl-k">function</span>() {</span>
<span class="pl-s1">                    <span class="pl-c">//导出服务器地址</span></span>
<span class="pl-s1"></span>
<span class="pl-s1">                    <span class="pl-k">var</span> url <span class="pl-k">=</span> export_server_url;</span>
<span class="pl-s1">                    <span class="pl-c">//导出格式</span></span>
<span class="pl-s1">                    <span class="pl-k">var</span> type <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>application/pdf<span class="pl-pds">"</span></span>;</span>
<span class="pl-s1">                    <span class="pl-c">//导出的文件名</span></span>
<span class="pl-s1">                    <span class="pl-k">var</span> filename <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>WAN Statistics<span class="pl-pds">"</span></span>;</span>
<span class="pl-s1">                    Highcharts.exportCharts([ chart1, chart2 ], <span class="pl-c1">null</span>, url,</span>
<span class="pl-s1">                            type, filename);</span>
<span class="pl-s1">                });</span>
<span class="pl-s1"></span>
<span class="pl-s1">        <span class="pl-c">// 与后台进行数据交互  </span></span>
<span class="pl-s1">        <span class="pl-k">function</span> <span class="pl-en">getForm</span>(<span class="pl-smi">neId</span>) {</span>
<span class="pl-s1">            $.ajax({</span>
<span class="pl-s1">                type <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>POST<span class="pl-pds">"</span></span>,</span>
<span class="pl-s1">                url <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>&lt;c:url value='/buz/wanStatus.do'/&gt;<span class="pl-pds">"</span></span>,</span>
<span class="pl-s1">                data <span class="pl-k">:</span> {</span>
<span class="pl-s1">                    <span class="pl-s"><span class="pl-pds">"</span>method<span class="pl-pds">"</span></span> <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>getDayStatusChartsOptions<span class="pl-pds">"</span></span>,</span>
<span class="pl-s1">                    <span class="pl-s"><span class="pl-pds">"</span>neId<span class="pl-pds">"</span></span> <span class="pl-k">:</span> neId</span>
<span class="pl-s1">                },</span>
<span class="pl-s1">                <span class="pl-en">success</span> <span class="pl-k">:</span> <span class="pl-k">function</span>(<span class="pl-smi">data</span>) {</span>
<span class="pl-s1">                    <span class="pl-k">var</span> categoriesResult <span class="pl-k">=</span> data.categories;</span>
<span class="pl-s1">                    <span class="pl-k">var</span> chartTitleResult <span class="pl-k">=</span> data.chartTitle;</span>
<span class="pl-s1">                    <span class="pl-k">var</span> timeGranularityResult <span class="pl-k">=</span> data.timeGranularity;</span>
<span class="pl-s1">                    <span class="pl-k">var</span> seriesResult <span class="pl-k">=</span> data.series;</span>
<span class="pl-s1">                    <span class="pl-c">//wan端口带宽值</span></span>
<span class="pl-s1">                    <span class="pl-k">var</span> wanValue <span class="pl-k">=</span> <span class="pl-c1">0.5</span>;</span>
<span class="pl-s1"></span>
<span class="pl-s1">                    <span class="pl-c">//先清除series的值</span></span>
<span class="pl-s1">                    <span class="pl-k">var</span> series1 <span class="pl-k">=</span> chart1.series;</span>
<span class="pl-s1">                    <span class="pl-k">while</span> (series1.<span class="pl-c1">length</span> <span class="pl-k">&gt;</span> <span class="pl-c1">0</span>) {</span>
<span class="pl-s1">                        series1[<span class="pl-c1">0</span>].remove(<span class="pl-c1">false</span>);</span>
<span class="pl-s1">                    }</span>
<span class="pl-s1">                    <span class="pl-k">var</span> color <span class="pl-k">=</span> [];</span>
<span class="pl-s1">                    color.<span class="pl-c1">push</span>(<span class="pl-s"><span class="pl-pds">'</span>#0000ff<span class="pl-pds">'</span></span>);</span>
<span class="pl-s1">                    color.<span class="pl-c1">push</span>(<span class="pl-s"><span class="pl-pds">'</span>#00dd00<span class="pl-pds">'</span></span>);</span>
<span class="pl-s1">                    <span class="pl-c">//添加series的值</span></span>
<span class="pl-s1">                    <span class="pl-k">if</span> (seriesResult <span class="pl-k">!=</span> <span class="pl-c1">null</span>) {</span>
<span class="pl-s1">                        <span class="pl-k">for</span> (<span class="pl-k">var</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; seriesResult.<span class="pl-c1">length</span> <span class="pl-k">&gt;</span> <span class="pl-c1">0</span></span>
<span class="pl-s1">                                <span class="pl-k">&amp;&amp;</span> i <span class="pl-k">&lt;</span> seriesResult.<span class="pl-c1">length</span>; i<span class="pl-k">++</span>) {</span>
<span class="pl-s1">                            <span class="pl-k">var</span> sdata <span class="pl-k">=</span> seriesResult[i].<span class="pl-c1">data</span>;</span>
<span class="pl-s1"></span>
<span class="pl-s1">                            chart1.addSeries({</span>
<span class="pl-s1">                                name <span class="pl-k">:</span> seriesResult[i].<span class="pl-c1">name</span>,</span>
<span class="pl-s1">                                color <span class="pl-k">:</span> color[i],</span>
<span class="pl-s1">                                lineWidth <span class="pl-k">:</span> <span class="pl-c1">2</span>,</span>
<span class="pl-s1">                                data <span class="pl-k">:</span> sdata</span>
<span class="pl-s1">                            }, <span class="pl-c1">false</span>);</span>
<span class="pl-s1">                        }</span>
<span class="pl-s1">                    }</span>
<span class="pl-s1"></span>
<span class="pl-s1">                    <span class="pl-c">//重新画WAN带宽标示线，先删除</span></span>
<span class="pl-s1">                    chart1.yAxis[<span class="pl-c1">0</span>].removePlotLine(<span class="pl-s"><span class="pl-pds">'</span>ymax<span class="pl-pds">'</span></span>);</span>
<span class="pl-s1">                    <span class="pl-k">var</span> plotLine <span class="pl-k">=</span> {</span>
<span class="pl-s1">                        id <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>ymax<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                        color <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>red<span class="pl-pds">'</span></span>, <span class="pl-c">//线的颜色，定义为红色</span></span>
<span class="pl-s1">                        dashStyle <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>solid<span class="pl-pds">'</span></span>, <span class="pl-c">//默认值，这里定义为实线</span></span>
<span class="pl-s1">                        value <span class="pl-k">:</span> wanValue, <span class="pl-c">//表示线所在的值</span></span>
<span class="pl-s1">                        width <span class="pl-k">:</span> <span class="pl-c1">3</span>, <span class="pl-c">//标示线的宽度</span></span>
<span class="pl-s1">                        label <span class="pl-k">:</span> {</span>
<span class="pl-s1">                            text <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>WAN<span class="pl-pds">'</span></span>, <span class="pl-c">//标签的内容</span></span>
<span class="pl-s1">                            style <span class="pl-k">:</span> {</span>
<span class="pl-s1">                                color <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>#ff0000<span class="pl-pds">'</span></span>,</span>
<span class="pl-s1">                                fontWeight <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>bold<span class="pl-pds">'</span></span></span>
<span class="pl-s1">                            },</span>
<span class="pl-s1">                            align <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>right<span class="pl-pds">'</span></span>, <span class="pl-c">//标签的水平位置，水平居右,默认是水平居中center</span></span>
<span class="pl-s1">                            x <span class="pl-k">:</span> <span class="pl-k">-</span><span class="pl-c1">10</span>,</span>
<span class="pl-s1">                            y <span class="pl-k">:</span> <span class="pl-k">-</span><span class="pl-c1">10</span></span>
<span class="pl-s1">                        }</span>
<span class="pl-s1">                    };</span>
<span class="pl-s1">                    chart1.yAxis[<span class="pl-c1">0</span>].addPlotLine(plotLine);</span>
<span class="pl-s1"></span>
<span class="pl-s1">                    <span class="pl-c">//重新设y轴的最大最小值，将wan端口的值显示在图中</span></span>
<span class="pl-s1">                    chart1.yAxis[<span class="pl-c1">0</span>].setExtremes(<span class="pl-c1">0</span>, wanValue <span class="pl-k">+</span> wanValue <span class="pl-k">/</span> <span class="pl-c1">20</span>,</span>
<span class="pl-s1">                            <span class="pl-c1">true</span>);</span>
<span class="pl-s1"></span>
<span class="pl-s1">                    <span class="pl-c">//设置标题</span></span>
<span class="pl-s1">                    <span class="pl-k">var</span> title <span class="pl-k">=</span> {</span>
<span class="pl-s1">                        text <span class="pl-k">:</span> chartTitleResult,</span>
<span class="pl-s1">                        style <span class="pl-k">:</span> {</span>
<span class="pl-s1">                            fontWeight <span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>bold<span class="pl-pds">"</span></span></span>
<span class="pl-s1">                        }</span>
<span class="pl-s1">                    };</span>
<span class="pl-s1">                    chart1.setTitle(title);</span>
<span class="pl-s1">                    chart1.redraw();</span>
<span class="pl-s1">                }</span>
<span class="pl-s1">            });</span>
<span class="pl-s1">        }</span>
<span class="pl-s1"></span>
<span class="pl-s1">        <span class="pl-c">//定时刷新 列表数据  </span></span>
<span class="pl-s1">        $(<span class="pl-k">function</span>() {</span>
<span class="pl-s1">            <span class="pl-c">//每隔x秒自动调用方法，实现图表的实时更新   </span></span>
<span class="pl-s1">            <span class="pl-c1">window</span>.<span class="pl-c1">setInterval</span>(getForm(<span class="pl-c1">158238</span>), <span class="pl-c1">30000</span>);</span>
<span class="pl-s1">            <span class="pl-c1">setInterval</span>(<span class="pl-k">function</span>() {</span>
<span class="pl-s1">                getForm(<span class="pl-c1">158238</span>);</span>
<span class="pl-s1">            }, <span class="pl-c1">20000</span>);</span>
<span class="pl-s1">        });</span>
<span class="pl-s1">    &lt;/<span class="pl-ent">script</span>&gt;</span>
&lt;/<span class="pl-ent">body</span>&gt;
&lt;/<span class="pl-ent">html</span>&gt;</pre></div>

结果图如下：

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-27/1.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-27/1.png" style="max-width:100%;"></a>

<a href="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-27/2.png" target="_blank">
<img src="https://raw.githubusercontent.com/JonathonFly/jonathonfly.github.com/master/_posts/core-samples/pictures/2015-10-27/2.png" style="max-width:100%;"></a>