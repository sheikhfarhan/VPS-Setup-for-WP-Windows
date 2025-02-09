## Part 1 - SSH Setup for Windows

### :arrow_right_hook: Install Powershell 7.5.x

+ Installing Powershell 7.5.x from Windows Store
+ Other methods of installation and guide here:
https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5
+ Start PowerShell

![image](https://drive.google.com/uc?export=view&id=130cV5uuvJmtIihQX7jkdeLXtvQ2g4uOT)

### :arrow_right_hook: Generate Keys via PowerShell

```
ssh-keygen -t ed25519 -f C:/Users/{user}/.ssh/sfarhan-key -C ""
```

![image](https://drive.google.com/uc?export=view&id=13531RhTEtJX2wiwf-V-lk9U659fFpYFZ)

### :arrow_right_hook: Check Windows User ~/.ssh folder

![image](https://drive.google.com/uc?export=view&id=135mjes1PrPOyMhpw_DBJEWFcAr7RG7Vk)

> [!NOTE]
> + Can add any nickname/comment after -C (eg: -C homepc)
> + -C "" will not have any comments in the keys
> + Can remove the private keys from local directory after adding key to [ssh-add] setup and store it at Bitwarden for safekeeping

**_All keys are autosave to: C:/Users/{user}/.ssh folder_**

### :arrow_right_hook: Enable OpenSSH in Windows

Go to System -> Optional Features -> Add OpenSSH

![image](https://drive.google.com/uc?export=view&id=1-XfrWohXqF5buyT7Y6m7HR5xIyvGhpre)

### :arrow_right_hook: Set up [sshd] & [ssh-agent] services

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

### :arrow_right_hook: Start the sshd and ssh-agent services:
```
Start-Service sshd
```
```
Start-Service ssh-agent
```
### :arrow_right_hook: Check ssh-agent is running:
```
Get-Service ssh-agent
```
![image](https://drive.google.com/uc?export=view&id=136DHsa25Js9P4rv4stwiRw_76d1khBZR)

> [!NOTE]
> + This setup and configurations for Windows are for [ssh-agent] to securely store the private keys within Windows security context, associated with Windows account
> + And to start the [ssh-agent] service each time the computer is rebooted (to start automatically)
> + By default the [ssh-agent] service is disabled
>

### :arrow_right_hook: Load private key onto [ssh-agent]

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

+ [Part 2 - VPS Setup & Login](https://github.com/sheikhfarhan/VPS-Setup-for-WP-Windows/tree/main/Part%202%20-%20VPS%20Setup%20%26%20Login)

