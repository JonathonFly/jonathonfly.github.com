---
layout: page
title: 欢迎来到我的博客!
---
{% include JB/setup %}

我决定开始在github上write sth,学习过程、成果，忧愁，苦恼，以及日常生活、感悟，或者互联网相关新闻。<br>
题材不限，写一个真实的博客，分享一个真实的我。（年纪大了，记性不好，也许能和10年后的我神交一番）

    
## 最新博文

<ul class="posts">
  {% for post in site.posts limit:10 %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


