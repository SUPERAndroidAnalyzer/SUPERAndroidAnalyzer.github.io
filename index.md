---
layout: page
title: SUPER Android Analyzer
---
{% include JB/setup %}

<img src="{{ site.url }}/assets/logo.png" alt="SUPER logo" title="SUPER Android Analyzer" style="float:left;width:15em;margin:1em">

## Secure, Unified, Powerful and Extensible Rust Android Analyzer

**Welcome to SUPER!** Here you can find all the information relevant to the SUPER Android Analyzer.
You can find news too, and of course, the downloads. SUPER is a command-line application that can
be used in Windows, MacOS X and Linux, that analyzes *.apk* files in search for vulnerabilities. It
does this by decompressing APKs and applying a series of rules to detect those vulnerabilities.

SUPER has been developed in [Rust](https://www.rust-lang.org/), a systems programming language that
gives performance similar to well-written C/C++, while avoiding memory leaks, data races, and
giving memory safety. This programming language was selected to improve all the Java or Python based
alternatives. It also gives us the ability to perform much more powerful checks faster.

## Latest news

<ul class="posts">
  {% for post in site.posts %}
    <li><time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_string }}</time> » <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
