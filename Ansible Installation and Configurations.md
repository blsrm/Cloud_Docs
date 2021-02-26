### This section describes how to install and configure Ansible for IaaC

I. Installation is varrying from flavour of Linux OS
  ```
  1. Launch RED HAT Linux in EC2 instance
  
  2. Update your EC2 instance
  
  3. Add a third party repositories named EPEL (Extra Packages for Enterprise Linux)
    # rpm -Uvh https://dl.fedoraproject.org/pub/epel-release-latest-7-noarch.rpm
  
  4. Install Ansible
    # yum install ansible

  Testing
  
  $ ansible --version
  
  ```

II. Configuration
  ```
  1. Create new user for ansible administration and grant admin access to user (Master & Slave)
    # useradd ansadmin
    # passwd ansadmin
  
  2. Enable user login on all EC2 instances (Master & Slave)
    # vi /etc/ssh/sshd_config
  
  3. Login as a ansadmin user on master and generate ssh key (Master)
    # ssh-keygen
  
  4. Copy keys to target servers (Master)
    # ssh-key-id <target servers>
  
  5. Update target servers information on /etc/ansible/hosts file (Master)
    # echo "10.10.*.*" > /etc/ansible/hosts
  
  Testing 
  
  run ansible command as a ansadmin user it should be successful (Master)
  # ansible all -m ping
    
  ```

III. Ansible Simple commands
  ```
  Rebooting     - ansible all -a "/sbin/reboot"
  Copy file     - ansible all -m file -a "src=/home/dan dest=/tmp/home"
  Create User   - ansible all -m user -a "name=testuser passwor=<encrypted password>"
  Remove User   - ansible all -m user -a "name=testuser state=absent"
  Change File Permission  - ansible all -m file -a "dest=/home/dan/file1.txt mode=777"
  Install package - ansible all -m yum -a "name=httpd state=latest"
  Start a service - ansible all -m service -a "name=httpd state=started"
  Stop a Service  - ansible all -m service -a "name=httpd state=stopped"
  
  ```
