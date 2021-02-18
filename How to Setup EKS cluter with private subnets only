## How to setup AWS EKS Cluster in Private Subnets Only

This topic describes how to deploy a private cluster without outbound internet access.

## Requirements

The following requirements must be met to run Amazon EKS in a private cluster without outbound internet access.

1. A container image must be in or copied to Amazon Elastic Container Registry (Amazon ECR) or to a registry inside the VPC to be pulled.

1. Endpoint private access is required for nodes to register with the cluster endpoint. Endpoint public access is optional.

1. You may need to include the VPC endpoints found at VPC endpoints for private clusters.

1. You must include the following text to the bootstrap arguments when launching self-managed nodes. This text bypasses the Amazon EKS introspection and does not require access to the Amazon EKS API from within the VPC. Replace <cluster-endpoint> and <cluster-certificate-authority> with the values from your Amazon EKS cluster.

```
--apiserver-endpoint <cluster-endpoint> --b64-cluster-ca <cluster-certificate-authority>
```

1. The aws-auth ConfigMap must be created from within the VPC. For more information about create the aws-auth ConfigMap, see Managing users or IAM roles for your cluster.
