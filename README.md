# VPS-Cloudpanel-Wordpress-Setup (Win 11)
Version: 1.1\
Date: 30 Jan 2025

I have been deploying many VPSs for varied use-cases for past years and thought it was time to document the steps I use for my own reference. The idea is to spin up a basic VPS up, layer some security and allow me easy access to the server from my PC (Windows 11) and other devices I own. For this project, I want to set up a Wordpress website using a free control panel. Been using Cloudpanel and Cloudflare with no issues for the past year, am familiar with them and decided to use them for this project. And if these steps can help others navigate their DIY adventures, that would be great too! :)

+ Main workstation: Windows 11 PC
+ Terminal: Powershell 7.x
+ 1 x Admin/User use-case (me)
+ 1 x main SSH key (mine)
+ VPS (harderned)
+ DNS management via Cloudflare (+ Origin Certificates issuance)
+ Cloudpanel as the control panel under reverse proxy (to achieve clp.domain.com as my panel's login)
+ Cloudpanel default stack is LEMP (Nginx)
+ Installation of a Wordpress (hardened)

## Select a VPS (Virtual Private Server) Provider

Shortlisted options are all with SG-located datacenter. Going for a minimum of 2GB Ram.

**Digital Ocean:**
+ ~ SGD 11/mth: 1 vCPU | 1GB Ram | 35 GB NVMe SSD
+ ~ SGD 22/mth: 1 vCPU | 2GB Ram | 70 GB NVMe SSD
+ ~ SGD 23/mth: 2 vCPU | 2 GB Ram | 90 GB NVMe SSD

**Hetzner:**
+ ~ SGD 14/mth: 2 vCPU | 2 GB Ram | 40 GB NVME SSD

**OVH SG:**
+ VLE-2: ~ SGD 8/mth : 2 vCPU | 2 GB Ram | 40 GB NVME SSD
+ VLE-4: ~ SGD 16/mth : 4 vCPU | 4 GB Ram | 40 GB  

**Vultr:**
+ ~ SGD 8/mth: 1 vCPU | 1 GB Ram | 25 GB NVMe SSD
+ ~ SGD 16/mth: 1vCPU | 2GB Ram | 50 GB NVMe SSD
+ ~ SGD 25/mth: 2vCPU | 2GB Ram | 60 GB NVMe SSD

For this test deployment, am using Digital Ocean since I have $200 free referral credits to use up within the next 60 days :)\
Get your free $200 credit here: https://m.do.co/c/97213212086d\

For the production-ready site, may go for somewhere else or stick to DO, will see..

## Install Powershell 7.5 / 7.x

+ Install Powershell 7.x from Windows Store
+ Other methods of installation and guide here:
https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5
+ Start PowerShell

![image](https://drive.google.com/uc?export=view&id=130cV5uuvJmtIihQX7jkdeLXtvQ2g4uOT)

## SSH Keys Setup

#### Generate Keys via PowerShell

```
ssh-keygen -t ed25519 -f C:/Users/{user}/.ssh/sfarhan-key -C ""
```
Passphrase: xxxxxx

![image](https://drive.google.com/uc?export=view&id=13531RhTEtJX2wiwf-V-lk9U659fFpYFZ)

#### Check Windows User ~/.ssh folder

```
ls
```
![image](https://drive.google.com/uc?export=view&id=135mjes1PrPOyMhpw_DBJEWFcAr7RG7Vk)

> [!NOTE]
> + Can add any nickname/comment after -C (eg: -C homepc)
> + -C "" will not have any comments in the keys
> + Can remove the private keys from local directory after adding key to [ssh-add] setup and store it at Bitwarden for safekeeping

**All keys are autosave to: C:/Users/{user}/.ssh folder**

## Set up [sshd] service

#### Set the sshd service to start automatically

Open PowerShell as **Administrator** and enter:

```
Get-Service -Name sshd | Set-Service -StartupType Automatic
```

#### Start the sshd service
```
Start-Service sshd
```

![image](https://drive.google.com/uc?export=view&id=136DHsa25Js9P4rv4stwiRw_76d1khBZR)

## Setup [ssh-agent]

+ This setup and configuration for Windows is for [ssh-agent] to securely store the private keys within Windows security context, associated with Windows account
+ And to start the [ssh-agent] service each time the computer is rebooted (to start automatically)
+ By default the [ssh-agent] service is diabled

**Run Powershell as Administrator**

#### Get ssh-agent to start automatically
```
Get-Service ssh-agent | Set-Service -StartupType Automatic
```

#### Start the service: 
```
Start-Service ssh-agent
```

#### This should return a status of Running
```
Get-Service ssh-agent
```

#### Load private key onto [ssh-agent]:

```
ssh-add $env:USERPROFILE\.ssh\sfarhan-key
```
![image](https://drive.google.com/uc?export=view&id=138QwWijpZyN3PJkwXnm6ERj2uRiX5L6i)

> [!NOTE]
> + The ssh-agent in Windows will now automatically retrieve the local private key and pass it to SSH client.
> + If need be, can create new set of keys for each devices (Tablet, Phone) to SSH in.
> + Then standby to add those public keys onto the server when ready.

<p></p>

## Initiate & Deployment of Server

#### Password-less SSH
+ For Digital Ocean, Hetzner, Vultr or any other VPS providers, if there is an option to “park” public keys at their console, use the feature.
+ If not, deploy with root password.
+ If add an SSH key, no root credentials will be sent via email.

#### In Digital Ocean, the settings are under "My Account" -> "Manage Team Settings" -> "Security" -> "Add SSH Key"

![image](https://drive.google.com/uc?export=view&id=13DVacST8KJNqqWrJv9beCkuZdujufuAu)

#### After adding public key:

![image](https://drive.google.com/uc?export=view&id=13DlXEGmsVuoHIssaARsa7rmqV28DH6z8)

#### When creating server, can now have option to add the public key for a password-less entry

![image](https://drive.google.com/uc?export=view&id=13FmLD46ZIauh1qt87cei8jwHP0o8Fm0i)

## Server created!

![image](https://drive.google.com/uc?export=view&id=13HNaW5E5zAfEmNmQ7GQlvGcHa_ix781y)

## SSH Config file in Windows

We did rename the key to some other name than the default names (id_rsa etc..), so, useful to add some config info for Windows:

#### Create new Config file for User

Run Powershell as **Administrator** and navigate to .ssh folder and create the file.

```
cd C:/Users/{user}/.ssh
```
```
code config
```
or 
```
New-Item config
```

### Specifying Identityfile for Windows/Client SSH side of things:

Add the following to the config file:
```
# this is main user access
Host dosvr1
    Hostname {IP address}
    Identityfile C:/Users/{user}/.ssh/sfarhan-key
```

> [!Note]
> + Can replace Host (which is an alias, sort of shortcut name) to anything
> + Hostname is usually IP address or domain
> + IdentityFile is the path to the private keys
> + Can use Notepad to open and edit the config file from Windows Explorer
> + To use the same config file and configure change of SSH port later

Save file and Exit terminal

#### Extras:
Other examples of configs:

```
# Config for use specific key for github
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_ed25519_github
IdentitiesOnly yes

# For server 172.x.x.x
Host 172.x.x.x
User user
Port 2121
IdentityFile ~/.ssh/id_ed25519
IdentitiesOnly yes

# For all other servers
Host *
User root
```

## SSH via root access:

From Powershell:
```
ssh root@dosvr1
```

![image](https://drive.google.com/uc?export=view&id=13JILgMEERSxFgLuyzCLfT7G5-ENkd7Bb)

**:+1: VPS is up with root access and for easy and secured entry from Desktop PC.**

## Initial Housekeeping

```
apt update && apt upgrade
```
```
apt-get autoremove && apt-get autoclean
```
### Change server hostname
Realised that my original input for a hostname when spinning up the server was actually quite long (...@do-clp-wp-svr1) :)
Decided to change it to something shorter
```
hostnamectl set-hostname dosvr1
```

### Verify change takes effect
```
hostname
```
and
```
cat /etc/hostname
```
![image](https://drive.google.com/uc?export=view&id=13rhitH7hncm1_IN5HsVkCYgL2ayuKAOJ)

### Reboot server (and log back in to continue)
```
shutdown -r now
```

### Change System Time / Time Zone:
```
dpkg-reconfigure tzdata
```
Follow on screen instructions to set timezone. 

### Restart cron to ensure system picks up the change
```
service cron restart
```

### Disable unattended-upgrades:
```
dpkg-reconfigure unattended-upgrades
```
or can remove it completely:
```
apt remove unattended-upgrades
```
### Check SSH authorized_keys file

Ensure that the 2 public keys are in the /root/.ssh folder

![image](https://drive.google.com/uc?export=view&id=13wlVDx-7IzAv9cODGNNyDyUf-m2X9VGV)

If not..

### Create folders for .ssh and a file for authorized_keys:
```
cd /root
mkdir .ssh
cd .ssh
nano authorized_keys
```

Then put the relevant public keys in here.\
If need to add additional public keys (eg: for Laptop/Tablet/Phone devices entry points), can add them in now.

### Setup ownership and permissions:
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R root:root ~/.ssh
```
### Restart SSH service
```
$ service ssh restart
```
### Restart the sshd daemon, still as root, with:
```
systemctl restart sshd
```

> [!Note]
> All users (root and other users) all share the same config in /etc/ssh/sshd_config, but they don't all share the same 'authorized_keys' files. Thus, even when using same set of keys, need a seperate authorized_keys file in /root/.ssh/ and for
in /home/yournameuser/.ssh/ but in this case contains same set of public keys_

## Create New user

### Create sudo user
```
adduser user
usermod -aG sudo user
usermod -aG adm user
```
### Create the .ssh folder for new sudo user if it doesn't already exist:
```
mkdir /home/{user}/.ssh
```
### Copy the authorized_keys file that contains the public keys:
```
cp /root/.ssh/authorized_keys /home/{user}/.ssh/authorized_keys
```

### Setup ownership and permissions:
```
chown -R {user}:{user} /home/{user}/.ssh
chmod 700 /home/{user}/.ssh
chmod 600 /home/{user}/.ssh/authorized_keys
```
**Now the new sudo user has the public keys safely in their own authorized_keys file**

![image](https://drive.google.com/uc?export=view&id=13xGpj9uI-0o02swOXNLHBgflWjBRwEOw)

### Restart ssh
```
service ssh restart
```

## SSH in via sudo user

Open new Powershell terminal
```
ssh sfarhan@dosvr1
```
_Note:
The "dosvr1" is what we defined in the Windows side of things /.ssh config file above_

![image](https://github.com/user-attachments/assets/6264efc1-fa13-4894-a038-21a3a0df0742)

> [!NOTE]
> Test a few sudo commands to ensure all is okay, then next step is to disable root login and securing SSH access

############### #################

## Securing SSH Accecss

The idea is to:
+ Change SSH port from 22 to another (here I am using 22022)
+ Disable RootLogin
+ Disable Entry points via Passwords (only SSH keys)

#### Backup original config file
```
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

#### Make changes to file
```
sudo vim /etc/ssh/sshd_config
```
```
Port 22022
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
```
```
sudo service ssh restart
```

>[!NOTE]
> In Ubuntu 24.04, behavior is slightly different than what I am used to for changing of SSH port ie: change /etc/ssh/sshd_config file to indicate a new port #, then restart the service. There are additional steps in Ubuntu 24.04 it seems.
> If not changing the port, can skip some of the steps below

#### Activate new config (for the SSH port part)

To activate this new config, it is now required **to inform systemd about the change**:
```
sudo systemctl daemon-reload
```
Then the ssh service and socket can be restarted, to **activate the change**:
```
sudo systemctl restart ssh.socket
sudo systemctl restart ssh.service
```

#### Verify new SSH port number is listening:
```
sudo netstat -tuln | grep :22022
```

> [!IMPORTANT]
> Before logging off, need to setup new port in UFW (Firewall) settings and add new SSH port in Windows SSH config file
> Do not close the existing session! and continue next steps

### Add Rules in UFW

#### Start UFW

UFW is already pre-installed with Ubuntu 24.04 but disabled. 

```
sudo ufw enable
```

#### Check UFW is running
```
sudo ufw status
```

#### Establish Default Rule
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
```

#### Allow new SSH port
```
sudo ufw allow 22022/tcp
sudo ufw allow out 22022/tcp
```

#### Add other basic rules for UFW
```
sudo ufw allow http/tcp 
sudo ufw allow https/tcp
sudo ufw logging off # or sudo vim /etc/ufw/ufw.conf (LOGLEVEL=off).
```

#### Check UFW rules
```
sudo UFW status
```

INSERT IMAGE HERE!

> [!NOTE]
> To remove port 22/tcp only AFTER Cloudpanel is installed and all okay!
> If remove now, it is okay too, then will need to add the port info during installation process

### Add new SSH port in Window's SSH config file:

Open the config file via Notepad and add the port info. File is at C:/Users/{user}/.ssh/config
```
# this is main user access
Host dosvr1
    Hostname {IP address}
    Port 22022 # <-this
    Identityfile C:/Users/{user}/.ssh/sfarhan-key
```
Save file

> [!IMPORTANT]
> Before logging off, Start a new terminal to SSH in to test

#### Open new terminal (without closing the current session)

Log in into server as usual without need to specify port info:

## Fail2ban

$ sudo ufw default allow outgoing
$ sudo ufw default deny incoming

sudo ufw allow 22022/tcp
sudo ufw allow out 22022/tcp

Configure UFW
$ sudo ufw allow http/tcp 
$ sudo ufw allow https/tcp
$ sudo ufw allow bootpc/udp
$ sudo ufw allow ssh/tcp
$ sudo ufw allow 22022/tcp #this is for new ssh port
$ sudo ufw logging off or $ sudo vim /etc/ufw/ufw.conf (LOGLEVEL=off).
$ sudo ufw enable

Do this after Install Cloudpanel:
Deny 22/tcp for SSH:
$ ufw deny 22/tcp
(or go to Cloudpanel control panel to delete them)

Check: sudo ufw verbose
Check: sudo ufw status

Example Final look to UFW settings:

Create/Test new terminal session with port 22022 (do not exit current session)

Test multiple times. If all okay and after install Cloudpanel:

$ sudo ufw delete allow 22/tcp
$ sudo ufw delete allow out 22/tcp

#### Change PermitRootlogin config to no:
$sudo vim /etc/ssh/sshd_config




Fail2ban Install:

$ sudo apt-get update
$ sudo apt-get install fail2ban -y

Create new jail.local

"Every .conf file can be overridden with a file named .local. The .conf file is read first, then .local, with later settings overriding earlier ones. Thus, a .local file doesn't have to include everything in the corresponding .conf file, only those settings that you wish to override. Modifications should take place in the .local and not in the .conf. This avoids merging problem when upgrading. These files are well documented and detailed information should be available there."

Create new jail.local file
$ sudo touch /etc/fail2ban/jail.local

Edit the file:
$ sudo nano /etc/fail2ban/jail.local
clear
Variables to Change in jail.local:

[DEFAULT]

ignoreip = 127.0.0.1/8 {home or office IP address}
findtime = 3200
bantime = 86400

[sshd]
backend=systemd #this is the workaround for fail2ban not working in Debian
enabled = true
port = 22022
filter = sshd
logpath = /var/log/auth.log
maxretry = 3

# Add these below after install WP Fail2ban plugin

[wordpress-hard]
enabled = true
filter = wordpress-hard
logpath = /var/log/auth.log
maxretry = 3
port = http,https

[wordpress-soft]
enabled = true
filter = wordpress-soft
logpath = /var/log/auth.log
maxretry = 3
port = http,https                                                                                                                        

[wordpress-extra]
enabled = true
filter = wordpress-soft
logpath = /var/log/auth.log
maxretry = 3
port = http,https

To add new jails configs for Wordpress / Mysqp  PHPadmin / Apache / Nginx Bots etc…
https://webdock.io/en/docs/how-guides/security-guides/how-configure-fail2ban-common-services

Run this command:
$ sudo apt install python3-systemd

Then:
$ sudo systemctl restart fail2ban
$ sudo service fail2ban restart

Check the new rules by typing:
$ iptables -S
$ iptables -L

$ sudo fail2ban-client status sshd

To check the logs:
$ tail -f /var/log/auth.log
$ tail -f /var/log/fail2ban.log

For Debian12: to check tail -f auth.log:
$ sudo journalctl -u ssh.service

manually unban IP addresses 
sudo fail2ban-client set <jail> banip/unbanip <ip address>
# For example
sudo fail2ban-client set sshd unbanip 83.136.253.43

Check all the open network ports and their associated services

sudo netstat
can also use the Nmap command in order to discover hosts and services. 
nmap domain.com

## Installing Control Panel

We have a few choices, Cloudpanel, HestiaCP, Webadmin etc.. Here I am documenting the steps I take for using Cloudpanel for the Wordpress installation

 #### Install Cloud Panel
 
 https://www.cloudpanel.io/docs/v2/getting-started/other/

curl -sS https://installer.cloudpanel.io/ce/v2/install.sh -o install.sh && sudo bash install.sh --ssh_port <your_new_port> 

$ apt-get autoremove && apt-get autoclean

https://yourIpAddress:8443

Create new Firewall Rule in Cloudpanel
Add:
Custom SSH port: 22022
For ip4: 0.0.0.0/0
For ip6: ::/0

Delete Port 22

Add FTP for Port 20-21

sudo systemctl restart fail2ban

Cloudflare

Add Records for:

domain.com
{IP Address}
Proxied

clp
{IP Address}
Turn Off Proxy First

CNAME
www
domain.com
Proxied

Setting up subdomain for Cloudpanel

To upload Cloudflare Origin Certificate for clp.amnotadev.com (to log into the panel), do a CloudPanel Custom Domain via Reverse Proxy

Go to Sites and create a Reverse Proxy

Enter the Domain Name as clp.domain.com, enter https://127.0.0.1:8443 as Reverse Proxy Url.

Make sure the https is the url

User: reverseproxy (this is just to contain the /home docs)
Password: xxxxxxx

Do not close CF origin cert page yet, we need the keys to copy paste into Cloudpanel

Import CF origin certificate

https://clp.domain.com will bring to CloudPanel login page

If after installing Cloudpanel, fail2ban fails..

Here is a workaround:
wget https://launchpad.net/ubuntu/+source/fail2ban/1.1.0-1/+build/28291332/+files/fail2ban_1.1.0-1_all.deb
sudo dpkg -i fail2ban_1.1.0-1_all.deb
https://bugs.launchpad.net/ubuntu/+source/fail2ban/+bug/2055114/comments/22
https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-ubuntu-16-04

Server Security & Server Panel DONE!

TIME TO INSTALL WORDPRESS!!

Install /Run Wordpress on test.domain.com

test.domain.com

Adjust PHP Settings

Add CF's Cert

Allow from CF only

Enter A (AAAA) records + CNAME in CF

MYSQL:

Host: 127.0.0.1

User Name: root

Password: xxxxxxx

Port: xxxx

Plugins: 

Limit Login Attempts Reloaded

Wordfence: 

# Block WordPress xmlrpc.php requests
<Files xmlrpc.php>
order deny,allow
 
deny from all
allow from 123.123.123.123
</Files>

Use Wordfence to scan:

if they found a file that needs to hide but unable to hide at UI level, go to Cloudflare-> under Vhost and add:

location ~ \.user\.ini$ {
deny all;
}

Plugin: WP Fail2ban

Explore the plugin conf files at htdocs. find the appropriate conf file to copy over to fail2ban filter.d directory

clp /home/$User/htdocs/domain.com/wp-content/plugins/wp-fail2ban/filters.d/wordpress-hard.conf /etc/fail2ban/filter.d/ 

and also the -soft and the -extra conf files

add the filter to jail.local to activate the ban

vim /etc/fail2ban/jail.local

[wordpress-hard]

enabled = true
filter = wordpress-hard
logpath = /var/log/auth.log
maxretry = 3
port = http,https

[wordpress-soft]
enabled = true
filter = wordpress-soft
logpath = /var/log/auth.log
maxretry = 3
port = http,https

[wordpress-extra]
enabled = true
filter = wordpress-soft
logpath = /var/log/auth.log
maxretry = 3
port = http,https

At jail.local conf file, under wordpress filters/jail header, Use /home/sf-wp-admin /logs and /home/adco-wp-admin

exit

Limit Login Attempts Reloaded






