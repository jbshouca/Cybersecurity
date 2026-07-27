# Linux Fundamentals
 
Core shell concepts, basic commands, and bash scripting.
 
---
 
## Shell basics
 
A **shell** is a program that provides a command-line interface for interacting with an OS. Bash is one of the most common shells.
 
- `ps` — determine the shell type in use
- `date` — displays the current date
- `pwd` — displays the present working directory
- `ls` — lists the contents of the current directory
- `echo` — prints a string of text, or value of a variable
- `man <command>` — refers you to the command's manual
### Switching users
 
```bash
su - <username>
```
 
---
 
## Common bash commands
 
| Command | Purpose |
|---|---|
| `cd` | Change the directory |
| `ls` | List directory contents |
| `mkdir` | Create a new directory |
| `touch` | Create a new file |
| `rm` | Remove a file or directory |
| `cp` | Copy a file or directory |
| `mv` | Move or rename a file or directory |
| `echo` | Print text to the terminal |
| `cat` | Concatenate and print contents of a file |
| `grep` | Search for a pattern in a file |
| `chmod` | Change permissions of a file or directory |
| `sudo` | Run a command with administrative privileges |
| `df` | Display available disk space |
| `history` | Show previously executed commands |
| `ps` | Show information about running processes |
 
---
 
## Bash scripting
 
Bash scripting automates processes in Linux via a file containing commands executed together.
 
### File setup
 
- Bash scripts end with `.sh`
- Scripts start with a **shebang** — `#!` followed by the path to the interpreter
- The shebang must be the first line and tells the shell to execute the script via bash
```bash
#!/bin/bash
```
 
Find the interpreter path with:
```bash
which bash
```
 
Make a script executable:
```bash
chmod u+x script.sh
```
 
### Variables
 
```bash
country=Pakistan
echo $country
# Pakistan
 
new_country=$country
echo $new_country
# Pakistan
```
 
### Reading user input
 
`read -p` pauses the script to wait for input. Whatever is typed is stored in the named variable.
 
```bash
read -p "Enter your name: " name
echo "Hello $name"
```
 
### Case statements (decision gates)
 
`case $variable_name in` compares the stored value against a list of patterns and executes the block that matches first.
 
```bash
case $choice in
    1) echo "Option one" ;;
    2) echo "Option two" ;;
    *) echo "Unknown" ;;
esac
```
 
### Comparison operators
 
- `-gt` — greater than: `[ $num -gt 0 ]`
- `-lt` — less than: `[ $num -lt 0 ]`
### Redirection
 
```bash
# Append to a file
echo "More text." >> output.txt

# Append to a file requiring elevated privileges
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf

# Overwrite / create a new file with output
ls > files.txt
```
 
### Debugging flags
 
- `set -x` at the top enables debug mode — bash prints each command before executing it
- `set -e` causes the script to exit immediately when any command fails
---
 
## Cron (job scheduling)
 
Cron runs commands or scripts on a schedule. Edit your crontab with `crontab -e`, list with `crontab -l`.
 
### Cron field pattern
 
```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, 0 and 7 are Sunday)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```
 
### Examples
 
| Schedule | Meaning |
|---|---|
| `0 0 * * *` | Run at midnight every day |
| `*/5 * * * *` | Run every 5 minutes |
| `0 6 * * 1-5` | Run at 6 AM Monday to Friday |
| `0 0 1-7 * *` | Run on the first 7 days of every month |
| `0 12 1 * *` | Run on the first day of every month at noon |
| `30 2 * * *` | Run at 2:30 AM every day |
| `@reboot /path/to/script.sh` | Run at boot |
 
---
 
## Personal project ideas
 
1. **nmap command helper** — Bash script that prints the most commonly needed nmap commands based on user input for IPs and Ports.
2. **File/directory existence checker** — Script that checks if a directory exists, and if it does, checks whether a specific file exists within it. Creates the file if missing. Prints status for each check.
