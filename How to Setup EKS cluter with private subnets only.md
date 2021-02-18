## How to setup AWS EKS Cluster in Private Subnets Only

This topic describes how to deploy a private cluster without outbound internet access. This is applicable for **EKS Managed / Self Node Cluster** and **Fargate Profiles**

Nodes must be able to communicate with the control plane and other AWS services. If nodes are deployed in a private subnet, then it must have either:


    1. Set up a default route for the subnet to a NAT gateway. The NAT gateway must be assigned a public IP address to provide internet access for the nodes.
    2. Configuration of necessary settings for the subnet and taken the necessary actions listed below for Private clusters.


## Requirements

The following requirements must be met to run Amazon EKS in a private cluster without outbound internet access.

1. A container image must be in or copied to **Amazon Elastic Container Registry** (Amazon ECR) or to a registry inside the VPC to be pulled.

1. **Endpoint private access** is required for nodes to register with the cluster endpoint. **Endpoint public access** is optional.

1. You may need to include the **VPC endpoints found at VPC endpoints** for private clusters.

1. The **aws-auth ConfigMap** must be created from within the VPC. For more information about create the aws-auth ConfigMap, see Managing users or IAM roles for your cluster.

1. You must include the following text to the bootstrap arguments when launching self-managed nodes. This text bypasses the Amazon EKS introspection and does not require access to the Amazon EKS API from within the VPC. Replace <**cluster-endpoint**> and <**cluster-certificate-authority**> with the values from your Amazon EKS cluster.

    ```
    --apiserver-endpoint <cluster-endpoint> --b64-cluster-ca <cluster-certificate-authority>
    ```

## Considerations
Here are some things to consider when running Amazon EKS in a private cluster without outbound internet access.

1. **AWS X-Ray** is not supported with private clusters.

1. **Amazon CloudWatch Logs** is supported with private clusters, but you must use an Amazon CloudWatch Logs VPC endpoint.

1. Self-managed and managed nodes are supported. The instances for nodes must have access to the VPC endpoints. If you create a managed node group, the VPC endpoint security group must allow the CIDR for the subnets, or you must add the created node security group to the VPC endpoint security group.

1. IAM roles for service accounts is supported. You must include the **STS VPC endpoint**.

1. The Amazon EBS CSI driver is supported. Before deploying, the kustomization.yaml file must be changed to set the container images to use the same Region as the Amazon EKS cluster.

1. The Amazon EFS CSI driver is supported. Before deploying, the kustomization.yaml file must be changed to set the container images to use the same Region as the Amazon EKS cluster.

1. The Amazon FSx for Lustre CSI driver is not supported.

1. **AWS Fargate is supported with private clusters. You must include the STS VPC endpoint**. For more information, see VPC endpoints for private clusters. You can use the AWS load balancer controller to deploy AWS Application Load Balancers and Network Load Balancers with. The controller supports network load balancers with IP targets, which are required for use with Fargate. For more information, see Application load balancing on Amazon EKS and Load balancer – IP targets.

1. App Mesh is supported with private clusters when you use the App Mesh Envoy VPC endpoint.


## IP Block select for Private subnets

Be sure that the subnets that you specify have enough available IP addresses for the network interfaces and your pods.

1. Consider System resources consuming IPs - Minimum IPs (approx. ~ 10 to 20) are consumed by EKS Cluster and Internal resources core dns, kube-proxy, end point creations.
2. Conside no of application and its micro services as of now and will be grow in the future
3. Consider replicas of each services (approx. 2 to 3) based on work load will consume additional IPs allocation

It is better to block full subnet ranges (/24) for each cluster in order to avoid IPs shortage.

## Below are Check lists to verify and confirm for Private Cluster setup

1. Below are the important VPC endpoints for private clusters need to be created

    ```
    com.amazonaws.eu-central-1.ec2
    com.amazonaws.eu-central-1.ecr.api
    com.amazonaws.eu-central-1.ecr.dkr
    com.amazonaws.eu-central-1.s3
    com.amazonaws.eu-central-1.logs
    com.amazonaws.eu-central-1.sts
    com.amazonaws.eu-central-1.elasticloadbalancing
    com.amazonaws.eu-central-1.autoscaling
    ```
1. Endpoint private access is required for nodes to register with the cluster endpoint. Endpoint public access is optional.

    ```
    $ aws eks describe-cluster --name <cluster name> | grep -i access
            "endpointPublicAccess": false,
            "endpointPrivateAccess": true,
            "publicAccessCidrs": []
    ```
1. VPC must have DNS hostname and DNS resolution support. Check VPC details to see DNS values to be enabled.

1. Subnet tagging requirement

    ```
    kubernetes.io/cluster/<cluster-name>                             shared
    kubernetes.io/role/internal-elb                                  1
    ```
1. A container image must be in or copied to Amazon Elastic Container Registry (**Amazon ECR**) or to a registry inside the VPC to be pulled.

1. Amazon EKS security group considerations for **Cluster security group**, **Control plane and node security groups** and **Working with tags on SG**

1. The aws-auth ConfigMap must be created from within the VPC. Managing users or IAM roles for your cluster

1. Configuration of HTTP proxy for Amazon EKS worker nodes with user data

1. Include the following text to the bootstrap arguments when launching self-managed nodes: Replace <cluster-endpoint> and <cluster-certificate-authority> with the values from your Amazon EKS cluster


## Reference Links

1. https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html
2. https://docs.aws.amazon.com/eks/latest/userguide/private-clusters.html
3. https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html
4. https://docs.aws.amazon.com/eks/latest/userguide/cluster-endpoint.html
5. https://docs.aws.amazon.com/vpc/latest/userguide/integrated-services-vpce-list.html



    
