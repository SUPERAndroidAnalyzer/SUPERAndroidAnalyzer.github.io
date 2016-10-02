---
layout: page
title: SUPER Android Analyzer - News
---
# News
{% for post in site.posts %}
<article>
  <h2 style="margin-top:0"><a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br><small><time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time></small></h2>
  {{ post.excerpt }}
  <a href="{{ post.url }}" title="Read more">Read more</a>
</article>
{% endfor %}
