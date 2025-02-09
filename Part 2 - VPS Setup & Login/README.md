## Part 2 - VPS Deployment & LogIn

### :arrow_right_hook: Password-less SSH
+ For Digital Ocean, Hetzner, Vultr or any other VPS providers, if there is an option to “park” public keys at their console, use the feature
+ If not, deploy with root password
+ If add an SSH key, no root credentials will be sent via email

![image](https://drive.google.com/uc?export=view&id=13DVacST8KJNqqWrJv9beCkuZdujufuAu)

For Digital Ocean, the settings are under "My Account" -> "Manage Team Settings" -> "Security" -> "Add SSH Key":

### :arrow_right_hook: After adding public key:

![image](https://drive.google.com/uc?export=view&id=13DlXEGmsVuoHIssaARsa7rmqV28DH6z8)

When creating server, now will have the option to add the public key for a password-less entry:

![image](https://drive.google.com/uc?export=view&id=13FmLD46ZIauh1qt87cei8jwHP0o8Fm0i)

### :arrow_right_hook: Server created!

![image](https://drive.google.com/uc?export=view&id=1534QfOtszExXElbx-IMBFhkNulyzwAJ1)

### Going back to Windows to add the IP address to a Config file

### :arrow_right_hook: Create new Config file for User

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

### :arrow_right_hook: Add the following to the config file:
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

### :arrow_right_hook: SSH via root access:

From Powershell:
```
ssh root@dosvr2
```

![image](https://drive.google.com/uc?export=view&id=15GZ3Ryo9EztTYIfhO5RLxUkLlz2DX6wa)

### :white_check_mark: And we are in!
**Lets secure our VPS next.**

+ [Part 3 - Hardening: SSH, UFW & Fail2ban](https://github.com/sheikhfarhan/VPS-Setup-for-WP-Windows/tree/main/Part%203%20-%20Hardening%3A%20SSH%2C%20UFW%20%26%20Fail2ban)


