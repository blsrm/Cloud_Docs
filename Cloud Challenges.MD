## Technical Challenges / Problems Solved

1. In Private Cluster, after installing EKS we need to http_proxy/https_proxy to connect EKS control plane with worker nodes



### NT technology requirements (real time access requirements for VNF certification, NT Testlab): SRIOV, DPDK, NUMA core pinning

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sriov-networking.html
Enable enhanced networking with the Intel 82599 VF interface on Linux instances - Amazon Elastic Compute CloudEnable enhanced networking with the Intel 82599 VF interface on Linux instances in a VPC.

https://docs.aws.amazon.com/whitepapers/latest/ec2-networking-for-telecom/overview-of-performance-and-optimization-options.html
Overview of Performance and Optimization Options - Amazon EC2 Overview and Networking Introduction for Telecom CompaniesSingle-Root Input/Output Virtualization (SR-IOV) is a mechanism that virtualizes a single PCIe Ethernet controller to make it appear as multiple PCIe devices. Telecom providers have been deploying ...

https://docs.aws.amazon.com/whitepapers/latest/ec2-networking-for-telecom/enabling-enhanced-networking.html
Enabling Enhanced Networking - Amazon EC2 Overview and Networking Introduction for Telecom CompaniesAs covered in the previous section, Enhanced Networking can be based on Intel ixgbevf or EC2 ENA adaptor. The first step in enabling ENA is to check and verify what type of driver you have. Followi...
```
PV and HVM. you want select current generation HVM instances.
When it comes to EC2 instance support, DPDK is supported on Enhanced Networking instances, both Intel-based ixgbevf and AWS Elastic Network Adapter (ENA). 
All Nitro-based instances, such as C5, M5, I3, and T3, as well as Intel-based instances, such as C4, M4, and T2, provide DPDK support.
In Amazon EC2, all instances that support more than one CPU also support NUMA. These include i3.8xlarge, r5.8xlarge, c5.8xlarge, and above.
```
