---
layout: page
title: SUPER Android Analyzer - Download
---
Here you can download SUPER for [Windows]({{ page.url }}#download-for-windows),
[Linux]({{ page.url }}#download-for-linux) and [MacOS X]({{ page.url }}#download-for-macos-x). We
only provide 64-bit builds. If you need support for 32 bits, you will need to
[compile SUPER from source]({{ page.url }}#compile-from-source).

## Download for Windows

Windows download has been tested in Windows 8.1 and Windows 10, but it might work in other versions
as well.

<div class="download" style="margin-left:15em;width:10em;height:5em"><a href="https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.5.0/super-analyzer-0.5.0-windows-x86_64.exe" title="Download SUPER for Windows"><img src="{{ site.url }}/assets/os_logos/windows.svg" alt="Windows logo"><br>Windows 8.1+ (64-bit)</a></div>

<div style="clear:both;"></div>

## Download for Linux

Linux downloads have been tested in the given OS versions, but they might work in other versions as
well, or even in other distributions.

<div class="download" style="margin-left:0"><a href="https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.5.0/super-analyzer_0.5.0_debian_amd64.deb" title="Download SUPER for Debian"><img src="{{ site.url }}/assets/os_logos/debian.svg" alt="Debian logo"><br>Debian 9.5<br>(64-bit)</a></div>

<div class="download"><a href="https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.5.0/super-analyzer_0.5.0_ubuntu_amd64.deb" title="Download SUPER for Ubuntu"><img src="{{ site.url }}/assets/os_logos/ubuntu.svg" alt="Ubuntu logo"><br>Ubuntu 18.04<br>(64-bit)</a></div>

<div class="download"><a href="https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.5.0/super-analyzer-0.5.0-1.el7.x86_64.rpm" title="Download SUPER for CentOS"><img src="{{ site.url }}/assets/os_logos/centos.svg" alt="CentOS logo"><br>CentOS 7.5<br>(64-bit)</a></div>

<div class="download"><a href="https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.5.0/super-analyzer-0.5.0-1.fc28.x86_64.rpm" title="Download SUPER for Fedora"><img src="{{ site.url }}/assets/os_logos/fedora.svg" alt="Fedora logo"><br>Fedora 28<br>(64-bit)</a></div>

<div style="clear:both;"></div>

## Download for MacOS X

<div class="download" style="margin-left:15em;width:10em;height:5em"><a href="https://github.com/SUPERAndroidAnalyzer/super/releases/download/0.4.1/super-analyzer_0.4.1_macosx_x86_64.pkg" title="Download SUPER for MacOS X"><img src="{{ site.url }}/assets/os_logos/macos.svg" alt="Apple logo"><br>MacOS X (64-bit)</a></div>

<div style="clear:both;"></div>

## Compile from source

In the case that you want or need to compile SUPER from its sources, you will need to download
those sources first. You can download SUPER 0.5.0 sources in
[zip](https://github.com/SUPERAndroidAnalyzer/super/archive/0.5.0.zip) or in
[tar.gz](https://github.com/SUPERAndroidAnalyzer/super/archive/0.5.0.tar.gz). You can also clone
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

This will create `target/release/super`, which will be the executable that you will be able to
[use]({{ site.production_url }}#usage).

Note that if you are compiling SUPER in Windows, you will need to add an extra flag to the compiler
to avoid strange characters in the shell. To compile SUPER in Windows, use this command:

```
cargo build --release --features no-color
```