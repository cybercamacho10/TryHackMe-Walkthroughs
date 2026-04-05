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

The key to figure out is by looking at the context and correlation of other logs.

In the figured labeled Successful SSH Logins. With the ansible login, we can see that she used a key, which as we have seen is safer than using a password method. You can also analyze that the login was made from an internal ip address.
However, with the other logs for jsmith, we can analyze that a password authentication and an external ip address was used which raises a red flag.

We will practice this answering the questions on this task.

Questions:
<br />
Question 1 - When did the SSH password brute force start?
<br />
Answer Format: 2023-09-15.

This question tells us that we are analyzing an SSH brute force attack.

To recap, a brute force attack is an authentication attack where an attacker attempts to gain access by systematically trying multiple password combinations. This attack typically happens in an automated manner.

Brute force attacks can be identified in logs by:
* Repeated failed login attempts within a short period of time
* Multiple authentication attempts originating from the same IP address
* Attempts targeting one or multiple user accounts
* High-frequency login attempts indicating automation

With this in mind, we now know what we are looking for.

To analyze the logs, remember we will need to look at the auth.log logs since we are looking for authorization and authentication activity.

Since a characteristic of a brute force attack is a lot of failed login attempts, let us grep for "failed"

<img width="1421" height="726" alt="Screenshot 2026-04-05 at 1 28 38 PM" src="https://github.com/user-attachments/assets/86d031c0-8b0a-48cc-ba62-261f4b831eb3" />

We can analyze that there are multiple failed attempts in a short amount of time from the 193.46.255.33 ip address.

We can see that there is a Failed Password attempt from 197.39.195.136, but the brute force attack did not begin here. This attempt is not part of the brute force attack since there is only one attempt in a short amount of time. 

Answer: 2025-08-21

Question 2 - Which four users did the botnet attempt to breach?
<br />
Answer Format: Separate by a comma, in alphabetical order.

From this question we can see that the system is being attacked by a botnet, meaning that the attacker is using multiple ip addresses to brute force.

Looking at the results again
<img width="1424" height="729" alt="Screenshot 2026-04-05 at 1 36 20 PM" src="https://github.com/user-attachments/assets/e383bcfd-105f-4e53-a5ca-6c0af1edfb76" />

We see that the ip address 193.46.255.33 was trying to access user root multiple times in a short amount of time.
We can add the user "root" to our list

We also notice that same ip address accessing user sol. Even though there was only one attempt, from the analysis we can conclude that the ip address is part of the botnet.
We can the user "sol" to the list.

We also see that the 80.94.95.112 is making multiple attempts to the same user (roy) in a short ammount of time.
We will add "roy" to the list.

Looking at the later events
<img width="1418" height="725" alt="Screenshot 2026-04-05 at 2 06 13 PM" src="https://github.com/user-attachments/assets/139c36c1-748f-48fc-9523-bcf39b71fe67" />

We can see 80.94.95.112 is making multiple attempts to the user user in a short ammount of time.
We will add "user" to the list

We have found our four users

Answer: root, roy, sol, user

Question 3 - Which four users did the botnet attempt to breach?
<br />





    
