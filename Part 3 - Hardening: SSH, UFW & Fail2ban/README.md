## Part 3 - Hardening of VPS

#### :arrow_right_hook: Housekeeping
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

+ [Part 4 - Cloudflare, SSLs & DNS Setup](https://github.com/sheikhfarhan/VPS-Setup-for-WP-Windows/tree/4dbd8176e7e87ddba638db05f65eb5e608eebd9c/Part%204%20-%20Cloudflare%2C%20SSLs%20%26%20DNS%20Setup)
