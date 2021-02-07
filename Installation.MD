### Software Installation Steps

#### Helm 2
helm repo add aws https://aws.github.io/eks-charts
helm install --name my-aws-load-balancer-controller aws/aws-load-balancer-controller --version 1.1.3

#### Helm 3
helm repo add aws https://aws.github.io/eks-charts
helm install my-aws-load-balancer-controller aws/aws-load-balancer-controller --version 1.1.3
https://youtu.be/DMVS5PByxbg

