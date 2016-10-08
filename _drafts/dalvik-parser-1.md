---
layout: post
title: "Creating a Dalvik parser in Rust (Part 1)"
description: "One of the biggest challenges in SUPER will be to remove Java dependencies. First, we will need a complete Dalvik parser. (Part 1)"
category:
tags: [dalvik, parser, rust]
---
{% include JB/setup %}

As you could imagine, a good Rust Android analyzer would not be so if it wasn't 100% developed in
Rust. SUPER has its analyzer 100% developed in Rust, but it still has some dependencies for
translating the *.apk* file to the Java code we analyze. That is something that will change in the
future, and it will start by creating a really efficient [Dalvik][dalvik_wikipedia] parser. Even
though Dalvik itself is discontinued, *.dex* files inside *.apk* files still use the
[Dalvik executable format][dex_format].

[dalvik_wikipedia]: https://en.wikipedia.org/wiki/Dalvik_(software)
[dex_format]: http://source.android.com/devices/tech/dalvik/dex-format.html
