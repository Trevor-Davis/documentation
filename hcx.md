## Connect AVS to Your On-Premises Environment
To do this there are [three pre-requistes](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-expressroute-global-reach-private-cloud#prerequisites) - 

1. You have established connectivity to/from AVS and Azure vNets.  If you have gotten this far, this should be complete.  
2. Have an Express Route from your on-premises environment to Azure.
3. A /29 non-overlapping network address block for the ExpressRoute Global Reach peering (you should have [defined this already](/production-ready-deployment-steps.md/#global-reach-peering) as part of the planning phase).

**NOTE:** You can connect via VPN, but that is out of scope for this kickstart document.

### Establish Global Reach Connection
Follow [these instructions](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-expressroute-global-reach-private-cloud) to establish on-premises connectivity to/from AVS by configuring Global Reach.

### Verify On-Premises Network Connectivity
You should now see in your on-premises edge router where the Express Route connects the NSX-T network segments and the AVS management segments in the route tables.

Everyone's environment will be different, some need to allow these routes to propagate back into the greater on-premises network, some will not.  Some will have firewalls protecting the Express Route, some will not.  Assuming no firewalls and no route pruning is occurring you should at this point be able to ping from your on-premises environment the vCenter server in AVS and the [virtual machine](/production-ready-deployment-steps.md/#put-a-virtual-machine-on-the-nsx-t-network-segment) you put on the NSX-T segment.  

Additionally, from the virtual machine on the NSX-T segment you should be able to reach resources in your on-premises environment.


## Deployment and Configuration of HCX for Network Extension and/or Workload Migration

### Active HCX in AVS
... insert info here ...

### Deploy and Configure HCX On-Premises
... insert info here ...
