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

- Choose: **Internet Site**
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

## Configure Firewall (UFW)
if UFW is enabled 
```
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw reload
```
Firewall rules are required for web and Git access.

## Access GitLab Web Interface
```
http://gitlab.example.com
```

## Set Initial Root Password
- First page prompts you to set a root password
- Default username: root

This is standard behavior for modern GitLab installations.

## Enable HTTPS (Let’s Encrypt – Optional but Recommended)
```
sudo nano /etc/gitlab/gitlab.rb

external_url "https://gitlab.example.com"
letsencrypt['enable'] = true
letsencrypt['contact_emails'] = ['admin@example.com']

sudo gitlab-ctl reconfigure
```

## Verify GitLab Services
```
sudo gitlab-ctl status

Restart all services if needed:
sudo gitlab-ctl restart

logs location

/var/log/gitlab/
```

## Important Configuration Files & Commands
Purpose          Location / Command
Main config      /etc/gitlab/gitlab.rb
Reconfigure      gitlab-ctl reconfigure
Status           gitlab-ctl status
Logs            /var/log/gitlab/
Backup          gitlab-backup create

## Docker‑based GitLab
```
docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --volume gitlab_config:/etc/gitlab \
  --volume gitlab_logs:/var/log/gitlab \
  --volume gitlab_data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

