---
#layout: post-argon
title: Visual editors for Markdown
date: 2023-09-08 15:00:00 +0100
author: Igor
categories: Blog
tags: [azure, devops, caf]
#permalink: /post1/
slug: markdown-visual-editors
excerpt_separator: <!--more-->
#redirect_from: [/lorem, /post/lorem]
image: https://github.com/iromanovsky/irom.info/assets/15823576/bee5bd07-3fe6-4a5c-a373-c2513aec1fd3
description: >-
    It's frustrating to work with tables in Markdown when you need more complex formatting. I will share which visual editors for Markdown can solve this issue.
#published: false
---
<div aligh="center">

![looking-at-markdown-tables](https://github.com/iromanovsky/irom.info/assets/15823576/bee5bd07-3fe6-4a5c-a373-c2513aec1fd3)

</div>

[Markdown](https://www.markdownguide.org/getting-started/) became an industry standard for Docs as Code a long time ago. It is used on Azure DevOps, GitHub, MS Docs, and even on my humble blog. However, one thing has been bothering me for all that time: working with tables in Markdown when you need more complex formatting. On the cover picture, you can see how unfriendly it is. In this post, I will share which visual editors for Markdown can solve this issue.

<!--more-->

While many VS Code extensions, online, and desktop Markdown editors claim to offer visual editing capabilities (WYSIWYG), most of them turn out to be nothing more than code editors with snippets and live previews.

However, I found two standout tools that genuinely provide visual editing features:

- [Typora](https://typora.io), shareware with a 15-day free trial, available for Win, Mac, and Linux. Works really well for my needs.
- [MarkText](https://github.com/marktext/marktext), open source, available on the same platforms. I could not start the app on an ARM-based Mac because of a missing digital signature. On Windows, it works and looks like a clone of Typora, but much worse (some basic functions like "undo" do not work, it is touching the source markdown code too much). But it's free. 

This is how Markdown table editing looks in Typora:

<div aligh="center">

![Typora](https://github.com/iromanovsky/irom.info/assets/15823576/783acf88-8763-43e1-9c90-3ea6c30a3603)

</div>

If you know of any other tools that allow visually editing markdown tables, please let me know in the [comments](https://www.linkedin.com/feed/update/urn:li:activity:7105938894491168768/).
