## Part 5 - Installing a Control Panel

We have a few choices - CloudPanel, HestiaCP, Webadmin etc..\
Here I am documenting the steps I take for using CloudPanel for the WordPress installation

 ### :arrow_right_hook: Install CloudPanel
 ```
curl -sS https://installer.cloudpanel.io/ce/v2/install.sh -o install.sh && sudo bash install.sh --ssh_port 22022 
```

https://www.cloudpanel.io/docs/v2/getting-started/other/

  _Since I have a different SSH port than the usual 22, will need to add that SSH part for the installation_

### :arrow_right_hook: Housekeeping
```
apt-get autoremove && apt-get autoclean
```

### :arrow_right_hook: Log in to CloudPanel via:
https://yourIpAddress:8443

### :arrow_right_hook: Firewall in CloudPanel

Admin Area -> Security\
Add: Custom SSH Port - 22022 - ip4: 0.0.0.0/0\
Add: Custom SSH Port - 22022 - ip6: ::/0

![image](https://drive.google.com/uc?export=view&id=15j10ttzLrtfN5MSTBo9v07Ef8ifXgVAJ)

### :arrow_right_hook: Delete SSH Port 22 and summary as follows:

![image](https://drive.google.com/uc?export=view&id=15lBNoFNR8ThESjKIQs5z2-5eaWh4kPNZ)

### :arrow_right_hook: Setting up a subdomain for CloudPanel's log-in page

+ Go to Sites and Create a Reverse Proxy 
+ Enter the Domain Name as clp.domain.com, enter https://127.0.0.1:8443 as Reverse Proxy Url.
+ Make sure the https is the url
+ User: reverseproxy (this is just to contain the /home docs)

![image](https://drive.google.com/uc?export=view&id=15fE6i9sNaV5DYcIusBA7NWfDJls2aVVv)

+ Once created, go to Manage -> SSL -> and import Cloudflare Origin Certificate for clp.domain.com (to log into the panel)
+ The Cloudflare Origin Certificate was from the previous step

![image](https://drive.google.com/uc?export=view&id=15iLO7ePHRr7W9FWKEnOdGTCps83yeTdx)

### :arrow_right_hook: Enable Proxy for the clp.domain.com A record

_https://clp.domain.com will now bring to CloudPanel login page with SSL_

![image](https://drive.google.com/uc?export=view&id=15wstwfoOh_CWaUqsgSXjxH2U_T1h_n7B)

### **:+1: Server is hardened and our Control Panel is ready**

**Last few steps are to install WordPress and layer some basic WP securities**

+ [Part 6 - Install & Securing Wordpress](https://github.com/sheikhfarhan/VPS-Setup-for-WP-Windows/blob/main/Part%206%20-%20Install%20%26%20Securing%20Wordpress/README.md)
