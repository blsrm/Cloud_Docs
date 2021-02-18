## How to setup AWS EKS Cluster in Private Subnets Only

This topic describes how to deploy a private cluster without outbound internet access.

## Requirements

The following requirements must be met to run Amazon EKS in a private cluster without outbound internet access.

1. A container image must be in or copied to Amazon Elastic Container Registry (Amazon ECR) or to a registry inside the VPC to be pulled.

1. Endpoint private access is required for nodes to register with the cluster endpoint. Endpoint public access is optional.

1. You may need to include the VPC endpoints found at VPC endpoints for private clusters.

1. The aws-auth ConfigMap must be created from within the VPC. For more information about create the aws-auth ConfigMap, see Managing users or IAM roles for your cluster.

1. You must include the following text to the bootstrap arguments when launching self-managed nodes. This text bypasses the Amazon EKS introspection and does not require access to the Amazon EKS API from within the VPC. Replace <cluster-endpoint> and <cluster-certificate-authority> with the values from your Amazon EKS cluster.

```
--apiserver-endpoint <cluster-endpoint> --b64-cluster-ca <cluster-certificate-authority>
```

## Considerations
Here are some things to consider when running Amazon EKS in a private cluster without outbound internet access.

1. AWS X-Ray is not supported with private clusters.

1. Amazon CloudWatch Logs is supported with private clusters, but you must use an Amazon CloudWatch Logs VPC endpoint.

1. Self-managed and managed nodes are supported. The instances for nodes must have access to the VPC endpoints. If you create a managed node group, the VPC endpoint security group must allow the CIDR for the subnets, or you must add the created node security group to the VPC endpoint security group.

1. IAM roles for service accounts is supported. You must include the STS VPC endpoint.

1. The Amazon EBS CSI driver is supported. Before deploying, the kustomization.yaml file must be changed to set the container images to use the same Region as the Amazon EKS cluster.

1. The Amazon EFS CSI driver is supported. Before deploying, the kustomization.yaml file must be changed to set the container images to use the same Region as the Amazon EKS cluster.

1. The Amazon FSx for Lustre CSI driver is not supported.

1. AWS Fargate is supported with private clusters. You must include the STS VPC endpoint. For more information, see VPC endpoints for private clusters. You can use the AWS load balancer controller to deploy AWS Application Load Balancers and Network Load Balancers with. The controller supports network load balancers with IP targets, which are required for use with Fargate. For more information, see Application load balancing on Amazon EKS and Load balancer – IP targets.

1. App Mesh is supported with private clusters when you use the App Mesh Envoy VPC endpoint.

