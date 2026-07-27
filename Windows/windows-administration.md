# Windows Administration
 
CLI, PowerShell, icacls, firewall, PsExec, Task Scheduler, and Event Viewer.
 
---
 
## Useful CLI commands
 
```
powershell start cmd -v runAs        Run cmd as administrator
driverquery                          List all installed drivers
systeminfo                           Show system details
clip                                 Copy to the clipboard
    dir | clip                       (example: copy dir output to clipboard)
assoc                                List programs and their file extensions
fc "file1" "file2"                   Show differences between two files
netstat -an                          Show open ports, IPs, states
sfc /scannow                         System File Checker (scan + repair)
attrib +h +s +r <folder>             Hide a folder
attrib -h -s -r <folder>             Unhide
tasklist                             Show open programs
taskkill /IM "task.exe" /F           Force-kill by image name
time                                 Show / change current time
more <file>                          Page through file contents
cls                                  Clear the command line
```
 
**Windows equivalent of `ls`:** `dir`
 
**Show all saved Wi-Fi passwords:**
```
for /f "skip=9 tokens=1,2 delims=:" %i in ('netsh wlan show profiles') do @echo %j | findstr -i -v echo | netsh wlan show profiles %j key=clear
```
 
---
 
## Windows CLI quick reference
 
```
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
    net user newuser /delete
```
 
---
 
## icacls — file/folder permissions
 
Windows CLI tool for viewing and changing who has permission to access files and folders. Stands for **Integrity Control Access Control List**. DACLs (Discretionary Access Control Lists) are lists attached to every file/folder that say who can access them. icacls lets you read and edit that list.
 
### Basic operations
 
```
icacls myfile.txt                       See who has access
icacls myfile.txt /grant John:(F)       Give someone full control
icacls myfile.txt /deny John:(W)        Block someone
icacls myfile.txt /remove John          Remove permissions entirely
icacls myfile.txt /reset                Reset to defaults
```
 
### Permission letters
 
| Letter | Meaning | Plain English |
|---|---|---|
| F | Full control | Can do anything |
| M | Modify | Read, write, delete |
| RX | Read & Execute | Read and run |
| R | Read only | Read only |
| W | Write only | Can edit but not read |
| D | Delete | Can delete |
| N | No access | Blocked |
 
### Inheritance flags
 
When you see `(OI)` and `(CI)` in output, they describe whether permissions propagate:
 
| Flag | Meaning |
|---|---|
| `(I)` | Inherited from parent folder |
| `(OI)` | Object Inherit — files inside get this permission too |
| `(CI)` | Container Inherit — subfolders inside get this permission too |
| `(IO)` | Inherit Only — applies to children, not the folder itself |
| `(NP)` | No Propagate — pass down one level and stop |
 
There are also advanced/granular perms (DE for delete, WO for take ownership, WDAC for change permissions), but the basic letters cover most real-world cases.
 
---
 
## Account and password policy — net accounts
 
Used to set policy settings on the local computer. Must be run **on the local machine** (unless combined with `/DOMAIN`).
 
| Option | Purpose |
|---|---|
| `/FORCELOGOFF:{minutes | NO}` | Minutes before forced logoff when account expires or logon hours expire. `NO` (default) prevents forced logoff. |
| `/MINPWLEN:length` | Minimum password length (0–14; default 6) |
| `/MAXPWAGE:{days | UNLIMITED}` | Maximum password age (1–999; default 90). Can't be less than `/MINPWAGE` |
| `/MINPWAGE:days` | Minimum days before a user can change password (0–999; default 0). Can't be more than `/MAXPWAGE` |
| `/UNIQUEPW:number` | Password history — must be unique through N changes (max 24) |
| `/DOMAIN` | Operate on a domain controller of the current domain; otherwise it's local |
 
---
 
## Windows Firewall
 
### Enable / disable
 
```powershell
# PowerShell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```
 
```
:: netsh
netsh advfirewall set allprofiles state on
netsh advfirewall set allprofiles state off
```
 
### Create a rule — allow inbound for a program
 
```powershell
New-NetFirewallRule -DisplayName "Allow Inbound Telnet" `
    -Direction Inbound `
    -Program %SystemRoot%\System32\tlntsvr.exe `
    -RemoteAddress LocalSubnet `
    -Action Allow
```
 
```
:: netsh
netsh advfirewall firewall add rule name="Allow Inbound Telnet" ^
    dir=in program=%SystemRoot%\System32\tlntsvr.exe ^
    remoteip=localsubnet action=allow
```
 
### Block outbound on a port
 
```powershell
New-NetFirewallRule -DisplayName "Block Outbound Telnet" `
    -Direction Outbound `
    -Program %SystemRoot%\System32\tlntsvr.exe `
    -Protocol TCP `
    -LocalPort 23 `
    -Action Block
```
 
### Key parameters
 
| Parameter | Sets |
|---|---|
| `-DisplayName` | Human-readable rule name |
| `-Direction` | Inbound or Outbound |
| `-Action` | Allow or Block |
| `-Protocol` | TCP, UDP, etc. |
| `-LocalPort` | Which port on this machine |
| `-RemoteAddress` | Which IPs the rule applies to |
| `-Program` | Which executable the rule applies to |
 
### Change the remote IP a rule applies to
 
```powershell
Set-NetFirewallRule -DisplayName "Allow Web 80" -RemoteAddress 192.168.0.2
```
```
netsh advfirewall firewall set rule name="Allow Web 80" new remoteip=192.168.0.2
```
 
### Manage firewall on a remote machine
 
```powershell
# View all rules on a remote computer
Get-NetFirewallRule -CimSession RemoteDevice
 
# Delete a rule on a remote computer
$RemoteSession = New-CimSession -ComputerName RemoteDevice
Remove-NetFirewallRule -DisplayName "AllowWeb80" -CimSession $RemoteSession
```
 
### Related concepts
 
- **IPsec transport mode** — encrypts traffic between two specific machines
- **IPsec tunnel mode** — encrypted tunnel between two networks (VPN)
- **Domain isolation** — uses IPsec to require domain-joined machines only can talk, blocking non-domain devices
- **Server isolation** — restricts access to specific servers to certain user groups, with mandatory encryption
- **Authenticated bypass** — trusted devices (like security scanners) bypass firewall block rules if they can prove identity
### Disable the firewall via GUI
 
Control Panel → Windows Defender Firewall → Turn Windows Defender Firewall on or off
 
---
 
## PsExec
 
A free Microsoft tool that lets you run commands and programs on another Windows computer remotely — without installing anything on the target. You just need admin credentials on the target and network access.
 
### Basic format
 
```
psexec \\remote_computer command
```
 
### Copy a program to a remote machine and run it
 
```
psexec \\remote_computer -c myprogram.exe
```
`-c` copies the file first, then executes. Without `-c`, the program has to already exist on the remote machine.
 
### Run as SYSTEM (highest privilege)
 
```
psexec -i -d -s c:\windows\regedit.exe
```
 
### Run on multiple machines
 
Put a list of computer names in a text file:
```
psexec @computers.txt ipconfig /all
```
 
Run on every computer in the domain:
```
psexec \\* hostname
```
 
---
 
## Task Scheduler — recurring tasks
 
**GUI:**
1. Open Task Scheduler
2. Create Basic Task
3. Name it
4. Trigger: choose when it runs
5. Action: choose Start a program
6. Select program/script
7. Add any flags the program needs
---
 
## Viewing processes
 
### CMD
```
tasklist                        # basic
tasklist /v                     # full detail
tasklist /m                     # DLLs each process loaded
tasklist /svc                   # services running inside each process
tasklist /fi "imagename eq firefox.exe"
tasklist /fi "pid eq 1234"
```
 
Show executable path:
```
wmic process where "name='firefox.exe'" get Name,ProcessId,ExecutablePath
```
 
### PowerShell
```powershell
Get-Process
Get-Process firefox | Select-Object Id, ProcessName, Path
```
 
### Killing processes
 
```
taskkill /im firefox.exe           # by name
taskkill /pid 5678                 # by PID
taskkill /f /im firefox.exe        # force
taskkill /f /pid 5678
```
 
---
 
## Map a network drive
 
```
:: Map a drive
net use Z: \\192.168.170.133\SharedFolder
 
:: With credentials
net use Z: \\192.168.170.133\SharedFolder /user:username password
 
:: Persist across reboots
net use Z: \\192.168.170.133\SharedFolder /persistent:yes
```
 
---
 
## Event Viewer — Windows logs
 
1. Open Event Viewer
2. Expand **Windows Logs**
3. Categories:
   - **Application** — logs from installed apps
   - **Security** — logon events, auth, policy changes (needs auditing enabled to be useful)
   - **Setup** — install/upgrade events
   - **System** — OS and driver events
   - **Forwarded Events** — logs forwarded from other machines
