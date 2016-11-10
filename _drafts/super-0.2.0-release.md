---
layout: post
title: "SUPER 0.2.0 released!"
description: "As promised, after 6 weeks, here we have SUPER 0.2.0 with many new features and bug fixes!"
category:
tags: [super, release, 0.2.0]
---
{% include JB/setup %}

As promised, here we have SUPER 0.2.0 after 6 weeks since the 0.1.0 release. SUPER 0.2.0 is our
second release, and it makes a big step forward in analysis report customization. The main
characteristic of this release is the new templating system, that will enable users total
customization of reports. We will in fact [create a better template][25] for future releases. The
templating system will be explained in the [templates][templates] section of this release
announcement.

But first, lets talk about the new CLI:

```
USAGE:
    super [FLAGS] [OPTIONS] <package>

FLAGS:
        --bench       Show benchmarks for the analysis
        --force       If you'd like to force the auditor to do everything from the beginning
    -h, --help        Prints help information
        --open        Open the report in a browser once it is complete
    -q, --quiet       If you'd like a zen auditor that won't output anything in stdout
    -a, --test-all    Test all .apk files in the downloads directory
    -V, --version     Prints version information
    -v, --verbose     If you'd like the auditor to talk more than necessary

OPTIONS:
        --apktool <apktool>        Path to the apktool file
        --dex2jar <dex2jar>        Where to store the jar files
        --dist <dist>              Folder where distribution files will be extracted
        --downloads <downloads>    Folder where the downloads are stored
        --jd-cmd <jd-cmd>          Path to the jd-cmd file
        --results <results>        Folder where to store the results
        --rules <rules>            Path to a JSON rules file
        --template <template>      Path to a results template file
    -t, --threads <threads>        Number of threads to use

ARGS:
    <package>    The package string of the application to test
```

As you can see, the CLI has been completely redesigned to enable further customization of the
analysis. Users can now specify most of the configuration options that would be in the
`config.toml` file as command arguments. This can be great for software that could require specific
automation of analysis for each batch or application, we need to remember that this software is
intended to be used for massive analysis.

We have also added a `--test-all` flag that does not require a `<package>` to be specified, since
it will search for all applications in the *downloads* folder and analyze all of them. This was a
port of a small shell script we had that we were using many times, and now it's done in the core of
SUPER, which makes it multi-platform and really fast. About that *downloads* folder, it's no longer
required to have an actual *downloads* named folder, and *.apk* files can be in any place (by
default, the current directory). We now also support relative paths to *.apk* files too.

Another useful option we have added is the `--open` option. This option will open the HTML report
in your usual browser once it's finished. It could be a pain in the ass to search for it in the
tree.

In the report generation we have added vulnerable line highlighting, which makes it easier to spot
issues, and we also improved some `exported` attributes searching, which, BTW,
[still requires improvements][35].

We have of course made a ton of under-the-hood improvements with a total of 101 commits from 7
contributors. We want to specially thank all the help we have received from the community, with
contributions from **[@pocket7878][pocket7878]**, **[@VoltBit][VoltBit]**, **[@b52][b52]**,
**[@nxnfufunezn][nxnfufunezn]** and **[@atk][atk]**. This version has seen 95 changed files with
3,491 lines added and 1,974 lines deleted. It has been a real challenge that raises the total *LOC*
of the project to 12,061.

Complete changelog can be read [here][changelog];

## Templates
[templates]: #templates

The big rework for this release has been the new templating system. Now templates are stored in
their own folder in the *templates* folder (that will be in a different place depending on the OS
and can be configured via `config.toml`). For example, the default template, *super* is stored in
`templates/super`. Templates are written in [Handlebars][handlebars], and we offer some helpers
that we'll explain next. A template **must** have at least `code.hbs`, `report.hbs` and `src.hbs`
files. Other *.hbs* files in the template root directory can be used with template inclusion, the
same way *super* template does with its *vulnerability* template. To include it, we use
`{{> vulnerability }}`.

We have added some cool helpers to make the HTML code generation easier:

 - `line_numbers`: Given a vulnerability renders a list of the relevant lines in the vulnerability.
   Callers can add a second parameter, a line separator, that will separate lines. By default it's
   `<br>`.
 - `all_lines`: It does the same as above but creates a list for all the lines in the given code
   snippet, starting from 1. Useful for code representation. As before, it has a line separator that
   defaults to `<br>`.
 - `all_code`: Renders the given code with an specific format for each line, and accepts an
   optional line separator that defaults to `<br>`:
   ```html
   <code id="code-line-{{ line_number }}">{{ line_indent }}<span class="line_body">{{ line_code }}</span></code>{{ line_separator }}
   ```
 - `html_code`: Renders the code for the given vulnerability. It also accepts a line separator as
   an optional second argument (again, it defaults to `<br>`). Each line is rendered as-is if it's
   not vulnerable, or with this format if it is:
   ```html
   <code class="vulnerable_line {{ criticality }}">{{ line_indent }}<span class="line_body">{{ line_code }}</span></code>{{ line_separator }}
   ```
   Where the criticity is one of *critical*, *high*, *medium*, *low* or *warning*.
 - `report_index`: Generates the index number for the given vulnerability. It requires a second
   argument that would be the index of the vulnerability (can be obtained with the `@index`
   variable in an `each` block), and the length of the list of this vulnerability criticality. For
   example, for the third critical vulnerability in a report that contains 240 critical
   vulnerabilities it will render *C003*. The list length can be obtained with variables that we
   will explain next.
 - `generate_menu`: Generates the menu from the given menu object. It will generate an structure
   like this:
   ```html
   <ul>
       {{ for each file/folder }}
       <li>
       {{ if folder }}
       <a href="#" title="{{ folder_name }}"><img src="../img/folder-icon.png">{{ folder_name }}</a>
       <ul>
           {{ recursive menu rendering for all files/folders in menu }}
       </ul>
       {{else}}
       <a href="{{ code_html_file }}" title="{{ file_name }}"><img src="../img/{{ file_type (xml/java) }}-icon.png">{{ file_name }}</a>
       {{endif}}
       <li>
   ```

TODO: variables




[25]: https://github.com/SUPERAndroidAnalyzer/super/issues/25
[35]: https://github.com/SUPERAndroidAnalyzer/super/issues/35
[pocket7878]: https://github.com/pocket7878
[VoltBit]: https://github.com/VoltBit
[b52]: https://github.com/b52
[nxnfufunezn]: https://github.com/nxnfufunezn
[atk]: https://github.com/atk
[handlebars]: http://handlebarsjs.com/
[changelog]: https://github.com/SUPERAndroidAnalyzer/super/blob/0.2.0/CHANGELOG.md
