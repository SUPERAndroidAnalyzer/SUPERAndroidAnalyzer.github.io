---
layout: post
title: "SUPER 0.4.0 released!"
description: "After some time and an amazing work from our contributors, here we have SUPER 0.4.0 without the APKTool dependency"
category:
tags: [super, release, 0.4.0]
---
{% include JB/setup %}

It's been difficult, long and troublesome, but finally, SUPER has removed its dependency in ApkTool.
ApkTool was a library we used to get the XML resources of an *.apk* file to be able to read contents
such as the manifest file, the string tables, color tables and so on. The main issue was that using
that dependency, a Java dependency, was taking about 25% - 50% of the analysis time, depending on
the application being analyzed. While our analysis was still much faster than the competitors,
spending so much time only decompressing XML resources was not what we wanted to do. If we have to
spend most of the time on something, it should be analyzing the application looking for more
vulnerabilities.

That's why when our new core team developer (more on that later) joined us with an idea to
substitute it, we were extremely happy. Since then, and after a couple of months of development, we
can finally announce that we no longer depend on ApkTool. But what does this mean for analysis
times?

Well, these benchmarks on the *WhatsApp* application show that our analysis has gotten about 30%
faster thanks to a pure Rust apk resource decoder, while the decompression part has moved from two
steps that were costing us about 7 seconds to a unique subsecond step. This is a 1 order of
magnitude improvement!

Before:
```text
ApkTool decompression: 6.3411489063s
Dex extraction: 0.78994060s
Dex to Jar decompilation: 11.679701717s
Decompilation: 9.956497479s
Total static analysis: 1.210060706s
Report generation: 0.2060081s
Total time for com.whatsapp: 29.490066933s
```

After:
```text
Apk decompression: 0.683341042s
Dex to Jar decompilation: 11.527798122s
Decompilation: 9.779236797s
Total static analysis: 0.607118707s
Report generation: 0.1840899s
Total time for com.whatsapp: 22.888668955s
```

As you can see, now the main steps of the analysis are the Dex to Jar transition (done by a Java
dependency, `dex2jar`) and the actual decompilation of the code to Java (done by another Java
dependency, `jd-cli`). Their removal is being worked on in the [`dalvik`][dalvik] repository.

The removal of ApkTool also means that the CLI loses all references to ApkTool configuration, and
the configuration file itself also loses them.

## Other features or bug fixes

Some small features have also been added to the release, even though we didn't want to make you wait
for the ApkTool removal, so the rest of things are minor details. To start, benchmarks now show what
Java dependency is being used in the sections where Java dependencies are used, which will make it
easier to see Rust's benefits and to understand what should be improved.

A bug that would make SUPER try to open the HTML report even if only the JSON report had been
generated was fixed. Now, when you use the `--open` flag with the `--json` flag, SUPER will open
the JSON report if there is a program associated with `.json` file extension. If `--html` flag is
provided, or no `--json` flag is present, it will open the HTML report.

The `--force` flag is now less aggressive. It won't remove a JSON report if only the HTML report is
being generated, and the othey way around: it won't remove the HTML report if only the JSON report
is being generated.

## Development notes

In this release, the `error-chain` errors were moved to their own module, since we changed our error
reporting to use that dependency. We updated the Rust code formatting to `rustfmt` 0.8.3 version,
and we added [codecov][codecov] test coverage report, since Coveralls was no longer working
properly.

The project also moved away from `rust-crypto`, since it's now long unmantained. We now use `md5`,
`sha1` and `sha2` libraries for application fingerprinting. Documentation was improved in many areas
of the application, closing some GitHub issues in the process.

Dependencies were upgraded too, and a lot of new Clippy lints were added, some of them still allowed
by default, but that will change in the future.

## New core team member

As talked before, we have a new core team member, [**@gnieto**][gh_gnieto]. He has been helping us a
lot removing the ApkTool dependency, and he was the one who created the [`abxml`][abxml] library
that allowed us to remove the slow Java dependency. Welcome to the team!

## Inclusion in Android Tamer

As a small extra, you might have seen that we released SUPER 0.3.1 just a week after SUPER 0.3.0.
This was due to some incompatibility issues with Debian packaging system, that have now been solved.
The motivation behind this release was for SUPER to be included in the [Android Tamer][a_tamer]
operating system in their own repositories.

This Linux distribution gives tools to work with Android pentesting, and we are glad that they
selected us to be added to the distribution main repositories.

[dalvik]: https://github.com/SUPERAndroidAnalyzer/dalvik
[codecov]: https://codecov.io/gh/SUPERAndroidAnalyzer/super
[gh_gnieto]: https://github.com/gnieto
[abxml]: https://github.com/gnieto/abxml-rs
[a_tamer]: https://androidtamer.com/
