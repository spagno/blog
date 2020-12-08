---
title: 'TKG-S Setup with vCenter Server Network - Part 2'
subtitle: ''
summary: TKG-S Setup with vCenter Server Network.
authors:
- Spagno
tags:
- kubernetes
- vsphere 7
- vsphere 7 with kubernetes
- vSAN
categories:
- kubernetes
- storage
- vcenter
- vsphere
- vSAN

date: "2020-12-08"
lastmod: "2020-12-08"
featured: false
draft: false
---

{{% toc %}}

## Namespace
Go to *Namespaces* menu
{{< figure src="images/image1.png" title="Namespaces menu" >}}
Click *CREATE NAMESPACE* then select the *Cluster*. Set a *Name* for the *namespace* and select the *Network*. Add a *Description* if needed and click *CREATE*
{{< figure src="images/image2.png" title="Create Namespace" >}}