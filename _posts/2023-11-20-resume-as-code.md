---
#layout: post-argon
title: Resume as Code
date: 2023-11-20 18:00:00 +0000
last_modified_at: 2023-11-20 21:30:00 +0000
author: Igor
categories: [Life, Code]
tags: [career, code, oss]
permalink: /blog/resume-as-code
slug: resume-as-code
excerpt_separator: <!--more-->
#redirect_from: [/post/azure-pe-name-resolution]
image: https://github.com/iromanovsky/irom.info/assets/15823576/15eb18df-9c96-401d-b88d-14aa1ccb96a7
description: >-
    Explore how to use LinkedIn data to create versatile resumes in PDF/web formats, overcoming limitations of LinkedIn's export function
#published: false
---
<div style="text-align: center">

![Rock Star Developer Meme](https://github.com/iromanovsky/irom.info/assets/15823576/15eb18df-9c96-401d-b88d-14aa1ccb96a7)

</div>

LinkedIn's 'Easy Apply' feature streamlines job applications, enabling candidates to submit their profiles directly to recruiters. This bypasses the need for filling out additional forms or attaching separate resume files. Kudos to the employers who embrace this feature, as it helps them stand out in the competitive candidate-driven market.

However, the current employer-driven market often has different requirements, including the need for a detailed resume in a printable document format. This presents a challenge since LinkedIn's existing functionality restricts the export of profiles to PDFs with significant data loss and limited control over layout.

This post explores how you can use LinkedIn as your primary data source to craft adaptable resumes in both PDF and web page formats, leveraging open standards and automation for greater efficiency.

<!--more-->

## Step 1. Export your resume data to a standard format

[JSON Resume](https://jsonresume.org/) project aims to standardize resume data in JSON format. This serialized data becomes a versatile foundation for various tools.

Use [JSON Resume Exporter](https://chrome.google.com/webstore/detail/json-resume-exporter/caobgmmcpklomkcckaenhjlokpmfbdec) extension for Chrome (also works in Edge) to export your LinkedIn data into JSON Resume format.

There are alternative options to export LinkedIn data as well. One of them is [LinkedIn to Json Résumé](https://jmperezperez.com/linkedin-to-json-resume/) service which allows you to use [native LinkedIn export files](https://www.linkedin.com/help/linkedin/answer/a1339364/downloading-your-account-data).

For a more technical approach, the [LinkedIn API](https://learn.microsoft.com/en-us/linkedin/shared/integrations/people/profile-api) offers another possibility.

## Step 2. Check and amend your data if necessary

After exporting, ensure your data aligns with the [JSON Resume Schema](https://jsonresume.org/schema/). The [JSONResume Editor](https://marketplace.visualstudio.com/items?itemName=reflog.jsonresume) for VS Code can assist with syntax and, potentially, with preview and PDF generation.

Carefully review the exported data for personal or sensitive information, ensuring its accuracy and relevance.

## Step 3. Build you resume


Upload your JSON Resume data as a [Gist](https://gist.github.com/) named `resume.json`, like in [this example](https://gist.github.com/iromanovsky/1437d070ab6bc06164b9e2aed6ca7a39). Then, visit `https://registry.jsonresume.org/{your_github_username}` (e.g., [my resume](https://registry.jsonresume.org/iromanovsky)) to view it.

You may choose a theme by looking at previews [here](https://registry.jsonresume.org/themes) and adding the theme name as a parameter like `?theme=Resu` to the URL of your resume ([mine](https://registry.jsonresume.org/iromanovsky?theme=Resu) for example) to test the theme with your own data.

Take note that different themes may display or hide different sections of your JSON Resume data, and display it differently. When you are happy with the theme you selected, you can update the `meta` section of your `resume.json` gist with the parameter `"theme": "Theme-Name"`

Now you can use your browser to "print" the web page into PDF and you are done.

Alternatively, you can use JSON Resume format to import data to some visual tools such as [ResuMake](https://resumake.io/) or even build your own theme using existing [Themes](https://jsonresume.org/themes/) and [NPM packages](https://www.npmjs.com/search?ranking=maintenance&q=jsonresume-theme) as guidance, while using 'resume-cli' command like below to build your PDF:

```
resume export igor.pdf --resume ./my-resume.json -t ./node_modules/jsonresume-theme-full
```

## Step 4. Profit

You now have a PDF for job applications or a web page for hosting on platforms like GitHub Pages or Azure Storage.

## Alternatives

Consider these alternatives:

- [Reactive Resume](https://rxresu.me/), a free and open-source graphical resume builder, that works with JSON Resume data format
- Utilize static site generators like Jekyll with resume themes such as [online-cv](https://github.com/sharu725/online-cv).
- Use specialized services like [resume.io](https://resume.io/), which may be able to export data from Linkedin

## Discussion

As usual, I'm happy to discuss this topic with you in LinkedIn comments. If you want to support this post to get more exposure, please use [this link](https://www.linkedin.com/feed/update/urn:li:share:7132473822053564417/) to react, comment, or repost.
