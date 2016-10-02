---
layout: post
title: "Our application analysis results"
description: "Here we have, finally, the result analysis from the first SUPER version in 61 apps."
category:
tags: [contributing, hacktoberfest]
---
{% include JB/setup %}

Here we have, finally, the result analysis from the first SUPER version in 61 apps.

*Note: some of the results here show critical vulnerabilities that the application could have
patched in other ways, but they could exist under some conditions. False positives are not shown
here (yes, we have some false positives, but come on, is 0.1.0 version :P).*

## WhatsApp (com.whatsapp)

WhatsApp is the most used Android application, but it shows some potential vulnerabilities that should have been easily solved:

 - **SQL Injection**: Yes, seems impossible in 2016, and in such a widely used application, but its
    code has some SQL Injection vulnerabilities. As noted before. These vulnerabilities could be
    already solved by filtering parameters, but we already have prepared statements that avoid this
    issue in any circumstance. Here an image of one of the vulnerable points:

    ![SQL Injection in WhatsApp]({{ site.url }}/assets/posts/2016/10/whatsapp-sqli.png)

 - **Weak algorithms**: In multiple occasions, WhatsApp uses weak algorithms such as *MD5* for
    unknown purposes. An example is shoun here:

    ![Weak algorithms in WhatsApp]({{ site.url }}/assets/posts/2016/10/whatsapp-weak-alg.png)

Those are the most important ones. More can be seen if you generate a report yourself, or in the
future, in the online report repository, as said above.

## Facebook (com.facebook.katana)

TODO
