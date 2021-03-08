### Software Installation Steps

1. Install __kubectl__ 

    ```console
    For Mac
    $ curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.9/2020-11-02/bin/darwin/amd64/kubectl
    $ chmod +x ./kubectl
    $ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bash_profile
    $ kubectl version --short --client

    For Linux 
    $ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.8/bin/linux/amd64/kubectl && chmod u+x kubectl && mv kubectl /bin/kubectl
    $ kubectl version --short --client
    ```
1. Install EKSCTL

    ```
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version
    ```

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

1. Install  **Kubergrunt** 

    ```console    
    
    For Linux

    $ curl -L https://github.com/gruntwork-io/kubergrunt/releases/download/v0.6.9/kubergrunt_linux_amd64 > kubergrunt 
    $ chmod +x kubergrunt
    $ sudo mv kubergrunt /usr/local/bin/
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
    
1. Install AWS CLI in Linux 
    ```console
    
    For AWS CLI
    
    $ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64-2.0.1.zip" -o "awscliv2.zip"
    $ unzip awscliv2.zip
    $ sudo ./aws/install
    
    For Custom Path Installation
    
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64-2.0.30.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update

    sudo rm /usr/local/bin/aws
    sudo rm /usr/local/bin/aws_completer
    sudo rm -rf /usr/local/aws-cli
    ```
    
1. Installing AWS IAM authenticator ---> (This allows us to administor our EKS Clustor, using our IAM Identity)

    https://github.com/kubernetes-sigs/aws-iam-authenticator

    We are using the AWS Version of IAM authenticator ---> https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html
    ```
    curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.12.7/2019-03-27/bin/linux/amd64/aws-iam-authenticator

    chmod +x ./aws-iam-authenticator

    mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$HOME/bin:$PATH
    ```

1. Install Python version
    ```
    wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz
    tar xzf Python-3.6.8.tgz
    cd Python-3.6.8
    sudo ./configure --enable-optimizations
    sudo make altinstall
    python3.6 -V
    ```
1. Install GIT
    ```
    sudo yum install git -y
    ```
1. Install Jq
    ```
    sudo yum install jq
    ```
1. Install OpenJDK
    ```
    sudo amazon-linux-extras install java-openjdk11
    ```
1. How to install Ansible 2 on AWS Linux 2 (EC2)
    ```
    $ sudo yum update -y
    $ sudo amazon-linux-extras install ansible2 -y
    $ ansible --version
    ```
1. Install dig in Linux 
    ```
    sudo yum install bind-utils
    ```
1. Install 
    ```

    ```
1. Install 
    ```

    ```
1. Install 
    ```

    ```
