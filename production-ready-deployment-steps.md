---
title: Production Ready Deployment of Azure VMware Solution
description: This article outlines the workflow of an Azure VMware Solution (AVS) deployment.  The final result will be an environment ready for VM creation and migration.
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: tredavis 
---
# Azure VMware Solution (AVS) Deployment
This article outlines the workflow of an Azure VMware Solution (AVS) deployment. The final result will be a production ready environment.

## Planning
In this section you will be collecting data to use in subsequent steps.  

If you would like you can [download this file](https://github.com/tredavismicrosoft/documentation/raw/master/avs-pre-deployment-checklist.docx) so you can document all the information collected in this planning phase.  
 
### Identify - IP Address Segment for AVS Platform

The first step in deploying AVS will be to plan out the IP segmentation.  

During deployment AVS injests a /22 network you provide, and then carves it up into smaller segments and use those segments for IPing vCenter, HCX, NSX, vMotion network, etc...

AVS connects to your Azure vNet via an internal Express Route and (in most cases) will ultimately connect to your datacenter via Global Reach (Express Route and Global Reach will be discussed later in detail). 

What you need to be aware of is that AVS, your existing Azure environment and your on-premesis enviornment will all exchange routes (typically). That being the case you will need to ID a /22 network to input during the AVS deployment which does not overlap anything you already have on-premesis or in Azure.

**Example:** 10.0.0.0/22

[Please see this link for details](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-network-checklist#network-connectivity).  

![](/IPs.png)

---

### Identify - IP Address Segment for VM Workloads in AVS

Any IP segments you create in AVS need to be unique across your Azure and on-premises footprint.  

Identify a IP segment which you will use to create your first network (NSX Segment) in your AVS private cloud.

**Example:** 10.0.4.0/24

![](/nsxsegment.png) 

---

### Identify - Networks Which Will Be Extended to AVS From On-Premises (Optional)

You may choose to extend network segments from on-premises to AVS.  If you are planning on extending networks from on-premises those networks must connected to a [vSphere Distributed Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-B15C6A13-797E-4BCB-B9D9-5CBC5A60C3A6.html) in your on-premises VMware environment.  If the network(s) you will be extending live on a [vSphere Standard Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-350344DE-483A-42ED-B0E2-C811EE927D59.html) they cannot be extended.  

---

### Identify - Global Reach Peering Network
A /29 non-overlapping network address block for the ExpressRoute Global Reach peering.  The IPs in this segment will be used at each end of the Global Reach connection for purposes of connecting the two Express Routes.

**Example:** 10.1.0.0/29

![](/grip.png)

---

### Identify - Subscription
The subscription you plan to use for the deployment.  You can either create a new subscription or re-use an existing one.

**NOTE:** This subscription must be associated with a Microsoft Enterprise Agreement.

---

### Identify - Resource Group
Generally, a new resource group will be created for AVS and it is supporting Azure components, but this is not a requirement.

---

### Identify - Region
Identify which region you want AVS deployed, please see the [Azure Products Available By Region Guide](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=azure-vmware-cloudsimple).

---

### Identify - Resource Name
This is a friendly name which you will title your AVS private cloud.

---

### Identify - Size Nodes
Which size nodes you want to build out your AVS with.  [See AVS documentation for a complete list](https://docs.microsoft.com/en-us/azure/azure-vmware/concepts-private-clouds-clusters#hosts).

---

### Identify - Number of Hosts
How many hosts will you build out the AVS cluster with.  The minimum node count is 3, maximum is 16 per cluster.  See [this link](https://docs.microsoft.com/en-us/azure/azure-vmware/concepts-private-clouds-clusters#clusters) for more details.

---

### Identify - vCenter Admin Password
During the deployment you will be prompted to define the vCenter admin password.

---

### Identify - NSX Admin Password
During the deployment you will be prompted to define the NSX admin password.

---

### Azure vNet To Attach AVS
To access your AVS private cloud the Express Route which comes with AVS will need to attach to an Azure vNet.  

During deployment you can define a new vNet or choose an existing vNet and then during the AVS deployment the Express Route from AVS will be connected to an Express Route Gateway on the vNet defined during deployment.  

It is important to note that if you choose an existing vNet you need to select one which does not have a pre-existing Gateway Subnet.  

If you want to connect the Express Route from AVS to an existing Express Route Gateway, this can be done after deployment.  

So in summary, do you want to connect AVS to an existing Express Route Gateway?  
Yes = Identify the vNet for planning purposes, but during deployment this will not be used.
No = Identify an existing vNet or create a new vNet a new vNet during deployment.

Either way, document what you want to do in this step.

**NOTE:**  This vNet will be seen by your on-premises environment and AVS so make sure whatever IP segment is used in this vNet and subnets do not overlap.

![](/avsexr.png)

## Next steps

Now that all the needed information is documented/gathered, continue to the next section to create your AVS private cloud.

> [!div class="nextstepaction"]
> [Tutorial: Deploy AVS](avs-deployment.md)