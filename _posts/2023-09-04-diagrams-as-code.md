---
#layout: post-argon
title:  Diagrams as Code
date:   2023-09-04 20:00:00 +0100
author: Igor
categories: Blog
tags: [azure, devops, caf]
#permalink: /post1/
slug: diagrams-as-code
excerpt_separator: <!--more-->
#redirect_from: [/lorem, /post/lorem]
image: https://github.com/iromanovsky/irom.info/assets/15823576/49ec0de1-7953-4513-8d7b-6090d39bfdc0
#published: false
---
<div align="center">  

![mermaid-diagram-2023-09-04-232438](https://github.com/iromanovsky/irom.info/assets/15823576/480e2b26-a85a-40d7-b7b9-bf0b1f0e7743)

</div>

Did you know that you can do Diagrams as Code? In this post, discover how to generate machine-made diagrams of Azure Management Groups structure using a declarative approach.

<!--more-->

I never liked to draw, but as a cloud architect, drawing is a regular part of my job. Computers have certainly made my lines neater, but I still find myself spending too much time tweaking objects for that perfect alignment or losing interest altogether.

This is why I always liked the idea of declarative drawing.  With this approach, I can simply state what I want to see, and the machine takes care of the rest.

One of the most common candidates for drawing automation in my everyday job is creating diagrams of Azure Management Groups structure.

With the use of Mermaid, you can create machine-generated diagrams from simple declarative descriptions, and it gets rendered automatically from markdown on [Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/project/wiki/wiki-markdown-guidance?view=azure-devops#add-mermaid-diagrams-to-a-wiki-page) and [GitHub](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams).


[Mermaid Documentation](https://mermaid.js.org/intro/n00b-gettingStarted.html) is well organized with useful examples for some common diagram scenarios. 

[Mermaid Live Editor](https://mermaid.live/) allows you to generate the diagrams on the fly.

[ChatGPT](https://chat.openai.com/) may help you to get started with this prompt:

> build mermaid diagram for reference CAF azure management group structure

Here is my example declarative definition that was used to generate the picture in this post:

```
::: mermaid
graph TD

subgraph "Tenant Level"
    RMG[Root Management Group]
end

subgraph "Organization Level"
    ORG[Contoso]
end

subgraph "Domain Level"
    PL[Platform]
    BS[Business]
    SB[Sandbox]
    DC[Decom]
end

subgraph "Service Level"
    PM[Management]
    PC[Connectivity]
    PI[Identity]
    WSS[Shared Services]
    WA[AVD]
    WS[SAP]
    LZ[Landing Zones]
end

subgraph "Environment Level"
    PRD[Production]
    QA[QA]
    TST[Test]
    DEV[Dev]
end


RMG --> ORG

ORG --> PL
ORG --> BS
ORG --> SB
ORG --> DC

PL --> PM
PL --> PC
PL --> PI

BS --> WSS
BS --> WA
BS --> WS
BS --> LZ

LZ --> PRD
LZ --> QA
LZ --> TST
LZ --> DEV
:::
```

You're welcome.
