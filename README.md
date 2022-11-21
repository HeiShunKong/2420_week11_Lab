# Lab Week 11

# Stephen Patricio and Hei Shun Kong

# Title : Readme File for creating droplets and transferring files from local to remote and getting weather information using curl

# Date :November 20 2022

# Description :

# 1. Create 2 droplets on Digital Ocean

a. Go to Digital Ocean and create a remote droplet named server-one and backup server and use the ssh key from the local machine

# 2. Create a key pair on your local machine and creating regular users on the remote server

a. In WSL, create a key pair using ssh-keygen -t ed25519 -C "email"
b. create a file where the keys can be stored by using the command /home/username/.ssh/filename
c. access server-one using the command: ssh -i ~/.ssh/FILE_NAME root@DIGITALOCEAN_DROPLET_IP_ADDRESS
d. add a user by using the command: useradd -ms /bin/bash USER_NAME
e. add the user to the sudo group by using the command: usermod -aG sudo USER_NAME
f. create a password for the user by using the command: passwd USER_NAME
g. modify ownership by using the command:rsync --archive --chown=USER_NAME:USER_NAME ~/.ssh /home/USER_NAME
h. switch to the user by using the command: ssh -i ~/.ssh/FILE_NAME USER_NAME@DO_DROPLET_IP_ADDRESS
i. modify the config file by using the command: sudo vi /etc/ssh/sshd_config
j. change the line PermitRootLogin prohibit-password to PermitRootLogin no
k. restart by using the command: sudo systemctl restart ssh
l. update by using the command: sudo apt update and sudo apt upgrade

# 2. Transfer the file from the local machine to the remote machine

a. In WSL, transfer the file from the local machine to the remote machine by using the command: sftp -i .ssh/DO2_key USER_NAME@DO_IP_ADDRESS
b. transfer folder from the local machine to the remote machine by using the command: put -r FOLDER_NAME
c. if only files, remove -r in the command

# 3. Create a weather script

#!/bin/bash

curl -s wttr.in/Vancouver -o /etc/motd

# 4. Create a weather service file

[Unit]
Description=use curl and wttr to get the weather everday at 05:00

[Service]
Type=oneshot
ExecStart=/opt/wthr/wthr

[Install]
WantedBy=multi-user.target

# 5. Create a weather timer file

[Unit]
Description=Timer to start the gtwtr service which gets the weather everday at 05:00

[Timer]
OnCalendar=Fri _-_-\* 05:00:00
RandomizedDelaySec=10000
Persistent=True
Unit=backup-service=.service

[Install]
WantedBy=Timers.target

# 6. Create a backup script

#!/bin/bash

curl -s wttr.in/Vancouver -o /etc/motd

. /opt/backup/backup.config

rsync -aucz -e "ssh -i ~/.ssh/backup-server" ${backup_data} root@${droplet_ip}:~/backup

# 7. Create a config file and write scripts inside

backup_data=/root/Documents
droplet_ip=123.123.123
export {backup_data, droplet_ip}

# 8. Create a backup service file

[Unit]
Description = back up files from server-one to the backup server using rsync

[Service]
Type=oneshot
ExecStart=/opt/backup/backup-script

[Install]
WantedBy=multi-user.target

# 9. Create a backup timer file

[Unit]
Description=Timer to start the gtwtr service which gets the weather everday at 1:00

[Timer]
OnCalendar=Fri _-_-\* 01:00:00
RandomizedDelaySec=10000
Persistent=True
Unit=backup-service=.service

[Install]
WantedBy=Timers.target

# 10. Set up sevice
a. Go to /etc/systemd/system directory using cd /etc/systemd/system.
b. Put backup.service and backup.timer in the new directory.
c. Activate both the backup.service and backup.timer service. Using sudo systemctl enable backup.service and sudo systemctl enable backup.timer
d. To check the status of the services run sudo systemctl status backup.service and sudo systemctl statsus backup.timer
  
