Welcome to this walkthrough of the Linux Threat Detection 1 Room from the Linux Security Module part of the SOC Level 1 Path

This module is similar to the Windows Security Module but for Linux, where we will go through the MITRE attack framework and see how we can find various techniques in a Linux enviroment. 

This room will focus on how to detect Initial Access Techniques from ssh attacks and services breaches.

<img width="1410" height="144" alt="Screenshot 2026-04-04 at 6 16 50 PM" src="https://github.com/user-attachments/assets/ab1882d0-71d3-4f59-935f-32c0e3d806da" />

I hope this walkthorugh will help you clear any confusion regarding this room and you will learn a thing or two about cybersecurity.

Let's Begin!

Task 1 - Introduction
Linux is still used in our networks, therefore they are a major means which they get attacked. 
We must be ready to detect any inital access techniques in our linux system 

It is important you have knowledge of the MITRE Att&ck framework, Linux commands, and how to work with Linux logs.
It is reccomended to have completed these rooms before continuing with this room

Questions:
<br />
Question 1 - Read the Question Above
<br />
A: No answer needed

Task 2 - Initial Access via SSH
In this task, we learn that SSH is a very widely used service that allows users to connect remotely. 
However it can be used by attackers to gain access to our systems.

There are two common ways in which you can connect remotely via SSH: Password Authentication and SSH Key-Based Authentication
* Password Authentication is what we are mostly used to: username and password. We send to the ip address we are logging to the user we want to log into, and the password. If there is an ssh server configured, and if our credentials are correct, then we will be able to remotely log into our system.
  Password Autentication Syntax: ssh [user]@[ip address], you will then be prompted for the password


* SSH Key-Based Authentication is also known as Passwordless authentication because it relies on keys. There are two keys: Public key and Private key. The public key is for the server and the private key is for the client. In other words, the Public key is kept on the system you want to log into remotely, and the private key is kept on the system you are using to connect remotely. This is a passwordless method because it doesnt really on a password to prove an identity, rather, the cleint proves it owns the Private Key in order to gain connection. Key-Based Authentication is nearly impossible to brute force, making it safer than Password Authentication. However if someone obtains a copy of your Private Key, then your system is compromised. That is why it is best practice to add a passphrase to the Private Key to add an additional layer of defence.
  Passwordless Authentication Syntax: ssh [user]@[ip address], unlike above, you will not be prompted by the password

Hopefully this was usefull reminder of how ssh works!

With this information, we can see that we need to look out for:
  Weak Passwords in Password Authentication and access to our Private Key File

Questions:
<br />
Question 1 - When did the ubuntu user log in via SSH for the first time?
<br />
Answer Example: 2023-09-16.

We need to remember that from the previous room that the way we look for authentication events is through the auth.log logs

<img width="1382" height="401" alt="Screenshot 2026-04-04 at 7 07 37 PM" src="https://github.com/user-attachments/assets/2acde9c9-d0a1-465a-b856-58c5d4bf22b0" />

However this is to much information and going through it line by line will take forever, hence we will use the grep tool.
In order for the grep tool to be useful we need to know keywords to help us find what we are looking for.

Since we are looking for a succesful ssh log in, we grep for the keywords "ssh" and "Accept" since you find these words on log entries that log succesful ssh logins.
<img width="1419" height="290" alt="Screenshot 2026-04-04 at 7 12 46 PM" src="https://github.com/user-attachments/assets/e2a2fea0-fa3a-461a-bdb4-3aefba2a6ea5" />

You will see here that the first entry doesnt have the year, however by doing further analysis you can infer that the year for these logs is 2024. 
<img width="1423" height="315" alt="Screenshot 2026-04-04 at 7 17 48 PM" src="https://github.com/user-attachments/assets/91e2a16e-ad1c-49d2-a9f5-00ece379dd90" />

Something that was confusing for me is that how did the format change. If you grep for SSH logs, you can see that the SSH Servce (SSHD) restarted. When SSHD restarted, it could be that the logging system changed. 
The "Oct 22 08:55:25" logs follow the standard syslog timestamp format typical of the rsyslog system, while the later entries with the year and more accurate time use ISO 8601 format, which is typical of journald. Basically the way logs were processed changed.
Hope you found this helpful
<img width="1422" height="311" alt="Screenshot 2026-04-04 at 7 21 57 PM" src="https://github.com/user-attachments/assets/93adaa7c-44c4-4c7c-9840-f90c36909d98" />
Answer: 2024-10-22

Question 2 - Did the ubuntu user use SSH keys instead of a password for the above found date? (Yea/Nay)
This question is asking whether the user used Password or Keys to authenticate.
From the screenshot, you can see in the first entry that the authentication method used was 'publickey' meaning SSH Keys.
You will see at the bottom of the screenshot a succesful authentication via password.
<img width="1422" height="311" alt="Screenshot 2026-04-04 at 7 21 57 PM" src="https://github.com/user-attachments/assets/93adaa7c-44c4-4c7c-9840-f90c36909d98" />
Answer: YEA

Task 3 - Detecting SSH Attacks
We have seen how a sucessful SSH login looks like. Now, we will learn how to detect whether a login is malicious or not.
The key to figure out is by looking at the context.
With Alice, she used a key which as we have seen is safer than using a password method.

    
