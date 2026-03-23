# Securing, Organizing, and Automating a Newly Deployed Linux Server

## Update the system, upgrade installed packages and enable firewall
'''bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y nginx git ufw
sudo ufw allow ssh
sudo ufw allow "Nginx Full"
sudo ufw enable
sudo systemctl start nginx
'''

- This should get our server ready and fulfill all conditions


## Identity and Access Control (Create Groups and Users) - Add users to their specific groups, ensure home directories, bash shell, and force password change on first login
'''bash
sudo groupadd backend_team
sudo groupadd infra_team
sudo groupadd risk_audit
sudo useradd -m -s /bin/bash chinedu_okafor
sudo useradd -m -s /bin/bash amara_nwosu
sudo useradd -m -s /bin/bash sadiq_bello
sudo useradd -m -s /bin/bash esther_adebayo
sudo useradd -m -s /bin/bash femi_ogunleye
sudo passwd chinedu_okafor
sudo passwd amara_nwosu
sudo passwd sadiq_bello
sudo passwd esther_adebayo
sudo passwd femi_ogunleye
sudo chage -d 0 chinedu_okafor 
sudo chage -d 0 amara_nwosu 
sudo chage -d 0 esther_adebayo 
sudo chage -d 0 sadiq_bello 
sudo chage -d 0 femi_ogunleye 
sudo usermod -aG backend_team chinedu_okafor
sudo usermod -aG backend_team amara_nwosu 
sudo usermod -aG sudo sadiq_bello 
sudo usermod -aG infra_team esther_adebayo 
sudo usermod -aG risk_audit femi_ogunleye 
'''

-Sadiq was added to sudo as instructed
- These commands creates groups, creates users, gives users a password, forces the users to change password on first login and apportion the right users to their groups respectively


## Creating a Controlled Workspace Setup
'''bash
sudo mkdir -p /opt/vertexcore/{backend_services,infrastructure,compliance_logs,shared_resources}
cd /opt/vertexcore
sudo chown -R root:backend_team backend_services
sudo chown -R root:risk_audit compliance_logs
sudo chown -R root:infra_team infrastructure
sudo chown -R root:infra_team shared_resources
sudo chmod 770 backend_services
sudo chmod 770 infrastructure/
sudo chmod 775 shared_resources/
sudo apt install acl -y
sudo chmod 770 compliance_logs/
sudo setfacl -m g:risk_audit:rx backend_services
sudo setfacl -m g:risk_audit:rx infrastructure/
sudo setfacl -m g:risk_audit:rx shared_resources/
'''

- you can use 'ls -l' to verify the commands
- I had to create an access control list for control 3
- Here are the 4 access control commands
- 1. backend_team should access only backend_services
- 2. infra_team: infrastructute+full storage access
- 3. risk_audit: full compliance_logs, read-only others
- 4. shared resources: everyone reads, infra_team write

## Advances File Permission ( creating a file and restrciting the file to backend_team only)
'''bash
sudo touch /opt/vertexcore/backend_services/.env.production
sudo chown -R root:backend_team /opt/vertexcore/backend_services/.env.production
sudo chmod 660 /opt/vertexcore/backend_services/.env.production
sudo ls -l /opt/vertexcore/backend_services/.env.production
'''

- since it is an .env file - its a hidden file which only a backend user should read and write

## Process and service control
'''bash
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl status nginx
ps aux | grep nginx
'''

# SSH Hardening
'''bash
sudo su - (user)
ssh-keygen
nano /etc/ssh/sshd_config
'''

- You nano into the file and disable root login anf password authentication
- This mean you can only login via pub keys
- You can check the public keys here ' ls - l ~/.ssh '

# Git Repository Management (branch and merge)
'''bash
git init
git add .
git commit -m " your first commmit"
git checkout -b feature_payment_api
git checkout master
git merge feature_payment_api"
'''

- Created a branch and merge of feature_payment_api

# Git Collaboration Simulation
'''bash
cd /opt/repos
git clone /opt/repos/payment_engine new_payment_engine
cd new_payment_engine
'''

- After cloning the repo, you can try out the branch and merge function.. it works the same

# BASH SCRIPTING
'''bash
touch daily_health_check.sh
nano daily_health_check.sh
  GNU nano 4.8                                                     daily_health_check.sh                                                     Modified  
#!/bin/bash

echo " ====systemcheck=== " >> /var/log/system_health.log
date >> /var/log/system_health.log
uptime >> /var/log/system_health.log
free -h >> /var/log/system_health.log
df -h >> /var/log/system_health.log

sudo chmod +x daily_health_check.sh
./daily_health_check.sh

crontab -e
30 0 * * * /vagrant/daily_health_check.sh 
'''
- This last command signifies 12:30 AM daily
- And the automation was to create a script that automatically does a system log check daily at 12:30 AM

