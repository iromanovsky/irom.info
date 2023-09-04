---
#layout: post-argon
title:  Diagrams as Code
date:   2023-09-05 09:00:00 +0100
author: Igor
categories: Blog
tags: [azure, devops, caf]
#permalink: /post1/
slug: diagrams-as-code
excerpt_separator: <!--more-->
#redirect_from: [/lorem, /post/lorem]
#published: false
---

![mermaid-diagram-2023-09-04-232438](https://github.com/iromanovsky/irom.info/assets/15823576/480e2b26-a85a-40d7-b7b9-bf0b1f0e7743)

Did you know that you can do Diagrams as Code? Read this to know how to make a machine-generated diagram of Azure Management Groups structure in a declarative way.

<!--more-->

I never liked to draw. But as a cloud architect, I have to do it regularly. Computers are helping to make my curly lines straight, however, I tend to spend too much time adjusting the objects until I have everything aligned perfectly or until I lose my interest.

This is why I always liked the idea of declarative drawing. I just declare what I want to see, and the machine draws this automatically.

One of the most common candidates for drawing automation in my everyday job is creating diagrams of Azure Management Groups structure.

With the use of Mermaid, you can create machine-generated diagrams from the declarative, simple description, and it gets rendered automatically on Azure DevOps and GitHub markdown.


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

