### Software Installation Steps

#### Helm 2
´´´
helm repo add aws https://aws.github.io/eks-charts
helm install --name my-aws-load-balancer-controller aws/aws-load-balancer-controller --version 1.1.3
´´´
#### Helm 3
´´´
helm repo add aws https://aws.github.io/eks-charts
helm install my-aws-load-balancer-controller aws/aws-load-balancer-controller --version 1.1.3
https://youtu.be/DMVS5PByxbg
´´´

1. Install  **Terragrunt** 

    ```console    
    
    For Mac:

    $ curl -o terragrunt -LO https://github.com/gruntwork-io/terragrunt/releases/download/v0.26.7/terragrunt_darwin_amd64

    $ chmod +x terragrunt

    $ mv terragrunt /usr/local/bin


    For Linux

    $ curl -o terragrunt -LO https://github.com/gruntwork-io/terragrunt/releases/download/v0.26.7/terragrunt_linux_amd64

    $ chmod +x terragrunt

    $ mv terragrunt /usr/local/bin

    ```


1. Install **Terraform** (version 0.13.5) using **tfenv**. With tfenv you can maintain multiple version of terraform

    ```console
    For Mac
    
    $ tfenv install 0.13.5

    $ tfenv use 0.13.5

    You can pin the version you want in your bash profile as the default

    $ echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bash_profile

    For Linux
    
    $ curl -o terraform.zip -OL https://releases.hashicorp.com/terraform/0.13.5/terraform_0.13.5_linux_amd64.zip

    $ unzip terraform.zip -d /usr/local/bin
    
    ```
    
