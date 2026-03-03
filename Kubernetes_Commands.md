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

#### ACM internal certificate
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=test.com/O=test.com"
kubectl create secret tls tls-secret --key tls.key --cert tls.crt
aws acm import-certificate --certificate fileb://tls.crt --private-key fileb://tls.key --certificate-chain fileb://tls.crt --region eu-central-1
```

#### GCP Kuberntetes setup
```
google-cloud-sdk-gke-gcloud-auth-plugin is not available in default Ubuntu repositories.
It is provided only via Google’s Cloud SDK APT repo.

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates gnupg curl

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
| sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] \
https://packages.cloud.google.com/apt cloud-sdk main" \
| sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list

sudo apt-get update

sudo apt-get install -y google-cloud-sdk-gke-gcloud-auth-plugin

gcloud components list | grep gke-gcloud-auth-plugin

which gke-gcloud-auth-plugin


export USE_GKE_GCLOUD_AUTH_PLUGIN=True

echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc
source ~/.bashrc

gcloud container clusters get-credentials <CLUSTER_NAME> \
  --region <REGION> \
  --project <PROJECT_ID>

kubectl get nodes

gcloud version
sudo apt-get install -y google-cloud-sdk-gke-gcloud-auth-plugin
which gke-gcloud-auth-plugin
gke-gcloud-auth-plugin --version
export USE_GKE_GCLOUD_AUTH_PLUGIN=True
echo 'export USE_GKE_GCLOUD_AUTH_PLUGIN=True' >> ~/.bashrc
source ~/.bashrc

sudo apt-get install -y kubectl

gcloud container clusters get-credentials gke-vmo2-trg-test-cluster   --region europe-west2   --project gci-*****-pjpc-01n******
kubectl get nodes
```
