---
#layout: post-argon
title: Choosing the right Level of Infra as Code management tools for Azure
date: 2023-09-29 17:00:00 +0100
#last_modified_at: 2023-09-26 16:00:00 +0100
author: Igor
categories: [Azure, Networking]
tags: [azure, devops, caf]
permalink: /blog/azure-iac-levels
slug: azure-iac-levels
excerpt_separator: <!--more-->
#redirect_from: [/post/azure-pe-name-resolution]
image: https://github.com/iromanovsky/irom.info/assets/15823576/ac2a60ea-0ab3-4823-a3d7-32f7a695bcb4
description: >-
    There are various tools IaC management at different levels of complexity. Let's explore these and determine which level is the best fit for your requirements.

#published: false
---
<div style="text-align: center">

![tree-swing-what-customer-wanted](https://github.com/iromanovsky/irom.info/assets/15823576/ac2a60ea-0ab3-4823-a3d7-32f7a695bcb4)

</div>

When selecting IaC management tools for Azure, it's essential to consider your specific needs and preferences. There are various tools and approaches at different levels of complexity. Let's explore these options and determine which level best fits your requirements.

<!--more-->

## Level 0

### Azure Portal

IaC enthusiasts often underestimate the Azure Portal, but it has its merits. Here's when it shines:

Good for:

- **Education**: the portal provides an intuitive way to grasp Azure concepts, making it accessible to a broad audience.
- **Everyday observation in read-only mode**: when you need to view resources quickly or monitor your Azure environment
- **Troubleshooting**:iIt's an irreplaceable tool for diagnosing issues and finding solutions, especially for Networking
- **Emergency changes**: in urgent situations, it allows for quick manual interventions, saving time when IaC automation isn't feasible.

Bad for:

- **Routine change operations**: while the portal is great for emergencies, it's not the ideal choice for routine or repetitive tasks.
- **Keeping things in order**


### Azure Resource Manager REST API

Azure Resource Manager REST API
The Azure API is structured according to industry REST standards and is a powerful tool, though not for everyday use.

Use for:

- **When command line tools are unavailable**: in cases where command line tools have limitations or are unavailable, the REST API comes to help.
- **Building custom solutions**: If you're creating your own solution or tool, the REST API is your way to directly interact with the management plane.

Too Complex for:

- **General daily usage**: Due to its complexity and low-level nature, the REST API isn't suitable for typical daily tasks, unless you wrap it into your own script snippets.

Hints:

- **Browser Dev Tools (F12)**: when higher-level management tools lack specific functionality that the portal offers, you can inspect the REST API calls made by the portal in your browser's developer tools. This insight can be valuable for building custom solutions.
- **Pure PowerShell**: PowerShell is a robust choice for interacting with the REST API due to its powerful object processing capabilities and built-in support for REST calls and JSON.


## Level 2 - Scripting tools

### Azure CLI

The Azure CLI is frequently featured in educational materials for its apparent simplicity. 

However, it's essential to consider its nuances:

- Within the az interface, the arrangement of commands and parameters can vary based on the context. The extensive use of defaults and aliases can make it less reader-friendly, particularly for those not used to the spirit of this kind of tool.
- It is primarily designed keeping Linux shell in mind (though you can use PowerShell or even CMD), leveraging advanced text and array processing often necessitates the use of Linux utilities like grep, sed, awk, and others. These tools, while powerful, are quite nerdy and inconsistent in their syntax

Good for:

- **Simple, quick commands**: for straightforward, fast operations.
- **Nerds**: with a background in open-source and hardcore networking (such as Cisco CLI), some people may feel at home with this tool.

Bad for:

- **Complex scripting**: when it comes to more intricate scripts or automation, the CLI's inconsistent syntax and reliance on Linux utilities can become challenging.
- **Cross-platform scripting**: for scripts intended to work seamlessly across different platforms.
- **Educational purposes and exams**: if complexity gets beyond basic commands, this makes it less suitable for educational environments.

### Azure Powershell

> **Disclaimer**: I love PowerShell from the beginning. Since it become cross-platform, I love it even more.

It's a great tool for parsing strings and working with arrays, collections, and objects. This makes it very good for processing JSON and general work with REST APIs.

Good for:

- **Heavy Scripting**: Az module with the power of PowerShell excels in scripting complex tasks
- **Cross-Platform Scripting**: for building scripts that work across different operating systems
- **Education**: due to its consistent and well-thought-out syntax.

Bad for:

- **YAML Processing**: if your work heavily involves YAML-based configurations, other tools might be more suitable.
- **Religious or philosophical reasons**: may not align with specific beliefs or strong preferences coming from an OSS background.


## Level 2 - Native declarative templates

Native declarative templates enable you to define resources in a declarative manner, focusing on describing the desired end state. The template processing engine then takes care of deploying the resources in the correct order, while maintaining idempotency.

When using native Microsoft tools, you gain early access to the latest features and reduce the likelihood of errors introduced by translating Microsoft's ARM API into higher abstraction layers, as is the case with Terraform.

### ARM templates

I've already mentioned the ARM REST API, right? Policies, event logs, error messages, interactions in the portal and CLI, and even the internal definitions of Azure resources – they're all in JSON.

Well, ARM templates come incredibly close to the ARM API because they're based on JSON format. This is what makes ARM templates the most precise and the least abstracting tool for IaC while maintaining a declarative approach.

Some people don't like JSON for ugly presentation when unformatted, and low tolerance for errors with commas. ARM Template [expressions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/template-expressions) and [fucntions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions) may look too complex and fragile until you get used to them.

One significant benefit of ARM templates is their ability to closely mirror the API. You can refer to API representations of resources (JSON View, Error Messages, CLI Debug) as a reference while creating templates. This minimizes the chances of errors in attribute names and the overall object structure.

Additionally, ARM templates often offer early access to new API features, giving you an advantage over tools like Terraform.

Good for:

- Deploying complex, comprehensive infrastructure solutions with multiple resource types and intricate dependencies, such as network hubs or network virtual appliances.
- Embracing new features not yet available in higher-level infrastructure-as-code (IaC) tools.
- Use cases where no alternatives exist, such as defining policies, blueprints, or Azure Marketplace offers.

Challenging for:

- Adopting a modular approach, especially when you intend to maintain templates as building blocks or modules for your customers to employ in constructing their systems.
- Seeking greater accessibility for both authoring and usage among a broader audience.


### Bicep templates

Bicep is an abstraction layer built on top of ARM templates, designed to simplify syntax, improve readability, reduce errors, and introduce [additional functionality](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview?tabs=bicep#benefits-of-bicep). While Bicep templates use YAML syntax for readability, they are internally compiled into JSON for Azure deployment.

Benefits of Bicep Templates Include:

- **Simplified syntax**: Bicep makes the template syntax more human-readable, enhancing the overall developer experience and reducing the likelihood of syntax-related errors.
- **Functionality enhancements**: Bicep introduces functionality that simplifies common tasks and improves usability, making it easier to work with templates as modules.

Challenges of Bicep Templates Include:

- **YAML to JSON compilation**: Although Bicep uses YAML for a more accessible syntax, it is internally [compiled into JSON](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/frequently-asked-questions#what-happens-to-my-existing-arm-templates) for interactions with ARM API. This abstraction can complicate troubleshooting, as your configuration source is in Bicep/YAML, while Azure interactions and messages remain in JSON.

Good for:

- Creating a catalog of centrally maintained modules for use by end-users, ideal for Cloud Center of Excellence (CCoE) scenarios.
- Simplifying the definition of typical resources for better accessibility by a broader audience.

Less suitable for:

- Scenarios where fine-grained control, closely resembling API interactions, is crucial. This might be important for preview resources or highly complex deployments, where the translation from Bicep to JSON complicates troubleshooting.

## Level 3 - Terraform and other "cloud-agnostic" tools 

Many customers are convinced that adopting a multi-cloud strategy and choosing Terraform as a single, "cloud-agnostic" solution for IaC management can streamline their efforts.

### Language

However, the devil is in the details. While Terraform is designed to be cloud-agnostic, this concept primarily applies within the Terraform language itself. Each cloud provider has its own unique set of resources and services. Terraform relies on abstract "providers" for each cloud, responsible for translating Terraform configurations into API calls specific to that cloud provider.

So, even though you can utilize a unified language and configuration syntax across multiple cloud providers, you'll still need to understand the intricacies and capabilities of each provider when writing Terraform code. Additionally, advanced features or services offered by a particular cloud provider may not be immediately supported by Terraform, necessitating either patience for Terraform updates or active contributions to its development to accommodate those features (this is not a joke).

### State tracking

The second challenge is the **state file**. While many consider the ability to track state a significant advantage of Terraform, there are cons in specific scenarios. I will delve into the topic of state files in detail in a separate post, but here is a high-level overview:

- The state file can hinder scenarios involving multiple users. This can be the case when various teams and policies need to manage shared infrastructure resources from different accounts.
- Azure infrastructure may not necessarily require external state tracking, as resource states are stored in Azure, and changes are tracked in the IaC repository.
- Shared infrastructure resources rarely require deletion, and the scenario of redeploying an entire system, like a network hub, is rarely feasible or desirable.
- There are alternatives to Terraform's "plan" function available in native template tools.
- Concerns regarding security (especially handling secrets), consistency (the need to reconcile external changes), locking, scalability, and performance can arise.

### Cargo cult and Profits

Lastly, there might be philosophical considerations beneath the technical surface. Some individuals may have an almost religious belief in universal Terraform applicability for any IaC, perhaps because they haven't thoroughly explored alternative options or critically evaluated Terraform's drawbacks. 

On the other hand, certain organizations, especially service providers, might find it more profitable to have their engineering teams specialize in a single IaC tool, providing complex IaC solutions based on Terraform that could demand more effort to maintain them.

### Do's and Don'ts

Terraform is well-suited for:
- Deploying software products that:
 - require external state tracking and support deletion and complete redeployment,
 - can and should be managed by a single team,
 - of software developers who are proficient in using Terraform.

Terraform may not be the best choice for:
- Deploying landing zones and other shared infrastructure that:
  - require management or modifications by multiple principals, including Azure policies,
  - don't need routine deletions and complete redeployments,
  - and managed by a team dedicated to specific cloud rather than general software development

## Level 4 - Pulumi and other "language-agnostic" tools

We are launching into space here, my dear readers.

[Pulumi](https://github.com/pulumi/pulumi) adds one more abstraction, which redefines IaC "Tool" to IaC "Software". Now you can use the language of your choice to manage your cloud.

This pushes the pros and cons of the previous concept to the next level.

Pulumi is suitable when:
- you have a team of Software Engineers specializing in specific languages,
- and management of the cloud is (part of) their software product


## Conclusion

Use the right tool for the right job.

- Use portal and self-service tools for simple mortal end-users
- Use scripts for admin tasks
- Use native templates for infrastructure management
- Use state tracking "meta"-tools for individual teams and solutions, if they know how to use it

 Kind regards,  
 Your Captain Obvious

## Discussion

Happy to discuss and learn new things from you. 
