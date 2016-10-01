---
layout: page
title: SUPER Android Analyzer
---
{% include JB/setup %}

<img src="{{ site.url }}/assets/logo.png" alt="SUPER logo" title="SUPER Android Analyzer" style="float:left;width:15em;margin:1em">

## Secure, Unified, Powerful and Extensible Rust Android Analyzer

**Welcome to SUPER!**

Here you can find all the information relevant to the SUPER Android Analyzer. You can find news
too, and of course, the downloads. SUPER is a command-line application that can be used in Windows,
MacOS X and Linux, that analyzes *.apk* files in search for vulnerabilities. It does this by
decompressing APKs and applying a series of rules to detect those vulnerabilities.

## Latest posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> » <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
