---
title: Production Ready Deployment of Azure VMware Solution
description: This article outlines the workflow of an Azure VMware Solution (AVS) deployment.  The final result will be an environment ready for VM creation and migration.
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: tredavis
---
# Azure VMware Solution (AVS) Deployment

This article outlines the workflow of an Azure VMware Solution (AVS) deployment. The final result will be a production ready environment ready for VM creation and migration.

## Planning

### IP Address Segment for AVS Platform

The first step in deploying AVS will be to plan out the IP segmentation.  AVS connects to your Azure vNet via an internal Express Route and (in most cases) will ultimately connect to your datacenter via Global Reach.  Express Route and Global Reach will be discussed later in detail.  Because AVS will see Azure vNet networks and on-premesis networks the network used for AVS deployment needs to be unique across these environments.  Identify a /22 network to be used.  

[Please see this link for details](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-network-checklist#network-connectivity).  

### IP Address Segment for VM Workloads in AVS (Optional)

You can create new network segments in AVS.  Like the IP addressing for the platform, any IP segments you create in AVS need to be unique across your Azure and on-premesis footprint.  If you are planning on creating new IP segments in AVS make note of those.  You may not be planning on creating new segments if you are only extending on-premesis networks (discussed in the next section).  It's important to note than you can create new networks segments in AVS and/or extend network segments from on-premesis. 

### ID Networks Which Will Be Extended to AVS From On-Premesis (Optional)

You may choose to extend network segments from on-premesis.  If you are planning on extending networks from on-premesis those networks must connected to a [vSphere Distributed Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-B15C6A13-797E-4BCB-B9D9-5CBC5A60C3A6.html) in your on-premesis VMware environment.  If the network(s) you will be extending live on a [vSphere Standard Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-350344DE-483A-42ED-B0E2-C811EE927D59.html) they cannot be extended.  It's important to note than you can create new networks segments in AVS and/or extend network segments from on-premesis.

### Subscription
The subscription you plan to use for the deployment.  You can either create a new subscription or re-use an existing one.

### Resource Group
Generally, a new resource group will be created for AVS and it's supporting Azure components, but this is not a requirement.

### Region
Identify which region you want AVS deployed, please see the [Azure Products Availabe By Region Guide](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=azure-vmware-cloudsimple).

### Resource Name
This is a friendly name which you will call your AVS private cloud.

### Size Nodes
Identify which size nodes you want to build out your AVS with.  [See AVS documentation for a complete list](https://docs.microsoft.com/en-us/azure/azure-vmware/concepts-private-clouds-clusters#hosts).

### Number of Hosts
How many hosts will you build out the AVS cluster with.  The minimum node count is 3, maximum is 16 per cluster.  See [this link](https://docs.microsoft.com/en-us/azure/azure-vmware/concepts-private-clouds-clusters#clusters) for more details.

### vCenter Admin Password
During the deployment you will be prompted to define the vCenter admin password.

### NSX Admin Password
During the deployment you will be prompted to define the NSX admin password.

### Azure vNet To Attach AVS
To access your AVS private cloud the Express Route which comes with AVS will need to attach to an Azure vNet.  During deployment you can define a new vNet or choose an existing vNet and then during the AVS deployment the Express Route from AVS will be connected to an Express Route Gateway on the vNet defined during deployment.  It's important to note that if you choose an existing vNet you need to select one which does not have a pre-existing Gateway Subnet.  

If you want to connect the Express Route from AVS to an existing Express Route Gateway, this can be done after deployment.  If you want to do this, then do not specify an Azure vNet during deployment.

So in summary, do you want to connect AVS to an existing Express Route Gateway?  
Yes = Do not fill in this section.  
No = either identify an existing vNet or create a new vNet.

Remember, this vNet will be seen by your on-premesis enviornment and AVS so make sure whatever IP segment is used in this vNet and subnets do not overlap.

## Deployment and Configuration of AVS Platform

### Deploy AVS
Take the information you collected from the Planning section above and [deploy the AVS Private Cloud](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-create-private-cloud).

### Create AVS Jumpbox
When the AVS private cloud completes deployment you will need a virtual machine in the vNet where AVS connects so you can reach vCenter and NSX.  Typically this jumpbox will not be needed once all the networking is configured (express route to/from on-premesis and global reach).  But until then it's a handy tool to have so you can reach vCenter and NSX in AVS.  

Here are instructions on how to create a [virtual machine in Azure](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-access-private-cloud#create-a-new-windows-virtual-machine).  You will want to create this virtual machine in whatever vNet you have [identified and/or created as part of the deployment process](/production-ready-deployment-steps.md#azure-vnet-to-attach-avs).

### Connect AVS to vNet with Express Route 
If you did not define a vNet in the [deployment step](/production-ready-deployment-steps.md/#deploy-avs) and your intent is to connect the AVS Express Route to an existing Express Route Gateway then follow [steps 1-4 in this section](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-configure-networking#connect-expressroute-to-the-virtual-network-gateway), then move to the next section.

If you already defined a vNet in the [deployment step](/production-ready-deployment-steps.md/#deploy-avs) then skip to the next section.

### Verify Network Routes Being Advertised From AVS to Azure vNet
At this point you should have a jumpbox in the vNet where AVS connects via it's Express Route.  Go to that jumpbox's network interface in Azure and [view the effective routes](https://docs.microsoft.com/en-us/azure/virtual-network/manage-route-table#view-effective-routes).

You should see in the effective route list the networks which were created as part of the AVS deployment.  You will see multiple /24 networks which were derived from the [/22 network you defined](/production-ready-deployment-steps.md#ip-address-segment-for-avs-platform) input during the [deployment step](/production-ready-deployment-steps.md/#deploy-avs)

It will look something similar to this.  Notice there are a series of /24 networks, these were all derived from the 10.74.72.0/22 network being input during the deployment, in this example.  

If you see something similiar to this you should be able to connect to vCenter in AVS.

![](/effectiveroutes.png)

### Connect and Log Into vCenter and NSX-T
You will need to log into the jumpbox you created in the earlier step, when you haved logged into the jumpbox open a web browser and navigate to and log into both vCenter and NSX-T admin console.  

You can identify the IP addresses and credentials of both vCenter and NSX-T admin console by [following the instructions here](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-access-private-cloud#connect-to-the-local-vcenter-of-your-private-cloud).

### Create a Network Segment on AVS
If you plan on creating new network segments in your AVS environment you will need to create those via NSX-T.  In the planning section you may have defined which networks you are going to create in AVS, you can create those now.  If you didn't already decide which networks you are going to create do that now.  

Just like on-premesis you are going to want to define a CIDR block of your choice which does not overlap with anything in Azure or on-premesis.  If you are not planning on connecting AVS back to your on-premesis enviornment, still, don't overlap the networks, you never know down the road if you will connect them.

Follow [these instructions](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-nsx-t-network-segment) to create an AVS network segment.

### Verify NSX-T Segment is Advertised
Re-do the [Verify Network Routes Being Advertised From AVS to Azure vNet](/production-ready-deployment-steps.md#verify-network-routes-being-advertised-from-avs-to-azure-vnet) step. 

But now you should see an additional route in the list, you should see the network segment(s) you just created in the previous step.

### Identify DNS Server
In the next steps you are going to be configuring either staticly a virtual machine on the NSX-T segment or assigning IP addresses via DHCP to that virtual machine, either way, you are going to need to have a DNS server identified.  You can use a DNS server in your environment which is accessible, but that this point you may not have access to the DNS server in your environment because Global Reach has not been configured.  

If you have access to a DNS server you have already created somewhere in your enviornment, use that, if not identify the DNS server being used by your jumpbox and use that same DNS server.

### Provide DHCP Services to NSX-T Network Segment (Optional)
If you want to use DHCP on your NSX-T segment(s), follow the guidance in this step, if not, skip to XXXXXXXXXXXXXXXX.  

Now that you have created your NSX-T network segment in AVS you can either leverage NSX-T as a DHCP Server or you can relay DHCP requests from the NSX-T segment(s) to a DHCP server somewhere else in your environment.

If you want to use NSX-T as the DHCP Server for the segment(s) you just created, you will want to [create a DHCP server in NSX-T](https://docs.microsoft.com/en-us/azure/azure-vmware/manage-dhcp#create-dhcp-server) and [relay to that server](https://docs.microsoft.com/en-us/azure/azure-vmware/manage-dhcp#create-dhcp-relay-service).

If you want to relay DHCP requests to a DHCP server somewhere in your environment which you have built or already have, then [only do the relay configuration](https://docs.microsoft.com/en-us/azure/azure-vmware/manage-dhcp#create-dhcp-relay-service).

### Put a Virtual Machine On the NSX-T Network Segment
In your AVS vCenter you will want to deploy a VM, you will use this VM to verify connectivity from your AVS network(s) to the Internet, to Azure vNet(s) and later to on-premesis.

Do this as you would in any vSphere environment.  Attach this VM to the/one of the networks which you just created in NSX-T.  If you setup DHCP in the previous step you will get your network configuration for that VM from that DHCP server (don't forget to setup the scope), and if you are going to staticly configure, then just configure as you normally would.

### Test NSX-T Segment Connectivity 
Log into the virtual machine you created in teh previous step.  Verify connectivity from your AVS network(s) to the Internet, to Azure vNet(s) and to on-premesis.  To do this complete the following steps;

Ping an IP on the internet.

Go to an Internet site via web browser.

Ping the Jumpbox which sits on the Azure vNet.

** --- **
** --- ** 

> [!IMPORTANT]
> An AVS admin must not modify the configuration of NSX-T Edge or Tier-0 Gateway. This may result in a loss of service.



> At this point AVS is up and running and you have successfully established connectivity to/from Azure vNets and the Internet. 



### Configure Azure Global Reach To Connect AVS to On-Premesis
... insert info here ...

## Deployment and Configuration of HCX for Network Extension and/or Workload Migration

### Active HCX in AVS
... insert info here ...

### Deploy and Configure HCX On-Premesis
... insert info here ...
