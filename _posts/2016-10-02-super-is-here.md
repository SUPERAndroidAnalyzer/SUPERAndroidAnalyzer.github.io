---
layout: post
title:  "SUPER Android Analyzer is here!"
date:   2016-10-02 12:00:00 +0000
categories: super release 0.1.0
---
<img src="{{ site.url }}/assets/logo.png" alt="SUPER logo" title="SUPER Android Analyzer" style="float:left;width:15em;margin:1em">

SUPER Android Analyzer is here. SUPER, the Secure, Unified, Powerful and Extensible Rust Android
Analyzer version 0.1.0 has been released, and with it, all the power of it is now on your hands.

But first, let's talk about SUPER. **What is SUPER?** SUPER is a command-line application that can
be used in Windows, MacOS X and Linux, that analyzes *.apk* files in search for vulnerabilities.
It does this by decompressing APKs and applying a series of rules to detect those vulnerabilities.
We currently detect SQL Injections, XSS vulnerabilities, *superuser* checks, and a bunch of other
vulnerabilities detected by a total of 37 rules, that will be expanded and optimized in future
updates.

But, why create a new analyzer? Is it not enough with *MobSF*, *Qark*, *Androbugs*…? Well, we think
it's not enough. All of them have two main issues we wanted to fix: They are written in Java or
Python and they are not easily extensible. They are not meant to be used by businesses directly
working in Android analysis, and don't put that kind of functionality first.

Our approach solves those issues in different ways: We first decided to use
**[Rust](https://www.rust-lang.org/)** as our programming language. The language developed openly
by Mozilla Foundation gives us lots of utilities to work with regular expressions, files etc. and,
most importantly, it enables us to create a secure software that does not depend in *JVM* or *JIT*
compilers. With Rust, stack overflows, segmentation faults etc. are directly not possible, which
makes sense in a security centered application. And it also gives us enough power to do efficient
analysis, giving us the option to automate it in high volume. This is given by Rust zero-cost
abstractions, that gives us an efficiency only comparable to C/C++.

And secondly, we decided to make the software 100% extensible: All rules are centered in a
`rules.json` file, and each company or tester could create its own rules to analyze what they need.
It's also modular, so that new developments can easily add new functionality. Finally, a templating
system for results reports (that will be improved in future updates) gives users the ability to
personalize the report.

Only with this first release, still not being stable, SUPER has shown to be much faster than other
frameworks, while catching more potential vulnerabilities. Moreover, SUPER gives some *good
practice* advices to enable programmers to select better their development practices.

It also gives great code review tools, directly in the HTML report, so that anyone can search
through the generated code with syntax highlighting for even better vulnerability analysis.

## Installation

There are multiple ways of installing SUPER in your computer. SUPER is really portable, and it's
possible to run under Windows, Linux and MacOS X:

### Windows

We have created a package for Windows 64 bits and can be downloaded below. If you want to run SUPER
in 32-bit Windows, you will need to [compile it from source]({{ page.url }}#compiling-from-source).

We currently depend on OpenSSL and Java, which are not installed in Windows by default. That's why
Windows users need to download and install OpenSSL and Java and add it to the `PATH` variable of
the system:

  - You can download OpenSSL [here](http://gnuwin32.sourceforge.net/packages/openssl.htm).
  - To add it to the PATH, go to *Control panel > System > Environment variables > PATH* and add
    an entry to OpenSSL (it should be something like this: `C:\Program Files (x86)\GnuWin32\bin`).
  - Do the same for Java, which can be downloaded [here](http://www.java.com/en/download/).

It's very easy to run SUPER in Windows. We provide an executable which extracts SUPER to the
specified folder so users don't need to configure anything else. To run SUPER, double click in the
`.exe` file that you just downloaded. Select the extraction folder, and once extracted, start a
console in that folder. You can run the program with the `super` command. For more information on
usage, check the [usage section]({{ page.url }}#usage) of the post.

### Linux

We have made it really easy to install SUPER in Linux. We have many packages for different
distributions so you will be able to probably find yours. If that's not the case, you can use a
package similar to your distribution (CentOS package for RedHat based distributions, Debian/Ubuntu
packages for Debian based distributions, etc.) or if you prefer (or don't find an appropriate
package), you can compile it from source, as it's shown in the last section of the installation.

 - [Debian 8.6](https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.1.0/super_0.1.0_debian_amd64.deb) (DEB)
 - [Ubuntu 16.04](https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.1.0/super_0.1.0_ubuntu_amd64.deb) (DEB)
 - [CentOS 7](https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.1.0/super-0.1.0-1.el7.centos.x86_64.rpm) (RPM)
 - [Fedora 24](https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.1.0/super-0.1.0-1.fc24.x86_64.rpm) (RPM)

### MacOS X

For now, MacOS users will need to build the package from source. We already have an issue
([#18](https://github.com/SUPERAndroidAnalyzer/super/issues/18)) to create packages for it, though,
and should be solved by SUPER 0.2.0.

### Compiling from source

In the case that you want or need to compile SUPER from its sources, you will need to download
those sources first. You can download SUPER 0.1.0 sources in
[zip](https://github.com/SUPERAndroidAnalyzer/super/archive/0.1.0.zip) or in
[tar.gz](https://github.com/SUPERAndroidAnalyzer/super/archive/0.1.0.tar.gz). You can also clone
the *Git* repository by running:

```
git clone https://github.com/SUPERAndroidAnalyzer/super.git
```

Once you clone or download+decompress the source code in a folder and move to it, you will need to
compile it. This is really easy to do, but you will need Rust/Cargo to be installed in your system.
You can install it with the instructions provided in [rustup.rs](https://rustup.rs/). Once
installed, compiling SUPER is as simple as running the following:

```
cargo build --release
```

This will create `target/release/super`, which will be the executable that you will be able to use
in the following steps.

## Usage

To use SUPER you will need to configure in the `config.toml` file the paths of `vendor`,
`downloads` and other folders. If the package was installed using one of the provided packages,
the default configuration will work. If not, probably the same folder where `super` is called from
will work (usually the installation folder, or the current workspace). If willing to use other
paths, a sample configuration file can be found in the `config.toml.sample` file.

Lots of parameters can be configured in that file, such as the threads that will be used for the
analysis or the rule file. We currently require some Java dependencies that are included in the
packages. This should change once [#22](https://github.com/SUPERAndroidAnalyzer/super/issues/22) is
implemented.

Once configured, the use is simple: Move the *.apk* file that you want to test to the `downloads`
folder. The name of the *.apk* file should be `{package_name}.apk`. For example:
`com.instagram.android.apk`. After that, running SUPER is as easy as running this:

```
super {package_name}
```

If SUPER is installed in the current directory, in Unix, you will need to run `./super` instead.
The package name will be the *.apk* file name without the extension. So in the example above, the
command would be this:

```
super com.instagram.android
```

This will decompress all the files in the `dist/{package_name}` folder, analyze them with the rules
in the `rules.json` file (and configuration in the `config.toml` file) and generate the results in
the `results/{package_name}` folder.

In the results page, an `index.html` file will contain the report, while a `report.json` file will
have a machine readable format of that report. In the case of using the *verbose* mode (see below),
the report will also be shown in the terminal.

### Command line interface

If you run `super --help`, you will see the following:

```
SUPER Android Analyzer 0.1.0
SUPER Android Analyzer Team <contact@superanalyzer.rocks>
Audits Android apps for vulnerabilities

USAGE:
    super [FLAGS] <package>

FLAGS:
        --bench      Show benchmarks for the analysis.
        --force      If you'd like to force the auditor to do everything from the beginning.
    -h, --help       Prints help information
    -q, --quiet      If you'd like a zen auditor that won't talk unless it's 100% necessary.
    -V, --version    Prints version information
    -v, --verbose    If you'd like the auditor to talk more than necessary.

ARGS:
    <package>    The package string of the application to test.
```

The usage is really straightforward. The `-v` or `--verbose` option will show loads of information
in the terminal itself, and the analysis will be performed much slower. The intention in verbose
mode is partially to show how the vulnerabilities are cached and how the analysis is performed.

In normal mode, small messages will be printed indicating the analysis stage. The analysis is
usually really fast (takes a really big app less than one minute in a Carbon X1), but it will in
any case show some prints. This can be avoided by using the `-q` or the `--quiet` option. This will
avoid **all** output from *stdout* even though there might be some output in *stderr* if something
doesn't go as expected.

The `--force` option will regenerate the distribution files and results even if that application
has already been analyzed. By default, if the application distribution files exist in
`dist/{package_name}` they won't be generated, and if the results are already in
`results/{package_name}` they won't be generated either. Those are independent, so if the
distribution files are generated but the report has been deleted, only the report will be generated.

The `--bench` flag will generate benchmarks for the current analysis. This is useful to see the
real performance of the analysis. An example benchmark can be this:

```
Benchmarks:
ApkTool decompression: 7.278750647s
Dex extraction: 0.98327826s
Dex to Jar decompilation: 11.473415714s
Decompilation: 11.695567790s
Manifest analysis: 0.5059071s
Certificate analysis: 0.5984248s
Rule loading: 0.15836818s
File analysis: 1.312931838s
Total code analysis: 1.349633866s
Total static analysis: 1.362122448s
Report generation: 1.154849626s
Total time: 33.824436350s
```

As you can see, most of the time is expent in the extraction and decompilation phases, which are
one of the priorities for the next version
([#22](https://github.com/SUPERAndroidAnalyzer/super/issues/22)), since they use external Java
dependencies.

## Development

SUPER has been developed in [GitHub](https://github.com/SUPERAndroidAnalyzer/super) as an open
source software. It's licenses under GPLv3 (or, at your option, any later version). We use
[Git-flow](http://nvie.com/posts/a-successful-git-branching-model/) as our development model and we
have created some
[guidelines](https://github.com/SUPERAndroidAnalyzer/super/blob/master/contributing.md) to
contribute. It has been developed by the [SUPER team]({{ site.url }}/about/#team).

### Things to do

There are still lots of things to do (that's why this version is 0.1.0 and not 1.0.0). You can find
them in the [issue tracker](https://github.com/SUPERAndroidAnalyzer/super/issues). We have created
some [labels](https://github.com/SUPERAndroidAnalyzer/super/labels) to make it easier track issues
and to select those to fix or implement. So, for example, we have a list of
[easy issues](https://github.com/SUPERAndroidAnalyzer/super/labels/D-Easy), a list of
[bugs](https://github.com/SUPERAndroidAnalyzer/super/labels/Bug), etc. We even have some issues
[for newcomers](https://github.com/SUPERAndroidAnalyzer/super/projects/5)!

## Our results

For this first version we analyzed a total of 61 applications in the Google Play Store, in
different categories. The complete list is shown here. An online report repository is going to be
created in the future to show all the generated reports for many applications and versions, but for
now, a post will be shown in the next days with information about the vulnerabilities that were
found.

The applications analyzed have been these:

 - 16 of the 20 most downloaded apps in Spain:
   - WhatsApp (com.whatsapp)
   - Facebook Messenger (com.facebook.orca)
   - Wallapop (com.wallapop)
   - Bottle Flip (com.metroxylon.Bottle.Flip)
   - Facebook (com.facebook.katana)
   - Instagram (com.instagram.android)
   - Clash Royale (com.supercell.clashroyale)
   - FIFA 17 Companion (com.ea.gp.fifaultimate)
   - Spotify (com.spotify.music)
   - Pokémon GO (com.nianticlabs.pokemongo)
   - Snapchat (com.snapchat.android)
   - musical.ly (com.zhiliaoapp.musically)
   - S Photo Editor (com.steam.photoeditor)
   - Rolling Sky (com.turbochilli.rollingsky)
   - DIA (es.dia)
   - Flip Diving (com.motionvolt.flipdiving)
 - 18 of the 20 most downloaded apps ('Beauty' category):
   - Beauty tips for skin (net.linktomedia.trucosdebelleza)
   - BirchBox (com.birchbox.birchbox)
   - Contourning Step (com.mobincube.contouring_paso_a_paso.sc_3PCIFL)
   - Cortes de Cabello para mujer (com.cortesdecabello.mujer)
   - La dieta de la manzana (com.mobincube.la_dieta_de_la_manzana.sc_39YVRJ)
   - Factor Quema Grasa (com.instagram.android)
   - Fake Braces (com.krissitop.Fakebraces)
   - Gold Rose (com.launcher.gold.rose.business)
   - Cortes de pelo 2016 (com.mobincube.android.sc_33JPWH)
   - La Dieta Radical (com.mobincube.la_dieta_radical.sc_3X6S6A)
   - Luxury Brand Stores (com.luxury.nivorboq_pjzbjddw)
   - Mirror Zoom (com.barilab.handmirror.googlemarket)
   - Perfect 365 One Tap Makeover(com.arcsoft.perfect365)
   - Pink Piano (com.launcher.theme.t204067398)
   - Pink Rose Diamond (com.launcher.theme.pink.rose.diamond)
   - Snap Doggy Face for Snapchat (com.snapdoggyface.snapphotofilterdoggyface)
   - Beauty Make Up Photo Editor (app.alert.bueatymackup)
   - Academy Beauty Club (com.ionicframework.mobile860711)
 - 12 of the 20 most downloaded shopping apps in Spain:
   - Bershka (com.inditex.ecommerce.bershka)
   - chicfy - compra y vende moda (com.chicfy)
   - coches.net: anuncios de coches (coches.net)
   - eBay: Buy, sell and save (com.ebay.mobile)
   - Home - Design & Decor Shopping (com.contextlogic.home)
   - Lidl - Offers & Leaflets (de.sec.mobile)
   - Milanuncios (com.muba.anuncios)
   - Privalia, outlet moda online (com.privalia.privalia)
   - SheIn (THE YUB) - SheInside. (com.zzkko)
   - vente-privee (com.venteprivee)
   - Wish - Shopping made fun (com.contextlogic.wish)
   - Zalando - Shopping fashion (de.zalando.mobile)
 - 15 of the 20 most downloaded apps ('Pokeradar' category):
   - Map for Poke Radar (com.poke.radar.pokemon.go.maps-4)
   - GO Map - Para Pokemon GO (com.go.map-10)
   - GO Tools for Pokemon GO (com.aumentia.pokefind-28)
   - Live Map - for Pokemon GO (xyz.devmasters.pokemaster-16)
   - GO Track - For Pokemon GO (CS) (com.pokedev.poketrack-1475011804)
   - Radar for Pokemon GO (com.poke.radarriver-5)
   - Map for Pokemon GO (us.pokemap.find.pokemons.free2-4)
   - GO Map - For Pokemon (com.geoapps.gomap-44)
   - GO Utilidades de Pokemon GO (com.pokecrew.pokecrewmap-14)
   - Go Map for Pokemon GO (com.gomapforpokemongo-4)
   - GO Locator for Pokemon GO (com.map.pokemon.exchange.pokeexchange-3)
   - Monster Radar (ru.niceappsandgames.pokeradar-3)
   - Map Radar For Pkmn Go (com.mapradarforpkmn-1.apk)
   - Guide Pokemon GO Tips Tricks (com.bestgameguideandwalkthrough.pokemongo-2)
   - Guide Poke Vision Radar GO Map (guide.poke.radar-1)
