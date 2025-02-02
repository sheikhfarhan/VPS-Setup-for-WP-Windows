# VPS-Cloudpanel-Wordpress-Setup
Version: 1.1\
Date: 30 Jan 2025

// some intro goes here //



## VPS (Virtual Private Server)

Some options here. All with SG-located datacenter. Going for a minimum of 2GB Ram.

**Digital Ocean:**\
~ SGD 11/mth: 1 vCPU | 1GB Ram | 35 GB NVMe SSD\
~ SGD 22/mth: 1 vCPU | 2GB Ram | 70 GB NVMe SSD\
~ SGD 23/mth: 2 vCPU | 2 GB Ram | 90 GB NVMe SSD

**Hetzner:**\
~ SGD 14/mth: 2 vCPU | 2 GB Ram | 40 GB NVME SSD

**OVH SG:**\
VLE-2: ~ SGD 8/mth : 2 vCPU | 2 GB Ram | 40 GB NVME SSD\
VLE-4: ~ SGD 16/mth : 4 vCPU | 4 GB Ram | 40 GB  

**Vultr:**\
~ SGD 8/mth: 1 vCPU | 1 GB Ram | 25 GB NVMe SSD\
~ SGD 16/mth: 1vCPU | 2GB Ram | 50 GB NVMe SSD\
~ SGD 25/mth: 2vCPU | 2GB Ram | 60 GB NVMe SSD

For this test deployment, using Digital Ocean, since have $200 free referral credits to use up within the next 60 days :)\
Get your free $200 here with my referral code: https://m.do.co/c/97213212086d

## SSH Keys

Generate 2 sets of private+public keys (can use Putty, CLI via Windows Powershell or Termius):\
root-key\
yourown/user-key

## Generate keys via PowerShell or Termius app or Putty
### Using Powershell 7.x in Windows 11 as main client:

Install Powershell 7.x from Windows Store\
Open up PowerShell\
Generate SSH keys:

`ssh-keygen -t ed25519 -f C:/Users/$User/.ssh/root-key`\
Passphrase: xxxxxx

`ssh-keygen -t ed25519 -f C:/Users/$User/.ssh/admin-key`\
Passphrase: xxxxxx

All keys are autosave to: C:/user/$User/.ssh

_(when all is okay, can consider to remove the private keys from local PC and store it at Bitrwarden for safekeeping!)_

Close Terminal

## Set the sshd service to start automatically

Open PowerShell as **Administrator** and enter:

`Get-Service -Name sshd | Set-Service -StartupType Automatic`

## Now start the sshd service
`Start-Service sshd`

## Setup [ssh-agent]:

Setup [ssh-agent] to securely store the private keys within Windows security context, associated with Windows account + To start the [ssh-agent] service each time the computer is rebooted + Using [ssh-add] to store the private key (By default the [ssh-agent] service is disabled) and to configure it to start automatically.

**Run Powershell as Administrator**

#### Get ssh-agent to start automatically
`Get-Service ssh-agent | Set-Service -StartupType Automatic`

#### Start the service: 
`Start-Service ssh-agent`

#### This should return a status of Running
`Get-Service ssh-agent`

### Load key files into [ssh-agent]:
`ssh-add $env:USERPROFILE\.ssh\root-key`\
`ssh-add $env:USERPROFILE\.ssh\admin-key`

![image](https://github.com/user-attachments/assets/ca082e96-614b-4be4-aa40-b7183496c326)

The ssh-agent in Windows will now automatically retrieve the local private key and pass it to SSH client.\
Create new set of keys for each devices (Tablet, Phone) to SSH in. Then standby to add those public keys onto the server when ready.

## Initiate & Deployment of Server

For DO, Hetzner or any other VPS providers, if there is an option to “park” public keys at their console, use the feature.\
If not, deploy with root password.

![image](https://github.com/user-attachments/assets/ea987cd3-81eb-4212-8f06-e270c66950f4)

**Hostname: dosvr1**

## SSH in via Powershell/Windows as client

Since we rename the keys to some other name than the default names (id_rsa etc..), additional steps for Windows .ssh agent / client to SSH into the server:

### Config file for User in Windows

Run Powershell as **Administrator**

```
cd C:/Users/$USER/.ssh
```

Create config file (example if have VSCode installed)

```
code config
```

or 

```
type nul > config
```

Specifying Identityfile for SSH:

```
# Per-Host Per User basis config
Host dosvr1-root (to remove when the dust settles)
    Hostname {VPS IP Address} 
    User root
    Identityfile C:/Users/$USER/.ssh/root-key
    IdentitiesOnly yes

Host dosvr1-user
    Hostname {VPS IP Address} 
    User user
    Identityfile C:/Users/$USER/.ssh/admin-key
    IdentitiesOnly yes
```

Exit terminal

## SSH into the server via root access:

From Powershell:\
```
ssh dosvr1
```

Or

```
ssh root@{VPS IP address} -i ~/.ssh/root-key
```

![image](https://github.com/user-attachments/assets/e034de48-61e0-4d28-adb2-915fbdd1a6d0)


## Initial Housekeeping

$ apt update && apt -y upgrade && apt -y install curl wget
$ apt-get autoremove && apt-get autoclean
$ shutdown -r now

### Change System Time / Change Time and Time Zone:
$ dpkg-reconfigure tzdata

Follow on screen instructions to set timezone. 

### Restart cron to ensure system pick up the change
$ service cron restart

### Disable unattended upgrades:
$ dpkg-reconfigure unattended-upgrades

or can remove it completely:
$ apt remove unattended-upgrades

### Check SSH authorizedkeys file and the 2 public keys are in the /root/.ssh folder\
If not:

Create (if not already) folders for .ssh and a file for authorized_keys:
$ cd /root
$ mkdir .ssh
$ cd .ssh
$ vim authorized_keys

Then put the relevant public keys in here. If already have the public keys for Tablet/Phone devices entry points, can add them in now.

### Make the directory and file only executable by the root and setup ownership and permissions:

$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys
$ chown -R root:root ~/.ssh

$ service ssh restart

### Restart the sshd daemon, still as root, with:
$ systemctl restart sshd
$ exit

_Note:
All users (root and other users) all share the same config in /etc/ssh/sshd_config, but they don't all share the same 'authorized_keys' files, so if need be, root specific ones.
Cannot simply add the public key generated for the root account in the /home/yournameuser/.ssh/authorized_keys file - it seems that the system doesn't look there for root access._

## Create new user

### Create new sudo user: user

$ adduser user
$ usermod -aG sudo user
$ usermod -aG adm user

Password: xxxxx

### Create the folder if it doesn't already exist:
$ mkdir /home/$USER/.ssh

### Copy the authorized_keys file that contains the public keys:
$ sudo cp /root/.ssh/authorized_keys /home/$USER/.ssh/authorized_keys

### Make the directory and file only executable by the root and setup ownership and permissions:

$ chmod 700 /home/$USER/.ssh
$ sudo chown -R $USER:$USER /home/$USER/.ssh
$ sudo chmod 600 /home/$USER/.ssh/authorized_keys

### Restart ssh

$ service ssh restart

**Log out and log back in**

Change some settings for SSH

Disable password authorization first then change SSH Port when UFW is set up

$ sudo vim /etc/ssh/sshd_config

# Backup original config file
$ sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

Change SSH port from 22 to 22022 < later
PermitRootLogin yes (later change) 
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
AuthorizedKeysCommand none
AuthorizedKeysCommandUser nobody
PasswordAuthentication no
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication yes
GSSAPICleanupCredentials no
UsePAM no
X11Forwarding yes
PrintMotd no

$ sudo service ssh restart

$ sudo systemctl restart sshd

Before logging off, if using windows 11/Powershell as a terminal to SSH in, do this:

Config Windows to use port 22022 to SSH in

Run Powershell as Administrator
$ cd C:/Users/$USER/.ssh

Create new config file
$ code config

Paste in file: 
# Per-Host Per User basis config
Host dosvr1-root (to remove when the dust settles)
    Hostname {VPS IP Address} 
    Port 22022
    User root
    Identityfile C:/Users/$USER/.ssh/root-key
    IdentitiesOnly yes

Host dosvr1-user
    Hostname {VPS IP Address} 
    Port 22022
    User user
    Identityfile C:/Users/$USER/.ssh/admin-key
    IdentitiesOnly yes

Other examples of configs:

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

Start a new terminal to SSH in

Fail2ban & UFW

Secure SSH via UFW (HestiaCP no need)

apt install ufw

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

Change PermitRootlogin config to no:
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

Cloud Panel


Install Cloud Panel:  https://www.cloudpanel.io/docs/v2/getting-started/other/

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






