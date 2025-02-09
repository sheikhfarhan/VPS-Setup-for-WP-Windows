## Part 6 - INSTALL WORDPRESS

### :arrow_right_hook: 1-click Wordpress Installation

![image](https://drive.google.com/uc?export=view&id=1aei7zObzebNXOVBzG5fIi8SteQtsftEE)

### :arrow_right_hook: Copy and Save the generated Credentials

![image](https://drive.google.com/uc?export=view&id=1-7kMQ8vO9o8AVcNC0iWVNScxjV5eGmka)

### :arrow_right_hook: Add Cloudflare's Certificate

![image](https://drive.google.com/uc?export=view&id=1-8BTHuaQZZ-qpnnyqxjFMonCTHuNFFcN)
![image](https://drive.google.com/uc?export=view&id=1-AG4hzUfCRjrsGx_EOKC1u_cNmSmEEvh)

### :arrow_right_hook: Add a subdomain for the site in Cloudflare

![image](https://drive.google.com/uc?export=view&id=1-CjJqVPhAWIXNB-hmEWdmpiCjraNyquR)

### Additional Settings:
+ Site -> Manage -> Security -> Allow traffic from Cloudflare only
+ Site -> Manage -> Settings -> Adjust PHP Settings and Add SSH Keys for SSH/SFTP (though CloudPanel already has a decent file manager in-built)
+ Can start tweaking any web-related settings where makes sense!

### Wordpress/Website is Live!

![image](https://drive.google.com/uc?export=view&id=1-IE_KZnigr1iEN3vLUz7UGyonJON-r0f)

### :arrow_right_hook: Login to WP-Admin @ domain.com/wp-admin

Credentials are as saved earlier in Cloudpanel:

![image](https://drive.google.com/uc?export=view&id=1-zoDDEHORKhmxgxglL0scEl2dxc8Fh6G)

### :arrow_right_hook: Install Security Plugins

## 1. Wordfence

### :arrow_right_hook: Use the scan feature

If they found a file that needs to hide but is unable to hide at UI level eg:

![image](https://drive.google.com/uc?export=view&id=1-Rn_VrF9rJIT_4jGwXZDtu-B-t3Y_6h3)

### :arrow_right_hook: Add to Vhost:

_CloudPanel -> Site -> Manage -> Vhost_

```
location ~ \.user\.ini$ {
     deny all;
}
```

### :arrow_right_hook: Try another scan:

![image](https://drive.google.com/uc?export=view&id=1-W7Xn8GxUf-QRim7TJLefBDGplbbS2Eg)

## 2. Limit Login Attempts Reloaded

### :arrow_right_hook: Install and set accordingly:

![image](https://drive.google.com/uc?export=view&id=10H2jt7wFRRydK5cypKZUjqA6pH7KBsHs)

## 3. WP fail2ban – Advanced Security

WPf2b comes with three fail2ban filters that we need:\
  wordpress-hard.conf\
  wordpress-soft.conf\
  wordpress-extra.conf

### :arrow_right_hook: Add Plugin's conf files over

+ Explore the plugin's conf files at htdocs
+ Find the appropriate conf file to copy over to server's fail2ban filter.d directory

Via CLI:
```
sudo cp /home/$User/htdocs/$domain.com/wp-content/plugins/wp-fail2ban/filters.d/wordpress-hard.conf /etc/fail2ban/filter.d/
```
_and also the -soft and the -extra conf files_

![image](https://drive.google.com/uc?export=view&id=10QUq8EVKGYwWfF-Qc6_hm3ItfVvpuq4r)

_$User = Admin Username for the website over at CloudPanel_

Or, via File Manager at CloudPanel:

open -> copy -> create respective files in /etc/fail2ban/filter.d/ -> paste:

![image](https://drive.google.com/uc?export=view&id=10Neu-zhD1tw5UEF8UHcPE4UipMwM8juj)

### :arrow_right_hook: Add the filters to jail.local:

```
vim /etc/fail2ban/jail.local
```

### :arrow_right_hook: Add the following:
```
[wordpress-hard]
enabled = true
filter = wordpress-hard
logpath = /var/log/auth.log
backend = auto
maxretry = 3
port = http,https

[wordpress-soft]
enabled = true
filter = wordpress-soft
logpath = /var/log/auth.log
backend = auto
maxretry = 3
port = http,https

[wordpress-extra]
enabled = true
filter = wordpress-extra
logpath = /var/log/auth.log
backend = auto
maxretry = 3
port = http,https
```

Save file to activate the ban.

![image](https://drive.google.com/uc?export=view&id=10Wa0PY6PqOdgmBsL30oZX7OL1K1_u0ie)

### :arrow_right_hook: Restart Fail2ban
```
sudo systemctl restart fail2ban
```

### :arrow_right_hook: Check logpath

```
sudo tail -f /var/log/fail2ban.log
```

![image](https://drive.google.com/uc?export=view&id=10bkMEM4c3iXJZLY3dhKx2RYuTYdhVBm9)

#### Looks good! 

> [!NOTE]
> Potentially can add new jails configs for Wordpress / MSQL / PHP / Nginx Bots etc…
> https://webdock.io/en/docs/how-guides/security-guides/how-configure-fail2ban-common-services

##############

#### :+1: All right, WordPress website is up and running, WP-Admin is secured (relatively), with processes in place for ease of monitoring and tweaking.

