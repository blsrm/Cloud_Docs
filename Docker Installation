## Docker Installation on Ubuntu Server

## Step 1 Remove old Docker versions (if any)
```
sudo apt remove -y docker docker-engine docker.io containerd runc
```

### Step 2 Update package index & install prerequisites
```
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```
## Add Docker’s official GPG key
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
## Add Docker Repository
```
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Install Docker Engine
```
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Verify Docker installation
```
sudo docker version
sudo docker run hello-world
```

## (Optional but Recommended) Run Docker without sudo
```
sudo usermod -aG docker $USER
newgrp docker
```

## Verify
```
docker run hello-world
```

## Enable Docker at boot
```
sudo systemctl enable docker
sudo systemctl start docker
```
This error is expected when Docker is installed but your user is not allowed to access the Docker socket.

```
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```
Docker daemon runs as root.
Your user (murugavel_ramachandran) is not in the docker group.

## Add your user to the docker group
```
sudo usermod -aG docker murugavel_ramachandran
```

## Verify access
```
docker ps
docker pull gitlab/gitlab-ce
```


