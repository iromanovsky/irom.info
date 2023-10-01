---
#layout: post-argon
title: Choosing the right level of IaC management tools for Azure
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
    Explore Azure IaC tools like CLI, ARM, Bicep, Terraform & Pulumi. Understand their abstraction levels & find the best fit for your needs.

#published: false
---
<div style="text-align: center">

![tree-swing-what-customer-wanted](https://github.com/iromanovsky/irom.info/assets/15823576/ac2a60ea-0ab3-4823-a3d7-32f7a695bcb4)

</div>

Tools such as CLI & PowerShell, ARM & Bicep templates, Terraform & Pulumi -- are serving as management interfaces between your code and the Azure API, and, in the absence of a better term, all called just "Infa as Code management tools".

When choosing such a tool, it's important to make an informed decision based on your actual needs and capabilities. Various IaC tools span different levels of complexity and abstraction. Let's explore these options and determine which level best fits your situation.

My thoughts on general IaC practices, such as version control, code reuse, CI/CD, and testing will follow separately.

<!--more-->

## Level 0: Direct Interaction Tools

On this level, you "push the button to get the result". All reasoning and sequence of actions is up to you.

### Azure Portal

IaC enthusiasts often underestimate the Azure Portal, but it has its merits. Here's where it shines:

Good for:

- **Education**: user-friendly interface enabling understanding Azure concepts for a broader audience
- **Read-only observation**: efficient for quickly examining resources or overseeing the Azure environment.
- **Troubleshooting**: vital for diagnosing issues, particularly in networking.
- **Emergency changes**: useful for immediate manual actions when automation is impractical.

Bad for:

- **Routine change operations**: not ideal for recurrent tasks.
- **Keeping things in order** we are here for IaC, right?


### Azure Resource Manager REST API

The Azure API is structured according to industry REST standards and is a powerful tool, though not for everyday use.

Good for:

- **When command line tools are unavailable**: in cases where command line tools have limitations or are unavailable, the REST API comes to help.
- **Building custom solutions**: If you're creating your own solution or tool, the REST API is your way to directly interact with the management plane.

Too Complex for:

- **General daily usage**: Due to its complexity and low-level nature, the REST API isn't suitable for typical daily tasks, unless you wrap it into your own script snippets.

Hints:

- **Browser Dev Tools (F12)**: when higher-level management tools lack specific functionality that is suddenly available on the portal, you can inspect the REST API calls made by the portal in your browser's developer tools. This insight can be valuable for building custom solutions.
- **Pure PowerShell**: PowerShell is a proven choice for interacting with the REST API due to its powerful object processing capabilities and built-in support for REST calls and JSON.


## Level 1: Scripting tools

Scripting tools grant you an interface to the Azure API, facilitating the scripting of actions. This promotes the creation of more repeatable, precise, and complex scenarios.


### Azure CLI

The Azure CLI is frequently featured in educational materials for its apparent simplicity. 

However, it's essential to consider its nuances:
- Within the az interface, the position of commands and parameters can fluctuate based on the context. An extensive reliance on defaults and aliases may hinder readability, especially for those unfamiliar with such tooling conventions. For instance, learning the command structure for `acr repository` doesn't simplify understanding contexts like `acr run` and `aks`:
```
# list images in container registry "irominfo"
az acr repository list --name irominfo

# on 'irominfo' registry, run command 'jekyll --version' 
az acr run --registry irominfo --cmd '$Registry/test/jekyll jekyll --version' /dev/null

# on 'irominfo' registry, in ACL, allow access from specified cluster
az aks update -n ClusterName-aks -g GroupName-rg --attach-acr irominfo
```
- Because Azure CLI is tailored for the Linux shell (however it can be used in PowerShell or CMD), if you go for advanced text and array processing, you'll often need Linux utilities like `grep`, `sed`, and `awk`. These tools, while being powerful, might be perceived as esoteric and have an inconsistent syntax. For example, [courtesy of Cloudtrooper](https://blog.cloudtrooper.net/2023/05/02/you-want-to-use-as-path-as-your-virtual-hub-routing-preference/), the below codes is used:

```
az network vhub get-effective-routes --resource-type RouteTable \
--resource-id $rt_id -g $rg -n hub1 --query \
'value[].{Prefix:addressPrefixes[0], ASPath:asPath, \
NextHopType:nextHopType, NextHop:nextHops[0], \
Origin:route.Origin} ' -o table | awk \
'{ gsub(/\/subscriptions\/'$subscription_id'\/resourceGroups\/'$rg'\
/providers\/Microsoft.Network\//,""); \
print }'
   
# ... just to display these columns:
| Prefix | ASPath | NextHopType | NextHop | Origin |
```

Good for:

- **Simple, quick commands**:  for quick, straightforward operations..
- **Nerds**: with a background in open-source and hardcore networking (such as Cisco CLI), some people may feel at home with this tool.

Bad for:

- **Complex scripting**: the CLI's fluctuating syntax and dependence on Linux utilities can be challenging.
- **Cross-platform scripting**: for scripts intended to work seamlessly across various platforms.
- **Educational purposes and exams**: if complexity gets beyond basic commands, this makes it less suitable for educational environments.

### Azure PowerShell

> **Disclaimer**: I love PowerShell from the beginning. Since it become cross-platform, I love it even more.

PowerShell excels at string parsing and working with arrays, collections, and objects. This makes it very good for processing JSON and interfacing with REST APIs.

Az module for PowerShell gives you the necessary commands to manage most of the Azure platform in a consistent, object-oriented way.

Good for:

- **Heavy scripting**: leverages PowerShell's capabilities for intricate tasks.
- **Cross-platform scripting**: provides consistent scripting experience across operating systems.
- **Education**: due to its coherent and intuitive syntax

Bad for:

- **YAML processing**: if your work heavily involves YAML-based configurations, other tools might be more suitable.
- **Religious or philosophical reasons**: may not align with specific beliefs or [strong preferences](https://en.wikipedia.org/wiki/Not_invented_here) coming from an OSS background.


## Level 2 - Native declarative templates

Native declarative templates enable you to define resources in a declarative manner, focusing on describing the desired end state. The template processing engine then takes care of deploying the resources in the correct order, while maintaining idempotency.

When using native Microsoft tools, you gain early access to the latest features and reduce the likelihood of errors introduced by translating Azure API into higher abstraction layers, as is the [case with Terraform](#tf-private-endpoint).

### ARM templates

I've already mentioned the ARM REST API, right? Policies, event logs, error messages, interactions in the portal and CLI, and even the internal definitions of Azure resources – they're all in JSON.

Well, ARM templates come incredibly close to the Azure API because they're based on JSON format. This is what makes ARM templates the most precise and the least abstracting tool for IaC while maintaining a declarative approach.

Some people don't like JSON for ugly presentation when unformatted, and low tolerance for errors with commas. ARM Template [expressions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/template-expressions) and [fucntions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions) may look too complex and fragile until you get used to them.

The significant benefit of ARM templates is their ability to closely follow the Azure API. You can refer to API representations of resources (JSON View, Error Messages, CLI Debug) as a reference while creating templates. This minimizes the chances of errors in attribute names and the overall object structure.

Additionally, ARM templates often offer early access to new API features, giving you an advantage over tools like Terraform.

Good for:

- Deploying complex infrastructure solutions with multiple resource types and intricate dependencies, such as network hubs or network virtual appliances.
- Implementing new, public, and private preview features not yet available in higher-level IaC tools.
- Use cases where no alternatives exist, such as defining policies, blueprints, or Azure Marketplace offers.

Challenging for:

- Adopting a modular approach, especially when you intend to maintain templates as building blocks for your customers to re-use in constructing their own systems.
- Seeking greater accessibility for both "writers" and "readers" among a broader audience.


### Bicep templates

Bicep is an abstraction layer built on top of ARM templates, designed to simplify syntax, improve readability, reduce errors, and introduce [additional functionality](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview?tabs=bicep#benefits-of-bicep). While Bicep templates use YAML syntax for readability, they are internally [compiled into JSON](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/frequently-asked-questions#what-happens-to-my-existing-arm-templates) for interactions with ARM API.

Benefits of Bicep Templates Include:

- **Simplified syntax**: makes the template syntax more human-readable, enhancing the overall developer experience and reducing the likelihood of syntax-related errors.
- **Functionality enhancements**: simplify common tasks and improve usability, making it easier to work with templates as modules.

Challenges of Bicep Templates Include:

- **YAML to JSON compilation**: this translation can complicate troubleshooting, as your configuration source is in Bicep/YAML, while Azure interactions and messages remain in JSON.

Good for:

- Creating a catalog of centrally maintained modules for use by end-users, ideal for Cloud Center of Excellence (CCoE) scenarios.
- Simplifying the definition of typical resources for better accessibility by a broader audience.

Less suitable for:

- Scenarios where fine-grained control, closely resembling API interactions, is crucial. This might be important for preview resources or highly complex deployments, where the translation from Bicep to JSON complicates troubleshooting.

---

<br/>  

> ^^^ This could be a line dividing "right" and "may be too much" levels of abstraction for a general user looking to automate their Azure platform management with IaC, but not required to become a software developer. 

## Level 3 - Terraform and other "cloud-agnostic" tools 

Many customers are convinced that adopting a multi-cloud strategy and choosing Terraform as a single, "cloud-agnostic" solution for IaC management can streamline their efforts.

However, the devil is in the details. While Terraform offers a unified language for multiple cloud providers, it brings its own set of complexities.

### Language

Terraform's cloud-agnostic appeal primarily rests in its language. Each cloud, with its distinct resources, services, and structures, requires Terraform to rely on specific "providers". These abstract providers bridge Terraform configurations to the relevant cloud provider's API.

Yes, Terraform is designed to be cloud-agnostic, however, this concept primarily applies within the Terraform language itself. Each cloud provider has its own set of resources and services with their unique attributes, object structures, and dependencies. Terraform relies on abstract "providers" for each cloud, responsible for translating Terraform configurations into API calls specific to that cloud provider.

So, even though you can utilize a unified language and configuration syntax across multiple cloud providers:

- you still need to understand the intricacies and capabilities of each provider when writing Terraform code;
- advanced features or services offered by a particular cloud provider may not be immediately supported by Terraform, possibly requiring community contributions or waiting for updates;
- extra abstraction layers over Azure API can produce quirks, which, once established, might be time-consuming to rectify.

> <a id="tf-private-endpoint"></a>A testament to this in how API attribute [`VirtualNetworkPrivateEndpointNetworkPolicies`](https://learn.microsoft.com/en-us/rest/api/virtualnetwork/subnets/create-or-update?tabs=HTTP#virtualnetworkprivateendpointnetworkpolicies) was mixed up with  CLI parameter [`--disable-private-link-service-network-policies`](https://learn.microsoft.com/en-us/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-update()) in the below Terraform [interpretation](https://github.com/hashicorp/terraform-provider-azurerm/blob/8621a756ed4f3e1e14a54e99a3b24602186918df/internal/services/network/subnet_resource.go#L604):  
[`enforce_private_link_endpoint_network_policies`](https://registry.terraform.io/providers/hashicorp/azurerm/3.0.0/docs/resources/subnet#enforce_private_link_endpoint_network_policies) - Setting this to `true` will **Disable** the policy and setting this to `false` will **Enable** the policy.


### State tracking

The second challenge is the **state file**. While many consider the ability to track state a significant advantage of Terraform, there are cons in specific scenarios. A deeper dive might be required, but key points include:

- Multi-user scenarios can be obstructed by the state file, particularly when multiple teams and actors (like policies and service principals) need to operate shared resources.
- Azure already maintains resource states, and tracking changes in the IaC repository might render external state files redundant.
- Shared infrastructure resources rarely require deletion, and the scenario of redeploying an entire system, like a network hub, is rarely feasible or desirable.
- Alternatives to Terraform's "plan" feature exist in native tools like ([ARM](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-what-if) and [Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-what-if)).
- Concerns regarding security (especially handling secrets), consistency (the need to reconcile external changes), locking, scalability, and performance.

### Cargo cult and Profits

Lastly, there might be philosophical considerations beneath the technical surface. Some individuals may have an almost religious belief in universal Terraform applicability for any IaC, perhaps because they haven't thoroughly explored alternative options or critically evaluated Terraform's drawbacks. 

On the other hand, certain businesses, particularly service providers, might find it more profitable to train their engineering teams to specialize in a single IaC tool. Selling complex IaC solutions that require more man-hours to maintain can increase revenues.

### In summary

Pros:

- **State Management**: if you really need it.

Cons:

- **Provider limitations**: requires an understanding of each cloud provider's specifics.
- **State file challenges**: can complicate multi-user scenarios and might not be necessary for some Azure implementations.
- **Abstraction layer**: introduces potential quirks that can complicate Azure interactions.

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

[Pulumi](https://github.com/pulumi/pulumi) adds one more abstraction, which is breaking down barriers between traditional software development and infrastructure management. Now you can use the language of your choice to manage your cloud...

This pushes the pros and cons of the previous concept to the next level.

Pulumi is suitable when:
- you have a team of Software Engineers proficient in a specific language,
- and management of the cloud is (part of) their software product

You possess a team of software engineers proficient in a specific language. This means they can dive right in without the need to learn a new, domain-specific language.
Cloud management isn't just a sideline task but is integrated as a core component of the software product they are developing. This allows for a seamless fusion of application development and infrastructure management.

---

<br/>

> **"AHA moment"**  
>
>  Recently while looking for proof that I'm not alone who is trying to avoid too much abstraction, I stumbled upon a Wikipedia article about [DRY principle](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), and quite liked the alternative AHA principle, which boils down to 
> 
>  **"optimizing for change first, and avoiding premature optimization"**  
>
 > AHA stands for avoid hasty abstractions. AHA is rooted in the understanding that the deeper the investment we've made into abstracting a piece of software, the more we perceive that the cost of that investment can never be recovered ([sunk cost fallacy](https://en.wikipedia.org/wiki/Sunk_cost)). Thus, engineers tend to continue to iterate on the same abstraction each time the requirement changes. AHA programming assumes that both WET and DRY solutions inevitably create software that is rigid and difficult to maintain. Instead of starting with an abstraction, or abstracting at a specific number of duplications, **software can be more flexible and robust if abstraction is done when it is needed**, or, when the duplication itself has become the barrier and it is known how the abstraction needs to function.


## Conclusion

Each tool and level of abstraction offers its own set of benefits and challenges. Your choice should align with your project requirements, team expertise, and the desired level of abstraction. Choose the tool that best aligns with your goals, rather than adapting your goals to fit the tool.

Use the right tool for the right job:

- portal and self-service tools for simple mortal end-users
- scripts for admin tasks
- native templates for foundational infrastructure 
- state tracking "meta"-tools for specialized teams and solutions

Kind regards,  
Your Captain Obvious

## Discussion

Happy to discuss and learn new things from you. 
