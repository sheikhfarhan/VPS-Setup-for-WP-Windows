# Intro

I have been deploying many VPSs for varied use cases and wanted to finally document the steps I take for my reference. The idea is to spin up a VPS, layer adequate security, and allow me easy access to the server from my Home PC (Windows 11) and other devices.

For this project, I want to set up a WordPress website with a free control panel. Been using CloudPanel (taking advantage of the 1-click installation of WordPress and Sites' Management features) and Cloudflare with no issues for a couple of years already. I am familiar with them and they provide what I need for my use case/requirements.

And if these steps can help others navigate their DIY adventures, that would be great too! :)

#### Current stacks/environments:

+ Workstation/Client: Windows 11 PC
+ Terminal: Powershell 7.x
+ 1 x Admin/User (me)
+ 1 x main SSH key (mine)
+ VPS - 2vCore and 2GB Ram
+ Ubuntu 24.04 OS 
+ DNS management via Cloudflare (+ Origin Certificates issuance)
+ Cloudpanel as websites control panel
+ CloudPanel Login page under reverse proxy (to achieve clp.domain.com as my panel's login)
+ Cloudpanel default stack is LEMP (Nginx)
+ Installation of Wordpress (hardened)

## Win 11 + VPS + Cloudpanel + Cloudflare + Wordpress Setup

This page is where the entire steps are, but for specific sections/parts of the deployment, the shortcuts are:

+ [Part 1 - SSH Setup for Windows](https://github.com/sheikhfarhan/VPS-Setup-for-WP-Windows/tree/0ba0e4f94dd82047c42cae3570c38d2442edb446/Part%201%20-%20SSH%20Setup%20for%20Windows)
+ Part 2 - VPS Setup & Login
+ Part 3 - Hardening: SSH, UFW & Fail2ban
+ Part 4 - Cloudflare, SSLs & DNS Setup
+ Part 5 - CloudPanel Setup
+ Part 6 - Install & Securing Wordpress

### Selecting a VPS (Virtual Private Server) Provider

Shortlisted providers are all with SG-located datacenter. Going for a minimum of 2GB Ram.

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

Allright, Lets go!

<p>&nbsp;</p>

## Part 1 - SSH Setup for Windows

#### :arrow_right_hook: Install Powershell 7.5.x

+ Installing Powershell 7.5.x from Windows Store
+ Other methods of installation and guide here:
https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5
+ Start PowerShell

![image](https://drive.google.com/uc?export=view&id=130cV5uuvJmtIihQX7jkdeLXtvQ2g4uOT)

#### :arrow_right_hook: Generate Keys via PowerShell

```
ssh-keygen -t ed25519 -f C:/Users/{user}/.ssh/sfarhan-key -C ""
```

![image](https://drive.google.com/uc?export=view&id=13531RhTEtJX2wiwf-V-lk9U659fFpYFZ)

#### :arrow_right_hook: Check Windows User ~/.ssh folder

![image](https://drive.google.com/uc?export=view&id=135mjes1PrPOyMhpw_DBJEWFcAr7RG7Vk)

> [!NOTE]
> + Can add any nickname/comment after -C (eg: -C homepc)
> + -C "" will not have any comments in the keys
> + Can remove the private keys from local directory after adding key to [ssh-add] setup and store it at Bitwarden for safekeeping

**_All keys are autosave to: C:/Users/{user}/.ssh folder_**

#### :arrow_right_hook: Enable OpenSSH in Windows

Go to System -> Optional Features -> Add OpenSSH

![image](https://drive.google.com/uc?export=view&id=1-XfrWohXqF5buyT7Y6m7HR5xIyvGhpre)

#### :arrow_right_hook: Set up [sshd] & [ssh-agent] services

Open PowerShell as _Administrator_

For [sshd]:
```
Get-Service -Name sshd | Set-Service -StartupType Automatic
```
For [ssh-agent]:
```
Get-Service ssh-agent | Set-Service -StartupType Automatic
```
These will set the sshd and ssh-agent services to start automatically.

#### :arrow_right_hook: Start the sshd and ssh-agent services:
```
Start-Service sshd
```
```
Start-Service ssh-agent
```
#### :arrow_right_hook: Check ssh-agent is running:
```
Get-Service ssh-agent
```
![image](https://drive.google.com/uc?export=view&id=136DHsa25Js9P4rv4stwiRw_76d1khBZR)

> [!NOTE]
> + This setup and configurations for Windows are for [ssh-agent] to securely store the private keys within Windows security context, associated with Windows account
> + And to start the [ssh-agent] service each time the computer is rebooted (to start automatically)
> + By default the [ssh-agent] service is disabled
>
#### :arrow_right_hook: Load private key onto [ssh-agent]

```
ssh-add $env:USERPROFILE\.ssh\sfarhan-key
```
![image](https://drive.google.com/uc?export=view&id=138QwWijpZyN3PJkwXnm6ERj2uRiX5L6i)

> [!NOTE]
> + The ssh-agent in Windows will now automatically retrieve the local private key and pass it to SSH client.
> + If need be, can create new set of keys for each devices (Tablet, Phone) to SSH in.
> + Then standby to add those public keys onto the server when ready.

### :desktop_computer:	Windows is ready to remotely SSH to our VPS when ready!
**Lets now go over to our VPS Provider side of things..**

<p>&nbsp;</p>

## Part 2 - VPS Deployment & LogIn

#### :arrow_right_hook: Password-less SSH
+ For Digital Ocean, Hetzner, Vultr or any other VPS providers, if there is an option to “park” public keys at their console, use the feature
+ If not, deploy with root password
+ If add an SSH key, no root credentials will be sent via email

![image](https://drive.google.com/uc?export=view&id=13DVacST8KJNqqWrJv9beCkuZdujufuAu)

For Digital Ocean, the settings are under "My Account" -> "Manage Team Settings" -> "Security" -> "Add SSH Key":

#### :arrow_right_hook: After adding public key:

![image](https://drive.google.com/uc?export=view&id=13DlXEGmsVuoHIssaARsa7rmqV28DH6z8)

When creating server, now will have the option to add the public key for a password-less entry:

![image](https://drive.google.com/uc?export=view&id=13FmLD46ZIauh1qt87cei8jwHP0o8Fm0i)

#### :arrow_right_hook: Server created!

![image](https://drive.google.com/uc?export=view&id=1534QfOtszExXElbx-IMBFhkNulyzwAJ1)

#### Going back to Windows to add the IP address to a Config file:

#### :arrow_right_hook: Create new Config file for User

Run Powershell as **Administrator** and navigate to .ssh folder and create the file:
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

#### :arrow_right_hook: Add the following to the config file:
```
# this is main user access
Host dosvr2
    Hostname {IP address}
    Identityfile C:/Users/{user}/.ssh/sfarhan-key
```

![image](https://drive.google.com/uc?export=view&id=1-alnvO48gcScLHyvDkcfMO7KfKTyemzt)

> [!Note]
> + Host (which is an alias, sort of shortcut name for Windows/ssh-agent) can be anything
> + Hostname is usually IP address or domain
> + IdentityFile is the path to the private keys, as we did rename the key to some other name than the default
> + Can use Notepad to open and edit the config file from Windows Explorer
> + To use the same config file and configure change of SSH port later

Save file.

#### :arrow_right_hook: SSH via root access:

From Powershell:
```
ssh root@dosvr2
```

![image](https://drive.google.com/uc?export=view&id=15GZ3Ryo9EztTYIfhO5RLxUkLlz2DX6wa)

### :white_check_mark: And we are in!
**Lets secure our VPS next.**

<p>&nbsp;</p>

## Part 3 - Hardening of VPS

## Housekeeping
```
apt update && apt upgrade -y
```
```
apt autoremove && apt autoclean -y
```

#### :arrow_right_hook: Reboot server (and log back in to continue)
```
shutdown -r now
```

#### :arrow_right_hook: Change System Time / Time Zone:
```
dpkg-reconfigure tzdata
```
Follow on screen instructions to set timezone. 

#### :arrow_right_hook: Restart cron to ensure system picks up the change
```
service cron restart
```

#### :arrow_right_hook: Disable unattended-upgrades:
```
dpkg-reconfigure unattended-upgrades
```
or can remove it completely:
```
apt remove unattended-upgrades
```
#### :arrow_right_hook: Check SSH authorized_keys file

Ensure that the 2 public keys are in the /root/.ssh folder

![image](https://drive.google.com/uc?export=view&id=15H0ge-Mx3cqClT7ZhAX6QAVMIy34sVg3)

If not..

#### :arrow_right_hook: Create folders for .ssh and a file for authorized_keys:
```
cd /root
mkdir .ssh
cd .ssh
nano authorized_keys
```

Then put the relevant public keys in here.\
If need to add additional public keys (eg: for Laptop/Tablet/Phone devices entry points), can add them in now.

#### :arrow_right_hook: Setup ownership and permissions:
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R root:root ~/.ssh
```
#### :arrow_right_hook: Restart SSH service
```
systemctl restart ssh
```

#### :arrow_right_hook: Restart the sshd daemon, still as root, with:
```
systemctl restart ssh.service
```

> [!Note]
> All users (root and other users) all share the same config in /etc/ssh/sshd_config, but they don't all share the same 'authorized_keys' files. Thus, even when using same set of keys, need a separate authorized_keys file in /root/.ssh/ and for in /home/yournameuser/.ssh/ but in this case contains same set of public keys_

#### :arrow_right_hook: Create New Sudo user
```
adduser <user>
usermod -aG sudo <user>
```

#### :arrow_right_hook: Create the .ssh folder for new sudo user:
```
mkdir /home/{user}/.ssh
```
#### :arrow_right_hook: Copy the authorized_keys file that contains the public keys:
```
cp /root/.ssh/authorized_keys /home/{user}/.ssh/authorized_keys
```

#### :arrow_right_hook: Setup ownership and permissions:
```
chown -R {user}:{user} /home/{user}/.ssh
chmod 700 /home/{user}/.ssh
chmod 600 /home/{user}/.ssh/authorized_keys
```
**_Now the new sudo user has the public keys in their own authorized_keys file_**

#### :arrow_right_hook: Restart ssh
```
systemctl restart ssh
systemctl restart ssh.service
```

#### :arrow_right_hook: SSH in via sudo user

Open **NEW** Powershell terminal:
```
ssh sfarhan@dosvr2
```
_Note:
The "dosvr2" is what we defined in the Windows side of things /.ssh config file above_

![image](https://drive.google.com/uc?export=view&id=15MF9ssfpgB-gYTX1bmrJ2_w4Ce8PWsGJ)

Test a few sudo commands to ensure all is okay, then next step is to disable root login, securing SSH access and implement UFW and Fail2ban

## Securing SSH Accecss

+ Change SSH port from 22 to another (here I am using 22022)
+ Disable RootLogin
+ Disable entry points via Passwords (only SSH keys)

#### :arrow_right_hook: Backup original config file
```
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

#### :arrow_right_hook: Make changes to file
```
sudo vim /etc/ssh/sshd_config
```
```
Port 22022
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
```
Save And Close file

#### :arrow_right_hook: Restart Services for changes to take effect
```
sudo systemctl restart ssh
```
To activate this new config, it is now required **to inform systemd about the change**:
```
sudo systemctl daemon-reload
```

#### :arrow_right_hook: ssh service and socket can be restarted, to **activate the change**:
```
sudo systemctl restart ssh.socket
sudo systemctl restart ssh.service
```

#### :arrow_right_hook: Verify new SSH port number is listening:
```
sudo lsof -i tcp:22022
```

![image](https://drive.google.com/uc?export=view&id=1-cchi4xBJxn73l_h2ffXAnjaimz8tDng)

> [!IMPORTANT]
> Before logging off, need to allow the new port in UFW (Firewall) settings and add new port info in Windows SSH config file\
> Do not close the existing session! and continue next steps

## UFW

_UFW is already pre-installed with Ubuntu 24.04 but disabled._

#### :arrow_right_hook: Enable UFW

```
sudo ufw enable
```
_can proceed if system asks to_

#### :arrow_right_hook: Check UFW is running
```
sudo ufw status
```

#### :arrow_right_hook: Establish Default Rule
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
```

#### :arrow_right_hook: Allow new SSH port
```
sudo ufw allow 22022/tcp
```

#### :arrow_right_hook: Add other basic rules for UFW
```
sudo ufw allow http/tcp 
sudo ufw allow https/tcp
sudo ufw logging off # or sudo vim /etc/ufw/ufw.conf (LOGLEVEL=off).
```

#### :arrow_right_hook: Check UFW rules
```
sudo UFW status
```

![image](https://drive.google.com/uc?export=view&id=15Pr2cAXvCvapTLV1lVLg7_E9GrVEX9L4)

> [!NOTE]
> Will need to add the port info during Cloudpanel's installation process, as its default port is 22

#### :arrow_right_hook: Add new SSH port in Window's SSH config file:

Open the config file via Notepad and add the port info. File is at C:/Users/{user}/.ssh/config
```
# this is main user access
Host dosvr2
    Hostname {IP address}
    Port 22022 # <-this
    Identityfile C:/Users/{user}/.ssh/sfarhan-key
```
Save file

![image](https://drive.google.com/uc?export=view&id=15QSeFs6a3gpFChrYGq39zhXNF8uKZgfV)

> [!IMPORTANT]
> Before logging off, Start a new terminal to SSH in to test

#### :arrow_right_hook: Open new terminal (without closing the current session)

Log-in server as usual without the need to specify port info:

![image](https://drive.google.com/uc?export=view&id=15RGq_uH_yiBnISAQmnYGUTnaFtz-9Vjj)

## Fail2ban

#### :arrow_right_hook: Install Fail2ban
```
sudo apt-get update
sudo apt-get install fail2ban -y
```

#### :arrow_right_hook: Check Fail2ban status
```
sudo /etc/init.d/fail2ban status
```

![image](https://drive.google.com/uc?export=view&id=15RRASwdVI7NcwkFTTUT34pRiXfYjDKP5)

#### :arrow_right_hook: Create new jail.local file
```
sudo touch /etc/fail2ban/jail.local
```

#### :arrow_right_hook: Edit the file:
```
sudo nano /etc/fail2ban/jail.local
```
Add:
```
[DEFAULT]
ignoreip = 127.0.0.1/8
findtime = 3200
bantime = 86400

[sshd]
backend=systemd
enabled = true
port = 22022
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
```

![image](https://drive.google.com/uc?export=view&id=14pDUHgIp9L7Fkuzqk4ZsNyhlU7jwHrh_)

> [!NOTE]
> Every .conf file can be overridden with a file named .local. The .conf file is read first, then .local, with later settings overriding earlier ones. Thus, a .local file doesn't have to include everything in the corresponding .conf file, only those settings that you wish to override. Modifications should take place in the .local and not in the .conf. This avoids merging problems when upgrading.

#### :arrow_right_hook: Restart Fail2ban
```
sudo systemctl restart fail2ban
sudo service fail2ban restart
```

> [!NOTE]
> Install this if fail2ban jock error shows up:
> ```
> sudo apt install python3-systemd
> ```

#### :arrow_right_hook: Check the new rules by:
```
sudo iptables -S
sudo iptables -L
```

#### :arrow_right_hook: To Check Bans/Jails
```
sudo fail2ban-client status sshd
```

![image](https://drive.google.com/uc?export=view&id=1-kkAvncAMay8uwEpc0DNV4gYm-uBeyNr)

#### :arrow_right_hook: Check if loggings are working fine:
```
tail -f /var/log/auth.log
tail -f /var/log/fail2ban.log
```

### :+1: VPS is hardened and is ready for:
+ Easy and secured entry from a Windows setup
+ and Installation of applications at will

**Next Step is to go over to our Cloudflare account for the DNS and SSLs**

<p>&nbsp;</p>

## Part 4 - Cloudflare Setup

#### :arrow_right_hook: Point Nameservers to Cloudflare

Go to Domain Registrar and add Cloudflare's given nameservers

![image](https://drive.google.com/uc?export=view&id=14pLjdHOKozSED2eVMWjopYNb69k1Bm9b)

#### :arrow_right_hook: Adding A & CNAME records for domain

![image](https://drive.google.com/uc?export=view&id=153-rR06znAnvLD67EgRvgoNM1MkaSM2w)

> [!NOTE]
> CloudPanel's log-in page will be clp.domain.com\
> Do not proxy it through Cloudflare yet!

#### :arrow_right_hook: Switch to Full (Strict) mode

SSL/TLS -> Overview -> Configure

![image](https://drive.google.com/uc?export=view&id=1-jeIhfHYPGM-BvZVWfdL2Ba89uNhXZlh)

#### :arrow_right_hook: Create CF Certificate

SSL/TLS -> Origin Server -> Create

![image](https://drive.google.com/uc?export=view&id=151xiP1wt7WIU5KUP-9MVJPDYzMtG4556)

Copy the two sets of keys somewhere. We will need them later. Once copied, click OK.

<p>&nbsp;</p>

## Part 5 - Installing a Control Panel

We have a few choices - CloudPanel, HestiaCP, Webadmin etc..\
Here I am documenting the steps I take for using CloudPanel for the WordPress installation

 #### :arrow_right_hook: Install CloudPanel
 ```
curl -sS https://installer.cloudpanel.io/ce/v2/install.sh -o install.sh && sudo bash install.sh --ssh_port 22022 
```

https://www.cloudpanel.io/docs/v2/getting-started/other/

  _Since I have a different SSH port than the usual 22, will need to add that SSH part for the installation_

#### :arrow_right_hook: Housekeeping
```
apt-get autoremove && apt-get autoclean
```

#### :arrow_right_hook: Log in to CloudPanel via:
https://yourIpAddress:8443

#### :arrow_right_hook: Firewall in CloudPanel

Admin Area -> Security\
Add: Custom SSH Port - 22022 - ip4: 0.0.0.0/0\
Add: Custom SSH Port - 22022 - ip6: ::/0

![image](https://drive.google.com/uc?export=view&id=15j10ttzLrtfN5MSTBo9v07Ef8ifXgVAJ)

#### :arrow_right_hook: Delete SSH Port 22 and summary as follows:

![image](https://drive.google.com/uc?export=view&id=15lBNoFNR8ThESjKIQs5z2-5eaWh4kPNZ)

#### :arrow_right_hook: Setting up a subdomain for CloudPanel's log-in page

+ Go to Sites and Create a Reverse Proxy 
+ Enter the Domain Name as clp.domain.com, enter https://127.0.0.1:8443 as Reverse Proxy Url.
+ Make sure the https is the url
+ User: reverseproxy (this is just to contain the /home docs)

![image](https://drive.google.com/uc?export=view&id=15fE6i9sNaV5DYcIusBA7NWfDJls2aVVv)

+ Once created, go to Manage -> SSL -> and import Cloudflare Origin Certificate for clp.domain.com (to log into the panel)
+ The Cloudflare Origin Certificate was from the previous step

![image](https://drive.google.com/uc?export=view&id=15iLO7ePHRr7W9FWKEnOdGTCps83yeTdx)

+ Go back to Cloudflare DNS page and enable Proxy for the clp.domain.com A record
+ https://clp.domain.com will now bring to CloudPanel login page with SSL

![image](https://drive.google.com/uc?export=view&id=15wstwfoOh_CWaUqsgSXjxH2U_T1h_n7B)

## **:+1: Server is hardened and our Control Panel is ready**

**Last few steps are to install WordPress and layer some basic WP securities**

<p>&nbsp;</p>

## Part 6 - INSTALL WORDPRESS

#### :arrow_right_hook: 1-click Wordpress Installation

![image](https://drive.google.com/uc?export=view&id=1aei7zObzebNXOVBzG5fIi8SteQtsftEE)

#### :arrow_right_hook: Copy and Save the generated Credentials

![image](https://drive.google.com/uc?export=view&id=1-7kMQ8vO9o8AVcNC0iWVNScxjV5eGmka)

#### :arrow_right_hook: Add Cloudflare's Certificate

![image](https://drive.google.com/uc?export=view&id=1-8BTHuaQZZ-qpnnyqxjFMonCTHuNFFcN)
![image](https://drive.google.com/uc?export=view&id=1-AG4hzUfCRjrsGx_EOKC1u_cNmSmEEvh)

#### :arrow_right_hook: Add a subdomain for the site in Cloudflare

![image](https://drive.google.com/uc?export=view&id=1-CjJqVPhAWIXNB-hmEWdmpiCjraNyquR)

#### Additional Settings:
+ Site -> Manage -> Security -> Allow traffic from Cloudflare only
+ Site -> Manage -> Settings -> Adjust PHP Settings and Add SSH Keys for SSH/SFTP (though CloudPanel already has a decent file manager in-built)
+ Can start tweaking any web-related settings where makes sense!

### Wordpress/Website is Live!

![image](https://drive.google.com/uc?export=view&id=1-IE_KZnigr1iEN3vLUz7UGyonJON-r0f)

#### :arrow_right_hook: Login to WP-Admin @ domain.com/wp-admin

Credentials are as saved earlier in Cloudpanel:

![image](https://drive.google.com/uc?export=view&id=1-zoDDEHORKhmxgxglL0scEl2dxc8Fh6G)

#### :arrow_right_hook: Install Security Plugins

**Wordfence**

Scan and if they found a file that needs to hide but is unable to hide at UI level eg:

![image](https://drive.google.com/uc?export=view&id=1-Rn_VrF9rJIT_4jGwXZDtu-B-t3Y_6h3)

Go to CloudPanel -> Site -> Manage -> Vhost and add:

location ~ \.user\.ini$ {
deny all;
}

Try another scan and should be cleared of the notification:

![image](https://drive.google.com/uc?export=view&id=1-W7Xn8GxUf-QRim7TJLefBDGplbbS2Eg)

**Limit Login Attempts Reloaded**

Go to Settings and configure accordingly:

![image](https://drive.google.com/uc?export=view&id=10H2jt7wFRRydK5cypKZUjqA6pH7KBsHs)

**WP fail2ban – Advanced Security**

_WPf2b comes with three fail2ban filters that we need!: wordpress-hard.conf, wordpress-soft.conf, and wordpress-extra.conf._

Explore the plugin conf files at htdocs. find the appropriate conf file to copy over to fail2ban filter.d directory

clp /home/$User/htdocs/domain.com/wp-content/plugins/wp-fail2ban/filters.d/wordpress-hard.conf /etc/fail2ban/filter.d/ 

and also the -soft and the -extra conf files

add the filter to jail.local to activate the ban

vim /etc/fail2ban/jail.local

At jail.local conf file, under wordpress filters/jail header, Use /home/sf-wp-admin /logs and /home/adco-wp-admin

exit

Limit Login Attempts Reloaded

#### Add these below after install WP Fail2ban plugin
```
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
```

> [!NOTE]
> Potentially can add new jails configs for Wordpress / MSQL / PHP / Nginx Bots etc…
> https://webdock.io/en/docs/how-guides/security-guides/how-configure-fail2ban-common-services






