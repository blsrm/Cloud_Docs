## Installing Self hosted Gitlab Community Edition on Ubuntu Server

## Update System
```
sudo apt update
sudo apt upgrade -y
```
## Install Required Dependencies
```
sudo apt install -y \
  curl \
  openssh-server \
  ca-certificates \
  postfix \
  tzdata \
  perl
```
