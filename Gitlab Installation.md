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
Postfix configuration

- Choose: *Internet Site*
- System mail name: your domain (e.g. gitlab.example.com)

These dependencies are required by GitLab Omnibus packages.

## Add the Official GitLab CE Repository
```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```

## Set the External URL
```
sudo EXTERNAL_URL="http://gitlab.example.com" apt install gitlab-ce
```

Examples:

- Domain: http://gitlab.example.com
- IP only: http://192.168.1.10

During installation:

- GitLab installs all components (PostgreSQL, Redis, NGINX)
- Runs initial configuration automatically

## Initial GitLab Configuration
```
sudo gitlab-ctl reconfigure
```
This command:

Applies configuration from /etc/gitlab/gitlab.rb
Restarts services safely

GitLab Omnibus uses this as its primary configuration mechanism.
