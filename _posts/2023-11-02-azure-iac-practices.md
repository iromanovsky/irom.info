---
#layout: post-argon
title: General thoughts on IaC practices for Azure management
date: 2023-11-02 09:00:00 +0000
last_modified_at: 2023-11-02 10:00:00 +0000
author: Igor
categories: [Azure, Networking]
tags: [azure, iac, management, practices, automation, tools]
permalink: /blog/azure-iac-practices
slug: azure-iac-practices
excerpt_separator: <!--more-->
#redirect_from: [/post/azure-pe-name-resolution]
image: https://github.com/iromanovsky/irom.info/assets/15823576/ac2a60ea-0ab3-4823-a3d7-32f7a695bcb4
description: >-
    Exploring the depths of IaC practices for Azure, such as centralization, accelerators, need for automation, code structure, automated testing, and state management
published: false
---

In this post, I share my thoughts on established IaC practices for Azure management, often regarded as "best practices." However, they may benefit from a deeper, critical examination.

We will explore themes such as [Centralization and Democratization](#centralization-vs-democratization), [Solution accelerators](#solution-accelerators), treating [Infastructure as Software](#admins-vs-developers), the need to [Automate every task](#when-to-use-automation), [Modularization and version control](#code-structure), [Automated testing](#automated-testing), [State management](#terraform-state), and choosing the right [Configuration format](#yaml-vs-json).

Intended to evolve, this document will be periodically revised and expanded as I learn new insights and industry shifts occur.

<!--more-->

<div style="padding-left: 50%; font-style: normal">

> Watch what you say,  
> They'll be calling you a radical  
> A liberal,  
> Oh, fanatical, criminal  
> <br/>&mdash; ["The Logical Song"](https://youtu.be/kln_bIndDJg) by Supertramp

</div>

## Centralization vs. Democratization

The centralized approach to managing Azure cloud gained popularity as enterprises began their cloud adoption, often because they instinctively sought to replicate their existing approaches to IT management.

### Centralized cloud management

A reflection of this centralized mindset is the Cloud Center of Excellence (CCoE). Originally developed to establish best practices and standards across an organization, the CCoE has become synonymous with centralized cloud governance.

#### Pros:
- **Standardization**: Establishes consistent best practices across the organization.
- **Controlled Environment**: Easier to enforce security and compliance measures.

#### Cons:
- **Bottlenecks**: A centralized CCoE can inadvertently slow down processes. With numerous clients or departments relying on a singular entity for approvals, reviews, and implementation, delays become commonplace.
- **Overcomplication**: Central teams might set overly complex standards for others to follow. For instance, mandating automated and unit tests for every small task isn't always practical, especially for teams not well-versed in software development.

### Democratization

The other side of the coin is decentralized cloud management, which ties closely with the concept of democratization. In this model, various teams or departments have the freedom to manage their cloud resources, within certain bounds.

#### Pros:
- **Flexibility**: Teams can pick tools and processes aligning with their skill set and project needs, ensuring a smoother cloud adoption process.
- **Ownership and Care**: When a team manages its resources, they view those solutions as their "pets" rather than "cattle." The intrinsic sense of ownership often translates to better care and maintenance.
- **Accessibility**: This approach allows teams to embrace the cloud without heavy investments. Instead of being mandated to adopt full-fledged IaC, CI/CD, and comprehensive testing suites, teams can start with simpler templates and scripts, gradually progressing as their expertise grows. This ensures accessibility and humane treatment in cloud adoption.

#### Cons:
- **Lack of Standardization**: Without centralized governance, there's potential for disparate practices, which can lead to security and compliance challenges.
- **Increased Responsibility**: Democratization implies responsibility, necessitating proper education and training.

### Finding a balance

The challenge lies in finding the sweet spot between Centralized management and Democratization. A hybrid approach, where centralized guidelines establish the guardrails within which teams can operate with flexibility, might be the solution. 

It is a good idea to establish the CCoE for managing shared resources, defining standards and guardrails, and providing advisory while enabling individual teams to have the autonomy to choose tools and methods best suited for their tasks.

#### Further reading
- [Subscription Democratization](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/design-principles#subscription-democratization)
- [Operating model comparison](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/operating-model/compare#operating-model-comparison)


## Solution accelerators

Let's use a gastronomic analogy for navigating the world of solution accelerators.

- **Public Code Repositories**: Comparable to frozen pizzas. They're readily available and probably can feed you without damaging your health, but with the right culinary skills, you can make something more interesting based on them by adding the ingredients you like.
- **Proprietary Quickstart Accelerators**: Offered by IT integrators, are like the fast-food chains of the IT world. In McDonald's, you get predictable quality, but very little choice and customization possible.
- **Proprietary Advanced Accelerators**: Think of these as restaurants. More variety, and better quality, but at a premium. In an Italian restaurant, you probably get a good pizza (EPAM likes Terraform), and in an Austrian restaurant, you probably get a good schnitzel (MCS likes Bicep). So while an IT integrator might excel in one domain, it might not be the best fit for your specific needs (or tastes). _This option could be the right choice for your company if you do not have good experience in-house, but you need to keep in mind the price and limitations._
- **In-House Development**: Home-cooked meals can be delightful or disastrous, contingent on your expertise. For businesses deeply reliant on IT, in-house solutions, tailored precisely, might be the most cost-effective and efficient. _If your primary business depends on IT that much, you will probably do it better yourself, at much less costs._

In conclusion, selecting the right tool or solution for Azure management is crucial. The key is to assess your specific needs, weigh the pros and cons, and make an informed choice. As the [classic article from Joel on Software](https://www.joelonsoftware.com/2001/10/14/in-defense-of-not-invented-here-syndrome/) suggests, understanding your core requirements and capabilities can make all the difference.


## Admins vs. Developers

Azure infrastructure management is evolving from a traditionally admin-focused approach to one increasingly driven by developers. But what does this transition mean for organizations?

### Admins: keepers of stability

- **Stability Seekers**: Admins prioritize system stability. Their motto often is, "If it isn't broken, don't fix it."

- **Simplicity Advocates**: They try to find straightforward solutions to intricate challenges and prefer configurations over customizations. Too much abstraction can be counterproductive, leading admins to seek more tangible solutions closer to the platform itself, which can result in less ambiguity and a system that's easier to debug.

- **Silent Efficiency**: An efficient admin might appear inactive. But in reality, if they seem like they're doing nothing, they're doing their job right.

- **Maintenance Mindset**: Viewing cloud infrastructure like machines, admins emphasize maintenance and periodic upgrades. Their approach underlines the importance of robust architecture over fast-paced implementation, which can often lead to a stable and sustainable environment.

### Developers: restless innovators

- **Innovation Enthusiasts**: Drawn to new things, developers can sometimes prioritize excessive automation over robust architecture, which can complicate rather than simplify systems.

- **Abstract Thinkers**: Developers lean towards creating abstractions, often resulting in more complex systems. This developer-centric approach can increase the complexity and steepen the learning curve for newcomers who need to familiarize themselves with an array of new tools and concepts.

- **Busy Bees**: Developers typically earn higher salaries and may be mandated to fill timesheets. This can inadvertently push them to always appear busy, even if it means overcomplicating tasks.

- **Software Perspective**: Developers see cloud infrastructure as software. Their approaches, while suitable for software, might not always align with the nuances of shared infrastructure, which could not be easily redeployed from scratch or swapped with a new instance if things go wrong.

- **Legacy Creation Risks**: When developers treat infrastructure management as a software product, there's a risk that their creations may become unsupported legacy systems. These can become obstacles to evolution, potentially necessitating migration to newer tools or methodologies down the line. In contrast, utilizing native tools and keeping things simple from the outset can offer a smoother evolution path.

### Navigating the middle path

In Azure management, as with many things, the middle ground often yields the best results. 

A balanced approach that emphasizes the value of spending more time crafting a robust architecture rather than rushing into implementations and programming excessive automation can lead to an infrastructure that is both resilient and capable of evolving. 

This deliberate focus on architectural integrity over rapid development ensures that the systems built today will remain relevant and adaptable for the challenges of tomorrow. 

Embracing both the administrative focus on stability and simplicity, alongside the developers' drive for innovation and automation, fosters an environment that supports an easier onboarding process for new team members and avoids the pitfalls of creating inflexible systems that could become more of an obstacle than a help in the future.


## When to use automation

Automation in Azure can drive efficiency and consistency, but it's crucial to discern where its application is truly beneficial. For one-off tasks or unique infrastructure components like ExpressRoute, which you're unlikely to redeploy automatically, the automation investment may not always make sense.

### Why automate?

- **Consistency**: Automation is key to maintaining uniformity, particularly for tasks that are repeated regularly.
- **Scalability**: It enables easier scaling of operations across the board.
- **Time-saving**: It can greatly reduce the time spent on routine tasks.

### When not to automate?

- **Unique and Rare Tasks**: For example, configuring ExpressRoute typically happens once; automating this process might turn five minutes on the portal into days of unnecessary coding.
- **Complex Judgment Calls**: Certain tasks, such as firewall configurations with dynamic rules, require nuanced human judgment that automation can't replicate.
- **High Customization**: Unique infrastructure setups with specific requirements might not be well-served by a one-size-fits-all automation script.

### Finding a balance

- **Assessment**: It's important to evaluate the frequency and complexity of tasks before deciding to automate.
- **Selective Prioritization**: Focus on automating tasks that are performed frequently and are prone to human error.
- **Ongoing Review**: Regularly check that automated processes are still serving their intended purpose efficiently.

Restricting access to the Azure portal with the intent to automate everything can be counterproductive. Especially in cases of networking where immediate action might be required, portal management can be invaluable. Instead of completely blocking portal access, it's more practical to use it alongside automation tools for a more flexible approach.

For typical deployments that do benefit from automation, such as repeatable deployments of complex VMs, clusters, or self-contained solutions, sophisticated templates can be utilized. And for crafting templates for simpler solutions, tools like ChatGPT can be a resourceful aid.

In conclusion, while automation is a powerful tool, it's not a panacea. A balanced, thoughtful approach that leverages both manual and automated processes is often the most effective strategy.


## Code structure

How you structure your code affects many aspects of how well IaC may work for you. Here's a more detailed look at how to approach this:

### Modularity

Modularity is key in IaC. It promotes code reuse through the DRY (Don't Repeat Yourself) principle. However, creating excessive abstractions can lead to complexity, so it’s essential to balance reuse with the AHA (Avoid Hasty Abstractions) principle. Additionally, modules facilitate the separation of concerns, allowing teams to focus on specific parts of the infrastructure without being overwhelmed by unrelated details.

### Monorepo vs Multirepo

The choice between a monolithic repository (monorepo) and multiple repositories (multirepo) depends on several factors:

- **Multirepo Pros**: Using a multirepo approach, where each module has its own repository, simplifies versioning and referencing through tags. 
- **Multirepo Cons**: However, it can be cumbersome to clone, synchronize, and make changes across multiple repositories, and tracking changes separately can become complex.

- **Monorepo Pros**: A monorepo, in contrast, centralizes code, which simplifies dependency tracking and coordination across modules.
- **Monorepo Cons**: Yet, it can lead to challenges with managing permissions and quality gate chains.

My recommended strategy is to minimize the number of repositories used as much as possible while still maintaining a clear separation of concerns. For example, centralize your core modules in one repository, but keep configuration code in distinct repositories organized by team, region, project, or concern. Implement module versioning with file name prefixes or reference specific commits or tags.

### File structure

Organize your repositories using folders and files to categorize components, groups, solutions, subscriptions, or applications. Avoid the pitfall of using single files for multiple loosely related resources, as they are prone to merge conflicts and make code reviews more difficult. Large files with multiple resources are challenging to navigate, increasing the risk of errors, and begging for conflicts when changes are made.

When managing resources, aim:
- **When adding**: Introduce new resources or logical groups by creating new configuration files.
- **When modifying**: When a resource requires changes, edit the specific file where it’s defined.
- **When deleting**: If a resource or group is no longer needed, remove the corresponding files or folders.

This approach minimizes conflicts, simplifies tracking changes, and eases the process of reviewing pull requests. It also aligns with best practices for quality assurance, as changes are isolated and can be managed independently, avoiding the delays that come with resolving merge conflicts and rebasing.

In essence, a well-structured IaC setup respects the separation of concerns, minimizes merge conflicts, and facilitates both maintenance and scalability.


## Automated Testing

When talking about Infrastructure as Code (IaC), automated testing becomes a frequent discussion point. Let's delve deeper into its significance and limitations.

### The Case for automated testing for IaC

Automated testing is often named as a best practice for IaC, and for good reasons:
1. **Consistency**: aims to ensure that the infrastructure is consistently deployed every time.
2. **Error Reduction**: aims to catch mistakes that might have been missed during manual reviews as early as possible ("shift-left").
3. **Efficiency**: aims to Speed up the deployment process by reducing manual validation steps.

### Realities of automated testing for IaC

While automated testing is valuable, there are constraints to recognize:

- **Platform Limitations**: Is it even feasible to test every API capability more thoroughly than the platform itself? Attempting to outdo the platform's testing is like reinventing the wheel. Instead, trusting the platform's "what-if" deployments for sanity checks can be more practical. These deployments offer predictions and validations built upon the platform's intimate knowledge of its components.

- **Complex Dependencies**: Some resources, notably in networking, have intricate interdependencies and transient states. This complexity means they can't be effectively tested outside of their platforms, barring general syntax and sanity checks.

- **Dynamic Limits**: The value of unit tests for parameters like naming and resource limits can be questionable. Resource attributes and their limits can change over time. Trying to keep up feels like a race where you're perpetually trailing.

- **Naming Conventions**: Issues like naming should be addressed organizationally. Establishing a clear naming protocol can obviate the need for numerous tests.

- **Local Structure Testing**: Unit tests can be beneficial when you're checking local structures, especially if employing intricate solutions like Python objects in tools like Pulumi or when generating ARM/Bicep/TF templates from your code. However, this only applies to your internal structures and not external platform specifics.

### Limits of automated testing

In essence, while automated and unit testing can effectively validate the logic and structures of your solutions, they might fall short when it comes to platform-specific checks. Tools like "what-if" deployments inherently have a deeper understanding and integration with their platforms, making them better equipped for such tasks.

### Embracing proper testing environments

Instead of pursuing testing platform-specific nuances, it could be wiser to allocate resources to proper test environments. These environments let you trial actual deployments, templates, and solutions safely before migrating them to production. It's a real-world evaluation, offering insights that no mock test can replicate.

### Simpler path with native management tools

If you've followed my prior advice on using the platform's native management tools where possible, opting for declarative templates, and refraining from unnecessary abstractions, you might find a reduced need for intricate automated and unit tests. The reason is simple: much of what requires testing is already catered to by "what-if" deployments. If you did not implement your own complex code, functions, or abstractions, there is nothing to test.


## Terraform state

### Do you really need TF state?

While seasoned Terraform users understand the importance and use cases of the Terraform state file, I often see that less experienced users are just doing the same thing that they think everybody else is doing, like  in "cargo cult".

HashiCorp explains why Terraform needs a state file in the article [Purpose of Terraform State](https://developer.hashicorp.com/terraform/language/state/purpose). There are many letters in this article, but it boils down to the fact that Terraform design itself requires to use the state file, there is no evidence to support the reverse logic I often hear: _"You need a state file to manage Azure, and that's why you need to use Terraform"_.

<details style="padding-left: 3ex; padding-bottom: 1em">
<summary>TLDR summary from ChatGPT [you are welcome]</summary><br/>

The article on HashiCorp's website discusses the purpose and importance of the state in Terraform, which is a tool for building, changing, and versioning infrastructure safely and efficiently.

Here's a quick summary of the key points:

- **Terraform State**: Terraform keeps track of the infrastructure it manages through a file called the "state." This state file holds information about the resources Terraform created and the configuration it's based on.

- **Purpose of State**: The state serves several roles:
  - Mapping from Terraform configurations to real-world resources.
  - Storing metadata such as resource dependencies.
  - Performance optimization by caching attribute values.
  - Synchronization for Terraform actions if used in a team environment.

- **State Storage**: By default, the state is stored locally, but for team environments, it can (and should) be stored remotely.

- **Importance for Terraform Operations**: Terraform relies on the state file to determine what Azure resources to create, update, or delete.

- **Risk of Out-of-Sync State**: If the state doesn't match the real-world resources, Terraform might perform the wrong actions.

- **Handling Sensitive Data**: State files can contain sensitive data, so it's important to handle them securely, especially when stored remotely.

In conclusion, the state in Terraform is a critical component that allows Terraform to function correctly and efficiently. It must be managed carefully to ensure that it reflects the actual state of the infrastructure and is kept secure due to the sensitive data it may contain.
</details>

Azure, at the same time, maintains its own state of resources. When using IaC repositories, your version control system (Like Azure DevOps or GitHub) can track changes, and `what-if` functionality of ARM or Bicep template deployments can predict the plan changes similar to `terraform plan`. 

So, let's explore where the Terraform state may be really required.

### Self-Service Model: Deploying for Others

Terraform shines in environments where resources are consistently managed. 

However, when resources are provisioned on request by some central team and subsequently handed off to another team (self-service model), they are not centrally managed anymore, so why keep them in the central code base and state file? Complications arise:

- **State Conflicts**: If a resource registered in one team's state file is later managed by another, conflicts emerge. It means that you should implement a process to "forget" such resources after you deploy them.
- **Delegation Dilemmas**: Transferring or co-managing state files between teams is rarely possible, as it might contain other resources that shouldn't be managed by the other team. It is usually implied that deployments that are using the same state files need to be executed by the same identity to avoid inconsistent deployments caused by possibly different RBAC privileges of different identities.

Given these challenges, there are two common strategies:
1. Declare the resource as orphaned in the initial state file before passing it on.
2. Directly provision resources in the intended team's state file.

However, these workarounds underscore that Terraform might not be the ideal choice for self-service deployments, since the benefits of state files are not used, while only creating challenges.

### Managed Service Model: Owning the Lifecycle

Terraform is exceptionally adept when there's a need to oversee the entire lifecycle of resources—from creation to deletion and even full redeployment. 

While it's a good fit for software solutions deployed on Platform as a Service (PaaS), it might not be suitable for shared infrastructure that cannot be easily redeployed. 

For instance, completely redeploying stateful resources like domain-joined VMs will lead to domain detachment and data on disks will be lost.

### Democratization Model: Shared Resource Management

Co-management, where resources are jointly overseen by the deploying team and the application owner, can be tricky. An example is deploying a virtual network, which an application owner can further modify, like adding subnets. This shared management will lead to state conflicts.

### Resolving State Conflicts

What if an object needs to be altered outside of Terraform, either manually, by another tool, or by enforcing a policy? In such cases, somebody must import the existing state from Azure, ensuring that Terraform's state aligns with Azure's actual state. 

Common objects requiring management outside of TF include role-based access controls, policy assignments, exclusions, and exemptions. This makes managing such resources in TF not so practical.

### Terraform State vs. Native Tools: The Verdict

For infrastructure deployments that evolve over time, ARM, Bicep, and PowerShell scripts offer a balanced approach. 

Terraform state, on the other hand, excels for deploying complete products when there's a need for consistent redeployment from scratch.


## YAML vs JSON

YAML and JSON are both popular data serialization formats, but each brings its flair to the table. Here's a concise breakdown.

### YAML: Designed for Humans

- **Whitespace Matters**: Spaces in YAML denote structure, promoting visual clarity but also potential indentation errors.
  
- **Intuitive Syntax**: YAML's simple syntax and plain English terms, like `true` and `false`, make it user-friendly.


### JSON: Tailored for Machines

- **Clear Structure**: JSON’s curly braces (`{}`) and square brackets (`[]`) ensure machine-friendly parsing.

- **Widespread Use**: Born from JavaScript, JSON is now a universal web standard, making it ideal for APIs and data transfers.

### How to choose

Although JSON is technically a YAML subset, preferences divide due to:

- **Design Intent**: YAML prioritizes human readability, while JSON focuses on machine precision.
  
- **Error Susceptibility**: YAML can fail with wrong indentations, and JSON can break with a missing comma or brace.

- **Tool Adoption**: Tools like Kubernetes lean towards YAML, whereas web APIs often favor JSON.

### YAML vs JSON: The winner is?

In the end, the choice between YAML and JSON often boils down to the application and personal preference. 

- **YAML** excels in human-centric configurations,
  
- **JSON** shines in programmatic scenarios and tools like PowerShell.

As with most technology choices, it's not about which is universally better, but which is more appropriate for the task at hand.

## Discussion

As usual, I'm happy to discuss this topic with you in LinkedIn comments. If you want to support this post to get more exposure, please use this link to react, comment, or repost.
