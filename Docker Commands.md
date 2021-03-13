## Docker Commands

Step 1: Create a Docker image
```
touch Dockerfile
Edit the Dockerfile
docker build -t hello-world .
docker images --filter reference=hello-world
docker run -t -i -p 80:80 hello-world
docker-machine ip machine-name
```
Step 2: Authenticate to your default registry
```
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com

To describe the repositories in a registry
aws ecr describe-repositories

To list the images in a repository
aws ecr list-images --repository-name app-repo --region eu-central-1
```

Step 3: Create a repository
```
aws ecr create-repository \
    --repository-name hello-world \
    --image-scanning-configuration scanOnPush=true \
    --region us-east-1

aws ecr create-repository --repository-name app-folder/app-name --region eu-central-1 --image-scanning-configuration scanOnPush=true
```

Step 4: Push an image to Amazon ECR
```
docker images
docker tag hello-world:latest aws_account_id.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest

docker tag app-folder/app-name 61257*******.dkr.ecr.eu-central-1.amazonaws.com/app-folder/app-name:latest

docker push aws_account_id.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest

docker push 61257******.dkr.ecr.eu-central-1.amazonaws.com/app-folder/app-name:latest
```

Step 5: Pull an image from Amazon ECR
```
docker pull aws_account_id.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest
```

Step 6: Delete an image

```
aws ecr batch-delete-image \
      --repository-name hello-world \
      --image-ids imageTag=latest
```

Step 7: Delete a repository
```
aws ecr delete-repository \
      --repository-name hello-world \
      --force
      
aws ecr list-images --repository-name app-folder/app-name --region eu-central-1
docker pull 612573*******.dkr.ecr.eu-central-1.amazonaws.com/app-folder/app-name:latest
```

Sample Docker Repository and IMAGE Creation
```
aws ecr create-repository --repository-name app/busybox --region eu-central-1 --image-scanning-configuration scanOnPush=true
aws ecr list-images --repository-name app --region eu-central-1


docker build -t oss/oss-app1:latest -f spring-boot/src/main/docker/Dockerfile .
aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 61257********.dkr.ecr.eu-central-1.amazonaws.com
docker tag oss/oss-app1 61257********.dkr.ecr.eu-central-1.amazonaws.com/oss/oss-app1:latest
docker push 61257********.dkr.ecr.eu-central-1.amazonaws.com/oss/oss-app1:latest
docker image prune -f
```
