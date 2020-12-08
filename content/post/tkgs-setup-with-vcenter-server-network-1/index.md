---
title: 'TKG-S Setup with vCenter Server Network - Part 1'
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

date: "2020-11-23"
lastmod: "2020-11-23"
featured: false
draft: false
---

With the new vCenter 7.0.1 it is possible to deploy the *Workload Management* feature using the *vCenter Server Network* instead of *NSX-T*. I wanted to try it out immediately and this is my experience.

{{% toc %}}

## Prerequisites

* A VMware vCenter 7.0.1 installed
* A VMware ESXi 7.0.1 installed
* Distributed Switch with the Distributed Port Group configured for:
  * Management Network
  * Workload Network
  * Frontend Network
* Every Network *MUST* have its *Gateway*
* vSAN configured

## Network Configurations

* Management: 
  * subnet: 192.168.1.0/24
  * gateway: 192.168.1.254
  * dns: 192.168.1.254
  * ntp: 192.168.1.254
  * HAproxy management ip: 192.168.1.245
  * dataplane API port: 5556
  * SupervisorControlPlanes starting IP: 192.168.1.140
* Workload:
  * subnet: 192.168.140.0/24
  * gateway: 192.168.140.254
  * dns: 192.168.1.254
  * workload IPs range: 192.168.140.2-192.168.140.253
  * HAproxy workload IP: 192.168.140.1
* Frontend
  * subnet: 192.168.150.0/24
  * gateway: 192.168.150.254
  * dns: 192.168.1.254
  * frontend IPs range: 192.168.150.1-192.168.150.253
  * frontend IPs ranges with CIDR: 192.168.150.0/25,192.168.150.128/26,192.168.150.192/27,192.168.150.224/28,192.168.150.240/29,192.168.150.248/30,192.168.150.252/31
  * HAproxy frontend IP: 192.168.150.1

## Procedures
### Content Library
Go to the *Content Libraries* menu
{{< figure src="images/image01.png" title="Content Libraries Menu" >}}
and create a new one
{{< figure src="images/image02.png" title="Create Library" >}}
{{< figure src="images/image03.png" title="Kubernetes Library" >}}
Select *Subscribed content library* and select when you prefer to download the content.<br>Use *https://wp-content.vmware.com/v2/latest/lib.json* as subscription url.
{{< figure src="images/image04.png" title="Configure content library" >}}
Select the storage and click *NEXT*
{{< figure src="images/image05.png" title="add storage" >}}
Check if all the data are correct and then click *FINISH*
{{< figure src="images/image06.png" title="Finish" >}}

### Storage Policy
Go to the *Policies and Profiles* menu
{{< figure src="images/image07.png" title="Policies and Profiles" >}}
Click on *VM Storage Policies*
{{< figure src="images/image08.png" title="VM Storage Policies" >}}
and click *CREATE*. Select the right vCenter Server and choose a *Name* for the policy. Add a *Description* if needed. Then click *NEXT*
{{< figure src="images/image09.png" title="Create the Storage Policies" >}}
Click *Enable rules for "vSAN" storage and click *NEXT*
{{< figure src="images/image10.png" title="Policy Structure" >}}
Select the *vSAN* policies. My setup is a single vSAN node.
{{< figure src="images/image11.png" title="vSAN Policy" >}}
Click on *Advanced Policy Rules* and configure what it's needed. Then click *NEXT*
{{< figure src="images/image12.png" title="vSAN Advanced Policies" >}}
Check in the *Storage compatibility* page if the *vsanDatastore* is compatible with the choseb policies, then click *NEXT*
{{< figure src="images/image13.png" title="Storage compatibility" >}}
Review all the configurations, then click *FINISH*
{{< figure src="images/image14.png" title="Finish" >}}

### VMware-HAproxy OVA as LoadBalancer
Without *NSX-T* there is the need to use something which manages the LoadBalancing configuration. We'll use *HAproxy*. The *HAproxy* OVA is going to be deployed with the 3 networks: the *management* network, the *workload* network which is used to let the *SupervisorControlPlanes* communicate with the *nodes* of the *guest clusters* and also be used as *real server* and the *frontend* network used for the *VIPs* created by the *kubernetes service type LoadBalancer*. The *HAproxy* istance is configured by its [*dataplane api*](https://www.haproxy.com/documentation/dataplaneapi/latest/) using the *management IP*. The *frontend* IPs ranges with CIDR are used for the *anyIP* kernel feature. The *HAproxy* will respond to the *ARP* requests for **any** IP address in those subnets.<br><br>Go to the *Hosts and Clusters* menu.
{{< figure src="images/image15.png" title="Hosts and Clusters" >}}
Right click on the cluster and then click *Deploy OVF Template*
{{< figure src="images/image16.png" title="Deploy" >}}
Add *https://cdn.haproxy.com/download/haproxy/vsphere/ova/vmware-haproxy-v0.1.8.ova* as *URL* and then click *NEXT*
{{< figure src="images/image17.png" title="Add OVA" >}}
Select the *Virtual machine name* and the location for the virtual machine then click *NEXT*
{{< figure src="images/image18.png" title="Name and location" >}}
Select a *compute resource* then click *NEXT*
{{< figure src="images/image19.png" title="Compute resource" >}}
Review the details then click *NEXT*
{{< figure src="images/image20.png" title="Review" >}}
Accept all the *license agreements* and then click *NEXT*
{{< figure src="images/image21.png" title="License agreements" >}}
Select *Frontend Network* and then click *NEXT*
{{< figure src="images/image22.png" title="Frontend Network" >}}
Select the storage and then click *NEXT*
{{< figure src="images/image23.png" title="Select storage" >}}
Select the *source network* and then click *NEXT*
{{< figure src="images/image24.png" title="Source Network" >}}
Set the *root password* and add your *CA*. If you don't have one, leave blank and it will be generated
{{< figure src="images/image25.png" title="Appliance Configuration" >}}
Configure the *network*
{{< figure src="images/image26.png" title="Network Config 1" >}}
{{< figure src="images/image27.png" title="Network Config 2" >}}
Add the Load Balancing settings and click *NEXT*
{{< figure src="images/image28.png" title="Load Balancing" >}}
Review the configurations and then click *FINISH*
{{< figure src="images/image29.png" title="Review" >}}
Power On the *haproxy* VM
{{< figure src="images/image30.png" title="Power On" >}}

### Workload Management
Go to the *Workload Management* menu
{{< figure src="images/image31.png" title="Workload Management" >}}
Click *GET STARTED*
{{< figure src="images/image32.png" title="Get started" >}}
Select *vCenter Server Network* and then click *NEXT*
{{< figure src="images/image33.png" title="vCenter Server and Network" >}}
Select the compatible cluster and then click *NEXT*
{{< figure src="images/image34.png" title="Select a Cluster" >}}
Select the size for *control plane VM* and then click *NEXT*
{{< figure src="images/image35.png" title="Control Plane size" >}}
Select the *storage policy* to the *Control Plane VMs*
{{< figure src="images/image36.png" title="Storage" >}}
Configure load balancer for workloads created on the cluster
{{< figure src="images/image37.png" title="Load Balancer" >}}
If the *Server Certificate Authority* used is a self-signed cert, retrieve that data from the VM with the follow *powershell* snippet
```
  $vc = "10.174.71.163"
  $vc_user = "administrator@vsphere.local"
  $vc_password = "Admin!23"
  Connect-VIServer -User $vc_user -Password $vc_password -Server $vc
  $VMname = "haproxy-demo"
  $AdvancedSettingName = "guestinfo.dataplaneapi.cacert"
  $Base64cert = get-vm $VMname |Get-AdvancedSetting -Name $AdvancedSettingName
  while ([string]::IsNullOrEmpty($Base64cert.Value)) {
  Write-Host "Waiting for CA Cert Generation... This may take a under 5-10
  minutes as the VM needs to boot and generate the CA Cert (if you haven't provided one already)."
  $Base64cert = get-vm $VMname |Get-AdvancedSetting -Name
  $AdvancedSettingName
  Start-sleep -seconds 2
  }
  Write-Host "CA Cert Found... Converting from BASE64"
  $cert = [Text.Encoding]::Utf8.GetString([Convert]::FromBase64String($Base64cert.Value))
  Write-Host $cert
```
{{< figure src="images/image38.png" title="Snippet" >}}
and then click *NEXT*
{{< figure src="images/image48.png" title="Next" >}}
Configure the *Management Network* and then click *NEXT*
{{< figure src="images/image39.png" title="Management Network" >}}
Configure the *IP address for Services* and click *ADD* to configure the *Workload Network* 
{{< figure src="images/image40.png" title="IP address for Services" >}}
Configure the *Workload Network* and then click *SAVE*
{{< figure src="images/image41.png" title="Workload Network" >}}
Click *NEXT*<br>Add the *Content Library* clicking *ADD*
{{< figure src="images/image42.png" title="Add Content Library" >}}
Select the *Content Library* and click *OK*
{{< figure src="images/image43.png" title="Ok" >}}
Click *NEXT*
{{< figure src="images/image44.png" title="Next" >}}
Review the config and then click *FINISH*
{{< figure src="images/image45.png" title="Finish" >}}
Now, wait until the cluster is configured
{{< figure src="images/image46.png" title="Wait" >}}
When the *Config Status* turns green, the *Workload Management* is ready with the *Control Plane Node IP Address* needed to connect to the cluster
{{< figure src="images/image49.png" title="Workload Management Ready" >}}

The creation of *Namespace* and *Guest Clusters* will be covered in the second part
