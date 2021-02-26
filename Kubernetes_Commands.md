## List of Kubernetes Commands


1. Kubernetes important commands
```
$ aws configure list
$ aws configure
$ aws eks update-kubeconfig --name eks-demo
$ kubectl config use-context arn:aws:eks:eu-central-1:723571234567:cluster/eks-demo


$ kubectl port-forward nginx-deployment-688df47648-xfmnb 8081:80
$ kubectl port-forward nginx-deployment-688df47648-xfmnb --address 0.0.0.0 8081:80

$ kubectl run multitool --image=61257*****.dkr.ecr.eu-central-1.amazonaws.com/network-multitool:latest
$ kubectl exec -it multitool -- sh


$ aws sts assume-role --role-arn arn:aws:iam::74105*****:role/environment-admin --role-session-name test43243 --serial-number arn:aws:iam::2916******:mfa/murugavel.ramachandran --token-code 123456


Database Connect

psql --host=dbserver-dev1.***********.eu-central-1.rds.amazonaws.com --port=5432 --username=rds_user --password --dbname=rdsdbdev1

```

1. Kubectl port forward 
```
  https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward
```
