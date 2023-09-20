---
#layout: post-argon
title: Name Resolution for Azure Private Endpoints
date: 2023-09-20 08:00:00 +0100
author: Igor
categories: [Azure, Networking]
tags: [azure, devops, caf]
#permalink: /post1/
slug: azure-pe-name-resolution
excerpt_separator: <!--more-->
redirect_from: [/post/azure-pe-name-resolution]
published: false
---
<div style="text-align: center">

![Bike Fall Meme](https://github.com/iromanovsky/irom.info/assets/15823576/74a1de77-c905-48ed-84fa-f508418aff08)

</div>

In my previous post, I covered the scenarios when to use Private Endpoints and when it is not a silver bullet.

In this post, I will delve into why name resolution is so important topic when discussing private endpoints, what approaches are available, and what solution I recommend genreally. 

Tech nerd content warning, open with caution!

<!--more-->

## Why you need to care about name resolution

When you deploy a private endpoint for your resource, you just get a private network interface with an IP address on your VNet dedicated to a specific service, like a storage blob or an SQL database. Note that the same resource can provide multiple services, like blob, file, table, etc. and each service should get its dedicated PE.

> NIC with IP `10.18.0.4` for blobs accessible on this address `https://irom.blob.core.windows.net/`

You should not use this IP address directly from your applications, since most of the services are protected with SSL encryption, the client has to validate the resource name, while Microsoft still uses its own wildcard certificate.

> Issued to: Common Name (CN)	*.blob.core.windows.net

> Issued by: Microsoft RSA TLS CA 02

..so the name validation for your connection to `https://10.18.0.4/` will fail

If you use the original URL, your query will go to the public endpoint

```
nslookup irom.blob.core.windows.net
..
irom.blob.core.windows.net canonical name = blob.ams06prdstr05a.store.core.windows.net.
..
Name: blob.ams06prdstr05a.store.core.windows.net
Address: 52.239.141.196
```
If you block the public access on your resource, your query will fail, otherwise (did you know that deploying a private endpoint does not block public access for some resources?) it will be served by the public endpoint, which is probably not what you wanted in the beginning.

Now it's clear that you need to find a way to substitute the public IP for the name `irom.blob.core.windows.net` with your private endpoint IP `10.18.0.4`

## 0. Hosts file

This is a quick and dirty, but still valid solution. You add a record to your client's hosts file:

```
10.18.0.4 irom.blob.core.windows.net
```

Pros:
- ~~Quick~~
- Simple. You don't rely and don't depend on external name resolution tricks.

Cons:
- ~~Dirty~~
- Does not scale well. When you need to access the same resources from multiple clients, you need to make these changes to each of them.
- Fragile.  When you need to access multiple resources on PEs, and occasionally update them, this method is prone to errors and inconsistencies.

Overall, this method is still viable for dev/test environments in general, and for [private data access pattern](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/data-considerations) when used carefully.
    
## 1. Azure Private DNS Zones

When you need to serve the same private endpoints to multiple VNets, Azure Private DNS Zones may help you:

1. For each resource (like storage) and service (like blob) you create a zone like `privatelink.blob.core.windows.net` (see the full list of zones [here](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration))
2. You [link](https://learn.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links) this zone to all the VNets where you need to use your private endpoints. 
3. When creating private endpoint resources, register your PE names to the DNS zones: `irom A record to 10.18.0.4`. When manually creating private endpoints on the Azure portal, it can do this for you automatically, however, when you automate this task, you need to take care of this, or use [policies for automation](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/private-link-and-dns-integration-at-scale)
4. When your clients in the VNets are resolving names of Private Endpoint enabled resources, they get an additional DNS query to _resourcename_.**privatelink**.service.domain namespace. 
   - If it gets resolved by your Private DNS Zone, the client gets your private IP address for the endpoint. 
   - If **privatelink** name like irom.privatelink.blob.core.windows.net is resolved externally, the client gets the Public Endpoint IP, which could be not desirable but could be a life-saving failover solution for some situations.

> <p>Why the zones for private endpoints are named like privatelink.blob.core.windows.net?</p><p>Because the parent namespace like  blob.core.windows.net contains the names of all Azure storage accounts, including other customers. If you create a private DNS zone named blob.core.windows.net, the names not registered in this zone, including resources with no private endpoints, will not get resolved and will fail to connect. This could be a scenario for disaster, be careful to not disrupt the name resolution of existing resources on Public Endpoints.</p>

Pros:
- You can centralize and automate private endpoint name resolution at the scale of your landing zone

Cons:
- You need automation to keep this system consistent and manage the whole lifecycle for names in the Private DNS zone.
- If you use custom DNS servers for your VNet, this scenario does not work (until you configure more advanced scenarios as described below)
- This scenario does not work if you deploy multiple private endpoints for the same resource name in different vnets or regions.

## 2. Custom DNS servers

Most enterprise customers already have their own DNS implementation for name resolution in their networks, and the VNets are configured to use custom DNS servers. In this case, you need to integrate the resolution of private endpoint names into the company's DNS solution.

### Option 2.1 Host private link zones on custom DNS servers

You may host private link zones like privatelink.blob.core.windows.net on your DNS servers. If DNS is hosted on Active Directory domain controllers, you might get the benefits of DNS zone replication for centralized updates and high availability.

Pros:
- Integration of private endpoint name resolution into your custom DNS solution, including serving on-premise clients

Cons:
- Requires separate management or privatelink records lifecycle, losing benefits of Azure Private DNS Zones

### Option 2.2 Forward privatelink namespaces from custom DNS to Azure DNS

This is the most common scenario, where VNet and On-Premises clients are configured to use Custom DNS servers, but these servers are configured to forward **privatelink** namespaces to Azure DNS

Pros:
- Benefits of both options 1 and 2.1: privatelink name resolution is integrated into your custom DNS, but the records management lifecycle still remains on the Azure side

Cons:
- More complexity

Challenges:
- For forwarding to work, the DNS server needs to reach Azure DNS:
   - For servers hosted in Azure, you need to forward queries to [Azure Magic IP 168.63.129.16](https://learn.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16)
   - For on-premise servers, you may use existing servers hosted in Azure as forwarders, or use [Azure Private Resolver](https://learn.microsoft.com/en-us/azure/dns/private-resolver-architecture) deployed in your VNet


> Note: when configuring conditional forwarding from on-premises, contrary to [CAF article](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration), it is important to forward **privatelink**.service.domain namespaces instead of **service**.domain namespaces. If forwarding fails, you only lose resolution of **privatelink** resources vs. all resources of the **service** which also may use public endpoints. 


## 3. Advanced scenarios

### 3.1 Azure Firewall

For Azure Firewall FQDN rules to work, all the clients served with this firewall [should use](https://learn.microsoft.com/en-us/azure/firewall/dns-details#clients-not-configured-to-use-the-firewall-dns-proxy) the firewall as their DNS server, quoting Microsoft:

> If a client computer is configured to use a DNS server that isn't the firewall DNS proxy, the results can be unpredictable.

Azure Firewall, in turn, should be able to resolve privatelink zones and use the company's custom DNS solution, if it is present.

This means:
- If you need to resolve just privatelink namespaces from Azure Firewall, just link Azure Private DNS zones, like in option 1, to AzureFirewallSubnet.
- If you need to use both privatelink namespaces and the company's custom DNS solution, [configure the firewall](https://learn.microsoft.com/en-us/azure/firewall/dns-settings) to use custom DNS servers like in option 2.2.

### 3.2 Secure Hub with Azure Firewall on Azure VWAN

When using Secure Hub with Azure Firewall on Azure VWAN, you can't link Azure Private DNS zone to AzureFirewallSubnet, because this subnet does not exist. The only option to resolve privatelink namespaces is to configure the DNS server on Azure Firewall to use custom DNS servers, like in option 2.2.

This solution is discussed in great detail in the article [Guide to Private Link and DNS in Azure Virtual WAN](https://learn.microsoft.com/en-us/azure/architecture/guide/networking/private-link-virtual-wan-dns-guide).

### 3.3 Private Endpoints of the same resource in multiple VNets

Sometimes you need to deploy multiple private endpoints for the same resource, like a central fire share, to multiple VNets and/or regions, since traffic routing from multiple regions to the single private endpoint is less effective for bandwidth and latency than bringing dedicated private endpoint closer to the clients. However, you end up in a situation when the same name of private endpoint resource should be resolved to different IP addresses depending on the client's locations:

```
10.18.0.4 irom.blob.core.windows.net # if the client from West Europe
10.19.0.4 irom.blob.core.windows.net# if the client from North Europe
...
# ..and so on..
```


I do not have a perfect solution for this, just workaround options

#### 3.3.1 Private data store pattern

If your resource is consumed privately from a limited number of clients, like in [private data store pattern](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/data-considerations), the easiest solution could be to use the good old hosts file, like in option 0.

#### 3.3.2 Multiple private DNS Zones for the same namespace

Private DNS Zones are global resources that could be linked to VNets in any region, however, they cannot select which IP to return to clients based on their location.

We can work around this with the below approach:

1. Create a pinpoint Private DNS Zone for _each VNet_ and each _private endpoint_ for the same resource, but in a different Resource Group

> The pinpoint DNS entry is a zone created for a single host only

2. Link pinpoint zones to their respective VNets and/or regions.

| VNet | Region | DNS Zone | Resource Group | Target Resource |
| -- | - | -- | -- | -- |
| Hub-WE-vnet | West Europe | myglobalstore.privatelink.blob.core.windows.net | Hub-WE-Endpoints-rg | myglobalstore |
| Hub-NE-vnet | North Europe | myglobalstore.privatelink.blob.core.windows.net | Hub-NE-Endpoints-rg | myglobalstore |
| Hub-WE-vnet | West Europe | myglobalsql.privatelink.database.windows.net | Hub-WE-Endpoints-rg | myglobalsql |
| Hub-NE-vnet | North Europe | myglobalsql.privatelink.database.windows.net | Hub-NE-Endpoints-rg | myglobalsql |

Pros:
- This approach allows resolution of the same privatelink name to different IPs depending on the client VNet to allow effective traffic routing

Cons:
- It is not effective to manage the lifecycle of many pinpoint DNS zones, their VNet links, and private endpoint resources at scale, from management, automation, and [limits](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-dns-limits) perspective


#### 3.3.3 Policy-based DNS name resolution

By using 3rd party DNS tools, like DNS Query Resolution Policies on Windows Server starting from 2016, you can create multiple copies of DNS zones of the same name, like privatelink.blob.core.windows.net, and provide the replies based on the client location. See more info on these pages:

- [DNS Policies Overview](https://learn.microsoft.com/en-us/windows-server/networking/dns/deploy/dns-policies-overview)
- [Use DNS Policy for Geo-Location Based Traffic Management with Primary Servers](https://learn.microsoft.com/en-us/windows-server/networking/dns/deploy/primary-geo-location)

Pros: 
- The same as for 3.3.2: This approach allows resolution of the same privatelink name to different IPs depending on the client VNet to allow effective traffic routing

Cons:
- Similar as for 3.3.2, but you shift the complexity from Azure to your 3rd party solution.

## Conclusion

Based on experience, I recommend this approach:

1. Deploy Private DNS zones for each privatelink namespace you are using in your organization (option 1)

2. Configure your AD Domain Controllers for conditional forwarding of privatelink namespaces to Azure DNS (option 2.2)

3. If you use Azure Firewall, configure the Firewall DNS proxy to use AD DCs deployed in Azure as DNS servers (option 3.1)

4. If you use Azure Firewall, you have no choice but to use it as a DNS server on your clients, if not using Azure Firewall, use AD DC

5. If you need Private Endpoints of the same resource in multiple VNets, use option 3.3.3 for Policy-based DNS name resolution for _specific pinpoint DNS zones_.

When I get more inspiration, I plan to translate the above into a pretty [Mermaid diagram](/_posts/2023-09-04-diagrams-as-code.md).

## Links

- [Choosing between Public, Service, and Private Endpoints in Azure](/_posts/2023-09-14-azure-endpoints.md)

## Discussion

Before now, I never had the information described here structurally placed on one page. I will be so happy to discuss and learn better approaches for the covered scenarios. You are very welcome to Comments.
