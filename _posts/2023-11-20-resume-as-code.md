---
#layout: post-argon
title: Resume as Code
date: 2023-11-20 18:00:00 +0000
last_modified_at: 2023-11-21 09:30:00 +0000
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

The [JSON Resume](https://jsonresume.org/) project seeks to standardize resume data in JSON format, providing a foundation for various applications.

To export your LinkedIn profile into JSON Resume format, use the [JSON Resume Exporter](https://chrome.google.com/webstore/detail/json-resume-exporter/caobgmmcpklomkcckaenhjlokpmfbdec), a Chrome extension that also functions in Edge.

For alternatives, consider the [LinkedIn to Json Résumé](https://jmperezperez.com/linkedin-to-json-resume/) service, which utilizes [LinkedIn's native export files](https://www.linkedin.com/help/linkedin/answer/a1339364/downloading-your-account-data). 

Additionally, for those with a technical inclination, the [LinkedIn API](https://learn.microsoft.com/en-us/linkedin/shared/integrations/people/profile-api) provides another method for extracting your data.

## Step 2. Check and amend your data if necessary

Once you have exported your data, it's important to thoroughly review it for any personal or sensitive information. Ensure that all details are accurate and relevant to your purposes, amend if necessary.

For assistance with syntax validation according to the [JSON Resume Schema](https://jsonresume.org/schema/), consider using the [JSONResume Editor](https://marketplace.visualstudio.com/items?itemName=reflog.jsonresume) plugin for VS Code. This tool can also potentially help with previewing your resume and generating a PDF version.

## Step 3. Build you resume

Upload your JSON Resume data as a [gist](https://gist.github.com/) named exactly `resume.json`, similar to [this example](https://gist.github.com/iromanovsky/1437d070ab6bc06164b9e2aed6ca7a39). You can then view your resume by visiting `https://registry.jsonresume.org/{your_github_username}`, like [my own resume here](https://registry.jsonresume.org/iromanovsky).

For customization, explore different themes by previewing them [here](https://registry.jsonresume.org/themes). You can preview a theme with your data by appending its name to your resume URL, such as `?theme=Resu` ([see my themed resume preview](https://registry.jsonresume.org/iromanovsky?theme=Resu)). Keep in mind that various themes might alter the visibility and layout of your resume's sections, some of them honor rich formatting and markdown, and some do not. Once satisfied with a theme, update the `meta` section in your `resume.json` gist with `"theme": "Theme-Name"`.

To convert your resume into a PDF, simply use the print function in your browser.

Alternatively, you can use JSON Resume format to import data to some visual tools such as [ResuMake](https://resumake.io/), or even create your own theme using [available themes](https://jsonresume.org/themes/) and [NPM packages](https://www.npmjs.com/search?ranking=maintenance&q=jsonresume-theme) as guidance. Utilize the 'resume-cli' command as shown below to generate a PDF based on your custom theme:

```
resume export igor.pdf --resume ./my-resume.json -t ./node_modules/jsonresume-theme-full
```

## Step 4. Profit

You now have a versatile algorithm for the on-demand generation of PDFs suitable for job applications or web pages ready to be hosted on platforms like GitHub Pages or Azure Storage.

## Alternatives

For different approaches, consider these options:

- [Reactive Resume](https://rxresu.me/): A free, open-source graphical resume builder compatible with the JSON Resume data format.
- Static site generators: Tools like [Jekyll](https://jekyllrb.com/showcase/) offer resume themes, one notable example being [online-cv](https://github.com/sharu725/online-cv).
- Proprietary services: Platforms like [resume.io](https://resume.io/) might offer LinkedIn data export capabilities. However, they may involve costs and use proprietary standards, potentially limiting customization options.

## Discussion

As usual, I'm happy to discuss this topic with you in LinkedIn comments. If you want to support this post to get more exposure, please use [this link](https://www.linkedin.com/feed/update/urn:li:share:7132473822053564417/) to react, comment, or repost.
