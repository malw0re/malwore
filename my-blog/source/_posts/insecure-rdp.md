---
title: "INSECURE RDP"
date: 2023-10-13T17:21:14+03:00
draft: false
tags: [rdp]
categories:
	- Writeups
ShowReadingTime: true
description: "Exploiting insecure RDP Services"
ShowPostNavLinks: true
top_img: /images/cyberpunk-red.png
ShowToc: true
---

**Objective:** 

Exploit the application and retrieve the flag!

### Enumeration

Running nmap on our target, we see there's a distinct port `3333` with a very peculiar service `dec-notes`. Our focus primarily will be exploiting the insecure rdp service.

![](https://i.imgur.com/I1s9edD.png)


### Metasploit

To confirm if the target is running rdp we'll use a metasploit module called `rdp-scanner` what it does is it attempts to connect to the specified RDP port and determines if it *"speaks"* RDP. 

Setting up the required variables for the module, we get an interesting response it's running RDP!!

![](https://i.imgur.com/54Cb8jL.png)

### Bruteforcing credentials

But for us to get the flag we need credentials that we can login in using rdp to get the flag, for that we'll need to bruteforce some credentials.

We'll need to use hydra for this bruteforce.

```bash 
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt rdp://10.6.23.164:3333 
```

So basically the commands takes in a user and a password wordlist to bruteforce against the rdp service with the specified port number during enumeration.

As we can see immediately we get some credentials that we can try logging in with, the account administrator seems perfect cause it has more privileges as compared to the rest so we'll login with that one to get the flag.

![](https://i.imgur.com/qtz5e8G.png)

#### *N|B* 
during an engagement it's necessary to check or limit the bruteforce speed as not to cause a denial of service on the service you are pentesting.

### RDP login

Having enumerated some credentials, we can login via rdp using the xfreerdp tool since that's what is installed in this instance i have, but there are many tools out there that you can use to login via rdp.

Moving forward we'll login via xfreerdp.

![](https://i.imgur.com/Gp5D6kn.png)

![](https://i.imgur.com/LvnIpr5.png)

We see we have a GUI instance of RDP connection on the target machine, now we can fetch the flag from the C:/ drive.

![](https://i.imgur.com/VWadcyX.png)


### Mitigation Against Bruteforce
The Bruteforce attack that we just performed can be mitigated. It requires the creation of an Account Policy that will prevent Hydra or any other tool from trying multiple credentials. It is essentially a Lockout Policy. To toggle this policy, we need to open the Local Security Policy window. This can be done by typing **“secpol.msc”**. 

Checking out the Account policy we can see there no policies implemented which falls under misconfigurations which lead to such exploitations.

![](https://i.imgur.com/8XxUKeD.png)

To get to the particular policy we need to Account Policies under Security Settings. Inside the Account Policies, there exists an Account Lockout Policy. It contains 3 policies each working on an aspect of the Account Lockout. 

The first one controls the duration of the lockout. This is the time that is required to be passed to log in again after the lockout. Then we have the Lockout Threshold. This controls the number of invalid attempts. Please toggle these as per your requirements. This should prevent the Bruteforce attack.

![](https://i.imgur.com/vNRevnq.png)

After trying the Bruteforce attack using Hydra, it can be observed that it is not possible to extract the credentials as before. Although there is still some risk that can be prevented by forcing the users to change the passwords frequently and enforcing good password policies.

![](https://i.imgur.com/YwhqTHW.png)

As we enabled a lockout policy, we will not be able to log in on the machine even with the correct password until the time passed that we toggled in the policy. 







