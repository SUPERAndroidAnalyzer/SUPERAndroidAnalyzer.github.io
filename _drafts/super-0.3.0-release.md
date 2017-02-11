---
layout: post
title: "SUPER 0.3.0 released!"
description: "Here we have, finally, the third release of SUPER!"
category:
tags: [super, release, 0.3.0]
---
{% include JB/setup %}

It took a while, but we finally have SUPER 0.3.0 with us. SUPER 0.3.0 is centered around
improvements in speed and future compatibility. There are a few extra features, but most of the work
has been under the hood. Two new CLI options have been introduced, though:

- You can now specify the minimum criticality of a vulnerability for being reported. Using the
  `--min-criticality` CLI option, you can specify if the minimum reported criticality should be
  *warning*, *low*, *medium*, *high* or *critical*.
- Optional JSON and HTML reports: By default, SUPER will generate an HTML report, but no JSON
  report. This behaviour can be changed either by changing two configuration options in the
  *config.toml* file (`html_report` and `json_report`) or by invoking the script with `--json` or
  `--html` parameters. By default, if `--json` is used, the HTML report won't get generated, but if
  you want both, you can specify so by using both options: `--json --html`.

**And, finally, here we have tab completions!**

If you now install SUPER using one of the provided packages for UNIX, you will get tab completions.
So, anytime you don't exactly know the command, you can simply press TAB and you will get
suggestions or even command completions. This works for Bash, Fish and ZSH.

You can check the rest of the changes in the [changelog][changelog] of the project. This release
also start a new release cycle in SUPER. Until now, we were trying to release new versions every 6
weeks. As you can see in this version, that is no longer possible, and it effectively limits our
options to create big features (such as [removing Java dependencies][22]). That's why we have moved
to a "when ready" release cycle. We will now continue working in our spare time, and when we feel
that a new version should be released, we will do so. So, don't expect the 0.4.0 version in March!

We want to give special thanks to [@gnieto][gnieto] for his contributions to this release. As
always, you can download the package for your distribution at the
[downloads page]({{ site.url }}/download.html).

[22]: https://github.com/SUPERAndroidAnalyzer/super/issues/22
[gnieto]: https://github.com/gnieto
