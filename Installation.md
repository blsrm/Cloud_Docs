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
    
