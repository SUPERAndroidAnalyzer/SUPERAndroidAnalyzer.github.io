---
layout: post
title: "SUPER 0.4.0 released!"
description: "After some time and an amazing work from our contributors, here we have SUPER 0.4.0 without the APKTool dependency"
category:
tags: [super, release, 0.3.0]
---
{% include JB/setup %}

It's been difficult, long and troublesome, but finally, SUPER has removed its dependency in APKTool.
We were expecting to add some more features for this release, but It was too good to make you wait
for it. SUPER 0.4.0 is here, with really good news for analysis timings.

ApkTool was the Java dependency we used to extract the  XML resource files from the Android
application. Let's give some numbers: The complete analysis has improved by more than 22% thanks to
the removal of ApkTool. Only talking about the extraction part, the speed has improved more than 90%,
which means that now, 90% of the analysis is only in decompilation Java dependencies. Their removal
is being worked on in the [`dalvik`][dalvik] repository.

This also means that the command line options and configuration options regarding to ApkTool have
been removed.

We also moved to codecov for code coverage, and did some code improvements.

[dalvik]: https://github.com/SUPERAndroidAnalyzer/dalvik
