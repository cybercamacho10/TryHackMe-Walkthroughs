<!-- 
  Name of Room 
  
  <h1> [Name of Room] </h1>
-->
<h1> Linux Threat Detection 2 </h1>

<!-- 
  Welcome 
  
  Welcome to this walkthrough of the [Name of Room] Room from the [Name of Module] Module part of the SOC Level 1 Path!
-->
Welcome to this walkthrough of the Linux Threat Detection 2 Room from the Linux Security Module part of the SOC Level 1 Path!

<!-- What is the Module About -->
This module is similar to the Windows Security Module but for Linux, where we will go through the MITRE attack framework and see how we can find various techniques in a Linux enviroment. 

<!-- What is this Room About -->
This room will focus on how to detect Discovery techniques.

<!-- Image -->
<img width="1427" height="293" alt="Screenshot 2026-04-12 at 11 54 21 AM" src="https://github.com/user-attachments/assets/88f93307-1c24-41fe-880e-9356039cef01" />

<!--
I hope this walkthorugh will help you clear any confusion regarding this room and help you in your cybersecurity journey!
-->
I hope this walkthorugh will help you clear any confusion regarding this room and help you in your cybersecurity journey!

Let's Begin!

<!--
Task Template

<h2>Task [Task Number] - [Task Name]</h2>

<h3>Questions:</h3>

<h4>Question 1 - Read the Question Above</h4>
<h5>Answer: No answer needed</h5>

-->

<!------------------------------------------------------------------------------------------------------------------------------------------->

<h2>Task 1 - Introduction</h2>
<p>Linux is widely used in our networks, we must be prepared to respond to incidents</p>
<br />
<p>Unfortunately an attacker can gain initial access to our system and will start the Discovery Phase of the MITRE ATT&CK framework (more on Discovery on the next Task)</p>
<br />
<p>It is important you have knowledge of the MITRE Att&ck framework, Linux commands, and how to work with Linux logs.</p>
<p>It would also be benificial if you have finished the previous room that focuses on Initial Attack Techniques and how to detect them in Linux. </p>


<h3>Questions:</h3>

<h4>Question 1 - Read the Question Above</h4>
<h5>Answer: No answer needed</h5>

<!------------------------------------------------------------------------------------------------------------------------------------------->

<h2>Task 2 - Discovery Overview</h2>

<p>Most of the time, Inital Access is done by a bot. However Discovery is usually done by a person </p>
<p>MITRE ATTA&CK Framework defines Discovery Techniques as "techniques an adversary may use to gain knowledge about the system and internal network." </p>
<p>This concept is very reladable. The first thing we do when we are in a place we havent visited for the first time is gain an idea of where and how to get to different palces </p>
<p> The place being a relative or friends house, an airbnb, a cruise ship, the first thing we do in a new place is explore </p>
<p> Same thing here </p>
<br />
<p> The attacker has gained access using an initial access technique into a system he or she has no idea about </p>
<p> Therefore the attacker will try to do things in order to gain information on the systems, network, devices, users in an enviroment </p>
<p> We may ask questions to someone who has been to the place, a guide, house owner, or the the internet for more information on the new place, attackers however run commands</p>
<p> These commands are used in non malicious scenarios. Therefore, We must analyze if these commands are being used in a malicious way.</p>
<br />
<p> THM teaches that one command in particular that is characteristic of an attack is: <i>whoami</i> </p>
<p> Usually after there is a breach, <i>whoami</i> is the first command that the attacker runs</p>

<p>  </p>

<h3>Questions:</h3>

<p>Run systemd-detect-virt to detect the system's cloud.</p>
<h4><i>Question 1 - What is the command's output you discovered?</i></h4>

<p>The <i>systemd-detect-virt</i> command is a Linux utility command that tells you whether the system is running inside a virtualized environment and if so which one.</p>
<p>In this case, when we run the command in our vm we get the following answer</p> 

<img width="519" height="71" alt="Screenshot 2026-04-12 at 2 53 37 PM" src="https://github.com/user-attachments/assets/fc5a797e-1d77-4381-83e1-804fb9af0333" />

<p> The output tells us that the virtual machine is running inside AWS (Amazon Web Services). </p>
<h5>Answer: Amazon</h5>

<p> Now run ps aux and look for EDR or antivirus processes.</p>
<h4><i>Question 2 - What is the full path to the detected antimalware binary?</i></h4>

<p> The <i>ps aux</i> command shows all the running processes with detailed metadata on the system</p>
<p> The <i>aux</i> part of the command is now a word, but indicates three flags </p>
<ul>
  <li>a - show processes for all users</li>
  <li>u - displays the data in a user friendly format</li>
  <li>x - include processes without a terminal (background services, daemons)</li>
</ul>

<p>When you run the command, you will get a long list of all the processes currently running in the system</p> 

<img width="1425" height="665" alt="Screenshot 2026-04-12 at 3 28 56 PM" src="https://github.com/user-attachments/assets/71a16ba8-816a-41d7-9c27-0a37ee6abc34" />

<p> It can be easily intimidating finding the answer in so much output. </p>
<p> I read through all the entries in the COMMAND column to find one that sounded like an antivirus process </p>
<p> I ran into one that had malscan, which could be an abbreviation for malware scan</p>

<img width="1424" height="662" alt="Screenshot 2026-04-12 at 3 33 00 PM" src="https://github.com/user-attachments/assets/b7482b67-a6ac-4aa1-9b12-89045ff4719c" />

<h5>Answer: /var/lib/ultrasec/malscan</h5>

<p> We tan two commands which told us that this computer is running in an amazon virtual enviroment and the location of the antivirus software </p>
<p> We have discovered information of this device </p>

<!------------------------------------------------------------------------------------------------------------------------------------------->

<h2>Task 3 - Detecting Discovery</h2>

<p> Once an attacker gets general information about the system he is in, he will then try to gain more specific discovery where he is going to find what information he can take. </p>
<p> These commands could involve looking for passwords, GPU information, or network scans to find additional potential victims </p>
<br />
<p> It is important to have auditl configured to alert when some of these well known Discovery commands are being used </p>
<p> The context of an command is the key to know whether it was used maliciously: who ran it, when, how are importnat questions to answer. </p>
<p> Only way to get context of a command is through analysis, which involves analyzing the parent process </p>

<h3>Questions:</h3>

<h4>Question 1 - What is the path of the script that initiated the "hostname" command?</h4>
<p> We know that the <i>itsupport</i> user launched the <i>hostname</i> process </p>
<br />
<p> I will use ausearch to analyze the auditl logs </p>

<p> Since we are looking for the hostname process, we can simply look for this process using the -x flag </p>
<p> We will use the: <i> ausearch -i -x hostname </i> command </p>

<img width="1419" height="222" alt="Screenshot 2026-04-12 at 3 56 20 PM" src="https://github.com/user-attachments/assets/44ea69b3-9834-4c38-8354-b8c98dfc209d" />

<p> Using the results of this command, we have found that the parent process of this command is 3771 </p>

<p> If we know look for the parent process using the --pid flag, we get the following result </p>

<img width="1423" height="197" alt="Screenshot 2026-04-12 at 3 58 39 PM" src="https://github.com/user-attachments/assets/d0fcfe00-29d8-4bae-bc67-176c997be4fb" />

To recap the information that auditl provides:


<h5>Answer: No answer needed</h5>


