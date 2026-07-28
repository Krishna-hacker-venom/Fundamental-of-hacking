# Windows Security

**Author:** Security Analysis & Penetration Testing  
**Last Updated:** 2026  
**Target Audience:** Security Professionals, System Administrators, Ethical Hackers

---

## Table of Contents

1. [Windows Current Versions](#1-windows-current-versions)
2. [File System & NTFS](#2-file-system--ntfs)
3. [Alternate Data Streams (ADS)](#3-alternate-data-streams-ads)
4. [Windows System32 Folder](#4-windows-system32-folder)
5. [User Accounts, Profiles & Permissions](#5-user-accounts-profiles--permissions)
6. [User Account Control (UAC)](#6-user-account-control-uac)
7. [Settings and Control Panel](#7-settings-and-control-panel)
8. [Task Manager](#8-task-manager)

---

## 1. Windows Current Versions

### Overview
Microsoft releases different editions of Windows to cater to different user requirements and use cases.

### Main Editions

| Edition | Target Users | Key Features | BitLocker |
|---------|--------------|--------------|-----------|
| **Home** | Individual/Consumer Users | Basic functionality, Cortana, Windows Update | ❌ Not Available |
| **Pro** | Small Business/Power Users | Remote Desktop, Encryption, Group Policy | ✅ Available |
| **Enterprise** | Large Organizations | Advanced security, volume licensing | ✅ Available |
| **Education** | Educational Institutions | Academic pricing, student features | ✅ Available |

### BitLocker Encryption

**Definition:** BitLocker is Windows full-disk encryption technology that protects data on your device.

#### Key Points:

- **Availability:** Only available in Pro, Enterprise, and Education editions
- **Not in Home Edition:** Home edition users cannot use BitLocker
- **Purpose:** Encrypts entire drive to prevent unauthorized access to stored data
- **Protection Mechanism:** Uses AES (Advanced Encryption Standard) encryption with 128-bit or 256-bit keys
- **TPM Requirement:** Works best with Trusted Platform Module (TPM) 2.0 chip
- **Recovery Key:** Essential to save BitLocker recovery key to avoid data loss

#### Enabling BitLocker

**Method 1: Via Settings**
```
Settings → System → About → Device Encryption/BitLocker settings
```

**Method 2: Via Control Panel**
```
Control Panel → System and Security → BitLocker Drive Encryption
```

#### Security Implications:
- Prevents data theft from physical device access
- Requires PIN or USB key in some configurations
- Recovery key must be stored securely (Microsoft Account, USB, or print)

---

## 2. File System & NTFS

### What is NTFS?

**Full Form:** New Technology File System

**Definition:** NTFS is a modern file system developed by Microsoft that replaced FAT32, offering advanced features for security, reliability, and capacity.

### NTFS Key Features

| Feature | Description | Benefit |
|---------|-------------|---------|
| **File Permissions** | Granular access control | Enhanced Security |
| **File Compression** | Reduce storage space | Efficient Storage |
| **Encryption (EFS)** | Encrypt individual files | Data Protection |
| **Journaling** | Prevents data corruption | Reliability |
| **Large File Support** | Files larger than 4GB | Better for Modern Data |
| **Quota Management** | Limit disk space per user | Resource Control |

### NTFS Permissions

NTFS provides 7 main permission types that control what users can do with files and folders:

#### The 7 Permission Types

**1. Full Control**
- Highest permission level
- User can: Read, Write, Execute, Delete, Change Permissions, Take Ownership
- Example: Folder creator or administrator role

**2. Modify**
- Nearly equivalent to Full Control (cannot change permissions or take ownership)
- User can: Read, Write, Execute, Delete files/folders
- Example: Project manager with most privileges but limited permission changes

**3. Read & Execute**
- User can: Open and run files, view folder contents
- User cannot: Create new files, delete, or modify existing files
- Example: End users accessing application files

**4. List Folder Contents**
- Applies to folders only
- User can: See folder contents and subfolders
- User cannot: See file contents or execute files
- Example: Directory listing without file access

**5. Read**
- User can: View and open files, read file contents
- User cannot: Modify, delete, or create new files
- Example: Read-only document access

**6. Write**
- User can: Create new files and folders, modify existing files
- User cannot: Read or delete existing files
- Example: Log file writing without reading existing logs

**7. Special Permissions**
- Advanced, granular permissions for specific scenarios
- See detailed section below

#### Setting NTFS Permissions

**Step-by-Step Guide:**

1. Right-click on file/folder → **Properties**
2. Click on **Security** tab
3. Click **Edit** button
4. Select user/group and modify permissions
5. Check/uncheck desired permission boxes
6. Click **Apply** → **OK**

**Example Scenario:**
```
Folder: C:\ProjectData
Action: Give "Developers" group Read & Execute permission

Steps:
1. Right-click C:\ProjectData → Properties
2. Security tab → Edit
3. Select "Developers" group
4. Check "Read & Execute" checkbox
5. Apply changes
```

---

### Special Permissions (In Detail)

**Definition:** Special Permissions provide granular, advanced control beyond the standard 7 permissions. They offer precise control over file system operations.

#### Advanced Special Permissions

| Permission | What It Allows | Security Use Case |
|-----------|----------------|--------------------|
| **Full Control** | Complete control over object and permissions | Admin, Owner roles |
| **Traverse Folder / Execute File** | Move through folders or run executables | Application access |
| **List Folder / Read Data** | View folder contents or read file data | Directory browsing |
| **Read Attributes** | Read basic file properties | File metadata access |
| **Read Extended Attributes** | Read advanced file properties | Extended metadata access |
| **Create Files / Write Data** | Create new files or modify existing | Write access control |
| **Create Folders / Append Data** | Create folders or append to files | Append-only scenarios |
| **Write Attributes** | Modify basic file properties | Metadata modification |
| **Write Extended Attributes** | Modify extended properties | Advanced metadata changes |
| **Delete Subfolders and Files** | Delete items within folder | Selective deletion control |
| **Delete** | Delete the object itself | Object deletion control |
| **Read Permissions** | View permission settings | Audit and compliance |
| **Change Permissions** | Modify NTFS permissions | Administrative control |
| **Take Ownership** | Assume ownership of object | Administrator privilege |

#### Accessing Special Permissions

```
Right-click → Properties → Security → Advanced → Edit → Select User/Group → Edit Permissions
```

#### Real-World Example: Secure Folder Setup

**Scenario:** Create folder with restricted access

```
Folder: C:\Confidential
Requirements: Only Manager can Read/Write, Team Lead can Read-only

Step 1: Remove inherited permissions
→ Advanced → Disable inheritance

Step 2: Set permissions for Manager
→ Add "Manager" group
→ Allow: Modify, Write, Read & Execute

Step 3: Set permissions for Team Lead
→ Add "TeamLead" group
→ Allow: Read & Execute (only)

Step 4: Remove Everyone permission
→ Select "Everyone" → Remove
```

---

## 3. Alternate Data Streams (ADS)

### What are Alternate Data Streams?

**Definition:** Alternate Data Streams (ADS) is a feature specific to NTFS file system that allows a single file to contain multiple independent data streams.

**Key Concept:** Think of a file as having multiple "invisible" containers of data attached to it.

### ADS Fundamentals

#### Standard Data Stream vs Alternate Data Streams

```
Traditional File Structure:
file.txt → Contains visible data (single stream)

NTFS with ADS:
file.txt → Main data ($DATA) + Multiple hidden streams
          ├─ Stream 1 (hidden)
          ├─ Stream 2 (hidden)
          └─ Stream 3 (hidden)
```

#### The $DATA Stream

**$DATA** is the default, primary data stream that contains the visible file content.

### How ADS Works

| Aspect | Details |
|--------|---------|
| **Default Stream** | Every file has at least one $DATA stream (visible content) |
| **Additional Streams** | Can contain multiple additional named streams |
| **Visibility** | Additional streams are hidden from regular file exploration |
| **File Size** | Explorer shows only $DATA size, hiding other streams |
| **Syntax** | `filename.txt:streamname` |

#### Example ADS Structure

```
example.txt:$DATA          → Visible file content (main data)
example.txt:zone.identifier → Windows metadata
example.txt:hidden_data    → Custom hidden data stream
example.txt:malware_code   → Malicious payload (security threat)
```

### Creating and Accessing ADS

#### Creating an ADS

**Using Command Line (PowerShell):**

```powershell
# Create ADS with hidden data
echo "hidden content" > C:\example.txt:hidden_stream

# Create another ADS
echo "secret data" > C:\example.txt:secret
```

**Using Command Prompt:**

```cmd
# Write to alternate stream
type secret.txt > normalfile.txt:hidden
```

#### Reading ADS Content

```powershell
# View specific stream
Get-Content C:\example.txt:hidden_stream

# List all streams in a file
dir C:\example.txt /r

# Or using PowerShell
Get-Item C:\example.txt -Stream *
```

#### Deleting ADS

```powershell
# Remove specific stream
Remove-Item C:\example.txt -Stream hidden_stream

# Delete all alternate streams
Get-Item C:\example.txt -Stream * | ForEach-Object { Remove-Item $_ }
```

### Malware Use of ADS

#### Security Threat Vector

**How Attackers Exploit ADS:**

1. **Data Hiding**
   - Embed malicious code in alternate streams
   - Remains invisible to antivirus scans (if not configured to check ADS)
   - File appears normal in Explorer

2. **Persistence Mechanism**
   - Store malware in ADS of legitimate Windows files
   - Execute hidden payload while system file remains unaltered
   - Difficult to detect with standard monitoring

3. **Exfiltration**
   - Hide sensitive data in ADS
   - Compress and encode stolen information
   - Difficult to audit via standard file monitoring

#### Real-World Example: Malware in ADS

```cmd
# Attacker hides malware executable in Word document
C:\Users\victim\Documents\Report.docx:malware.exe

# File appears normal: 
   - File Explorer shows Report.docx with normal size
   - No obvious indication of hidden malware

# Payload executes via:
   - VBA macros
   - Scheduled task
   - Registry entry
```

### Detecting ADS Misuse

#### Detection Methods

**Method 1: PowerShell Detection**

```powershell
# Find all files with ADS
Get-ChildItem -Path C:\ -Recurse -Force | ForEach-Object {
    $ads = Get-Item $_ -Stream * -ErrorAction SilentlyContinue
    if ($ads.Count -gt 1) {
        Write-Host "File with ADS found: $_"
        $ads | Select-Object PSPath, Stream
    }
}
```

**Method 2: Command Prompt**

```cmd
# Find files with streams in directory
dir /r C:\Windows\System32\

# Look for streams besides :$DATA
```

**Method 3: Third-Party Tools**

- **Streams.exe** (Sysinternals) - Detects ADS in files
- **Alternate Data Stream Manager**
- **NTFS Streams Detector**

#### Security Hardening

```powershell
# Remove all ADS from a directory
Get-ChildItem -Path C:\Users -Recurse -Force |
ForEach-Object {
    Get-Item $_ -Stream * -ErrorAction SilentlyContinue |
    Where-Object { $_.Stream -ne ':$DATA' } |
    Remove-Item
}
```

---

## 4. Windows System32 Folder

### Overview

**System32 Folder Location:** `C:\Windows\System32\`

**Purpose:** Core system libraries, drivers, and executable files essential for Windows OS operation.

### Environment Variables

#### What are Environment Variables?

**Definition:** Environment variables are named text values that store system configuration information accessible by applications and scripts.

#### %WINDIR% Variable

**%WINDIR%** = Path to Windows installation directory (typically `C:\Windows`)

### Key System32 Components

| Component | Type | Purpose |
|-----------|------|---------|
| **cmd.exe** | Executable | Command prompt interpreter |
| **powershell.exe** | Executable | PowerShell console |
| **services.exe** | Executable | Windows service host |
| **lsass.exe** | Executable | Local Security Authority |
| **svchost.exe** | Executable | Service host processes |
| **kernel32.dll** | Library | Core Windows functions |
| **ntdll.dll** | Library | Native API library |
| **advapi32.dll** | Library | Advanced Windows API |

### Environment Variables Stored in System32

#### Information Stored

```
PATH          → Directories for executable files
TEMP          → Temporary file location
TMP           → Alternative temp location
PROCESSOR_ARCHITECTURE → CPU type (x86, x64, ARM)
NUMBER_OF_PROCESSORS   → Count of logical processors
APPDATA       → User application data folder
PROGRAMFILES  → Program installation directory
WINDIR        → Windows installation directory
SYSTEMROOT    → Windows system root directory
```

#### Viewing Environment Variables

**Method 1: Command Prompt**

```cmd
# View all environment variables
set

# View specific variable
echo %WINDIR%
echo %PATH%
echo %NUMBER_OF_PROCESSORS%
```

**Method 2: PowerShell**

```powershell
# View all environment variables
Get-ChildItem env:

# View specific variable
$env:WINDIR
$env:PATH
$env:PROCESSOR_ARCHITECTURE
```

**Method 3: GUI Settings**

```
System Properties → Advanced → Environment Variables
```

### Security Implications

#### Risks and Hardening

| Risk | Impact | Mitigation |
|------|--------|-----------|
| **Path Traversal** | Attackers manipulate PATH variable | Monitor/restrict PATH changes |
| **DLL Injection** | Malicious DLL in System32 | File integrity monitoring |
| **Privilege Escalation** | Exploit System32 executables | Apply security patches |
| **System Manipulation** | Modify environment variables | Lock down administrative access |

#### Securing System32

```powershell
# Check System32 folder permissions (should be restrictive)
icacls C:\Windows\System32 /grant:r Users:F  # NOT recommended
icacls C:\Windows\System32 /grant:r Users:RX # Read & Execute only

# Verify file signatures in System32
Get-ChildItem C:\Windows\System32\*.exe | 
ForEach-Object {
    $sig = Get-AuthenticodeSignature $_
    if ($sig.Status -ne "Valid") {
        Write-Host "Unsigned executable found: $_"
    }
}
```

---

## 5. User Accounts, Profiles & Permissions

### Types of User Accounts

Windows supports two primary user account types:

#### 1. Administrator Account

**Definition:** Account with full system access and permissions.

**Capabilities:**
- ✅ Install/uninstall software
- ✅ Modify system settings
- ✅ Change user account permissions
- ✅ Access all files and folders
- ✅ Change network settings
- ✅ Create/delete user accounts
- ✅ Modify registry
- ✅ Run system commands

**Use Case:** System administrators, IT personnel

**Security Concern:** Higher risk if compromised

#### 2. Standard User Account

**Definition:** Account with restricted permissions.

**Capabilities:**
- ✅ Run most applications
- ✅ Access assigned files/folders
- ✅ Change personal password
- ❌ Install software (limited)
- ❌ Modify system settings
- ❌ Change other user accounts
- ❌ Install drivers
- ❌ Modify Windows registry

**Use Case:** Regular end-users

**Security Benefit:** Limits damage from malware/attacks

### User Rights and Actions

| Action | Administrator | Standard User |
|--------|---------------|---------------|
| Install Software | ✅ Yes | ❌ No (may prompt) |
| Change System Date/Time | ✅ Yes | ❌ No |
| Modify Firewall Rules | ✅ Yes | ❌ No |
| View Event Logs | ✅ Yes | ✅ Limited |
| Backup System Files | ✅ Yes | ❌ No |
| Format Disk Partition | ✅ Yes | ❌ No |
| Access All User Files | ✅ Yes | ❌ Only own files |
| Change Network Settings | ✅ Yes | ❌ No |

### Local Users and Groups Manager (lusrmgr.msc)

**Full Name:** Local Users and Groups Manager  
**Location:** Built-in Windows MMC snap-in

#### Accessing lusrmgr.msc

**Method 1: Run Dialog**

```
Press: Windows Key + R
Type: lusrmgr.msc
Press: Enter
```

**Method 2: Computer Management**

```
Right-click This PC → Manage → Local Users and Groups
```

**Method 3: Command Line**

```cmd
lusrmgr.msc
```

#### Main Components

**Local Users:**
- View all user accounts
- Create new user accounts
- Delete user accounts
- Reset user passwords
- Enable/disable accounts
- Set account properties

**Local Groups:**
- View all groups
- Create new groups
- Add/remove members
- Set group permissions

#### Practical Tasks

**Task 1: Create New User Account**

```
1. Launch lusrmgr.msc
2. Right-click Users folder → New User
3. Enter Username, Password
4. Click Create
5. Close dialog
```

**Task 2: Add User to Administrators Group**

```
1. Open lusrmgr.msc
2. Double-click Users folder
3. Right-click desired user → Add to Group
4. Click Add... → Type user name → OK
5. Select Administrators → Apply → OK
```

**Task 3: Reset Forgotten Password**

```
1. Open lusrmgr.msc
2. Right-click user account
3. Select Set Password
4. Enter new password
5. Confirm and apply
```

**Task 4: Disable User Account**

```
1. Open lusrmgr.msc
2. Right-click user account
3. Check "Account is disabled"
4. Apply changes
```

---

## 6. User Account Control (UAC)

### What is UAC?

**Full Form:** User Account Control

**Definition:** Windows security feature that requires permission/confirmation before allowing administrative-level changes to the system.

### UAC Purpose and Function

#### Key Objectives

1. **Prevent Unauthorized System Changes**
   - Blocks applications from making system-wide modifications without user consent
   - Protects against silent installation of malware

2. **Privilege Escalation Prevention**
   - Standard users cannot escalate to administrator without approval
   - Requires administrator credentials to proceed

3. **User Awareness**
   - Notifies users of administrative actions
   - Gives users opportunity to cancel dangerous operations

### UAC Prompt Levels

#### UAC Settings (4 Levels)

| Level | Description | When UAC Prompts |
|-------|-------------|-----------------|
| **Level 4** | Always Notify | Any admin action, apps, settings |
| **Level 3** | Default (Most Secure) | Admin apps and manual system changes |
| **Level 2** | Notify (Less Secure) | Admin apps only, not all changes |
| **Level 1** | Never Notify | Never prompts (not recommended) |

### Configuring UAC

**Method 1: GUI Settings**

```
Settings → System → About → Advanced system settings
→ Advanced tab → User Account Control → Change Settings
```

**Method 2: Group Policy (Pro/Enterprise)**

```
Win + R → gpedit.msc
Computer Configuration → Windows Settings → Security Settings
→ Local Policies → Security Options
→ Find UAC settings
```

**Method 3: Registry (PowerShell)**

```powershell
# Disable UAC prompts (not recommended)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableUAC" -Value 0

# Enable UAC
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableUAC" -Value 1
```

### Security Implications

#### Benefits of UAC

- ✅ Prevents unauthorized system changes
- ✅ Protects against drive-by downloads
- ✅ Reduces malware impact
- ✅ Maintains system integrity

#### Risks if Disabled

- ❌ Any application can modify system settings
- ❌ Malware gains unrestricted access
- ❌ No user notification for dangerous actions
- ❌ Increased privilege escalation risk

---

## 7. Settings and Control Panel

### Overview

**Primary Locations for Windows Configuration:**

1. **Settings App** (Modern interface - Windows 10/11)
2. **Control Panel** (Legacy interface - Still available)

### Key Differences

| Aspect | Settings | Control Panel |
|--------|----------|---------------|
| **Interface** | Modern, Metro design | Traditional, categorized view |
| **Navigation** | Search-based, intuitive | Menu categories, nested options |
| **Performance** | Optimized for modern systems | Legacy, slower navigation |
| **Accessibility** | Better for touch devices | Better for keyboard navigation |

### Critical Configuration Areas

#### 1. System Security Settings

**Location:** Settings → Privacy & Security

**Configurations:**
- Windows Defender settings
- Firewall configuration
- Windows Update options
- App permissions
- Device encryption status

#### 2. Network Configuration

**Location:** Settings → Network & Internet

**Configurations:**
- WiFi connections
- VPN setup
- Proxy configuration
- Firewall rules (basic)

#### 3. User Accounts

**Location:** Settings → Accounts

**Configurations:**
- Password management
- Account security
- Account recovery options
- PIN/Biometric setup
- Family options

#### 4. Privacy Settings

**Location:** Settings → Privacy & Security

**Options:**
- App permissions (camera, microphone, location)
- Tracking (telemetry)
- App history
- Advanced privacy options

#### 5. Windows Update

**Location:** Settings → System → About → Advanced options

**Management:**
- Check for updates
- Update schedule configuration
- Restart timing
- Optional updates selection

### Control Panel Legacy Options

**Still Important for Administration:**

```
Control Panel → Administrative Tools
├─ Event Viewer
├─ Services
├─ Task Scheduler
├─ Device Manager
├─ Disk Management
└─ Computer Management
```

#### Accessing Control Panel

**Method 1: Direct Search**

```
Press Windows Key → Type "Control Panel" → Select
```

**Method 2: Run Dialog**

```
Windows Key + R → Type "control" → Enter
```

**Method 3: Settings Redirect**

```
Settings → System → About → Advanced system settings
```

---

## 8. Task Manager

### What is Task Manager?

**Definition:** System utility providing real-time monitoring and management of running processes, applications, and system performance.

### Opening Task Manager

#### Methods to Launch

**Method 1: Keyboard Shortcut (Fastest)**

```
Ctrl + Shift + Esc  → Opens directly
```

**Method 2: Ctrl+Alt+Delete**

```
Press: Ctrl + Alt + Delete
Select: Task Manager
```

**Method 3: Right-click Taskbar**

```
Right-click taskbar → Task Manager
```

**Method 4: Command Line**

```cmd
taskmgr.exe
```

### Task Manager Tabs

#### 1. Processes Tab

**Purpose:** View and manage running applications and background processes

**Columns Displayed:**
- **Name** → Process/application name
- **Status** → Running, Suspended, etc.
- **User** → Account running the process
- **CPU** → Processor usage percentage
- **Memory** → RAM usage in MB/GB
- **Disk** → Read/Write disk activity
- **GPU** → Graphics processor usage
- **Network** → Network bandwidth usage

**Actions:**
- End Process (force close)
- Right-click for more options
- Priority adjustment (high/normal/low)
- Affinity (processor assignment)

#### 2. Performance Tab

**Purpose:** Monitor real-time system performance metrics

**Metrics Shown:**

| Metric | Measured In | Indicates |
|--------|------------|-----------|
| **CPU** | Percentage | Processor workload |
| **Memory** | GB/MB | RAM utilization |
| **Disk** | Percentage | Disk I/O activity |
| **GPU** | Percentage | Graphics processor load |
| **Ethernet** | Mbps | Network speed |

**Use Case:** Identify performance bottlenecks

#### 3. App History Tab

**Purpose:** View resource usage statistics over time

**Information:**
- CPU time spent
- Network data transferred
- Metered network usage
- GPU time
- Long-running applications

**Use Case:** Identify problematic applications

#### 4. Startup Tab

**Purpose:** Manage applications launching at system startup

**Controls:**
- Enable/Disable startup applications
- View startup impact (high/medium/low)
- Open file location
- Search online

**Security Tip:** Disable unnecessary startup programs to improve boot time and reduce attack surface

#### 5. Services Tab

**Purpose:** View and manage Windows background services

**Information:**
- Service name
- Status (Running/Stopped)
- Group
- Trigger details

**Actions:**
- Start/Stop services
- Restart services
- Open Services management console

**⚠️ Warning:** Disabling critical services can break Windows

#### 6. Details Tab (Advanced)

**Purpose:** Detailed process information and management

**Advanced Options:**
- **Priority** → Change CPU priority (Real-time/High/Above Normal/Normal/Below Normal/Low)
- **Affinity** → Assign to specific CPU cores
- **UAC Virtualization** → Enable/Disable
- **DEP/NX** → Memory protection status
- **Command Line** → Full command used to launch process

**Security Uses:**
- Investigate suspicious processes
- Check command-line arguments for malware
- Monitor parent-child process relationships

### Performance Monitoring

#### CPU Monitoring

```
Task Manager → Performance → CPU tab

Shows:
- Overall utilization percentage
- Number of logical processors
- Current speed (GHz)
- Core temperatures (if available)
- L1/L2/L3 cache information
```

#### Memory Monitoring

```
Task Manager → Performance → Memory tab

Shows:
- Total installed RAM
- Currently in use (GB)
- Available memory
- Committed memory
- Cached memory
- Compression ratio (if enabled)
```

### Security Applications

#### Process Analysis

**Identify Suspicious Processes:**

```
Task Manager → Processes tab

Look for:
- Unknown application names
- Unusual parent processes
- High CPU/Memory with no user activity
- Processes with strange file paths
- Unsigned or unverified processes
```

#### Detect Malware Activity

```
Signs of Malware in Task Manager:
- Unusual process names (system32.exe, lsass.exe variants)
- High memory usage with no apparent function
- Multiple instances of same process
- Processes from temp folders
- Network activity without user action
```

#### Resource Hogging Investigation

```
1. Sort by CPU → Identify CPU hogs
2. Right-click suspicious process → Search online
3. Note file location → Check legitimacy
4. Check Properties → Company name verification
5. End process if confirmed malicious
```

### Practical Security Tasks

#### Task 1: Identify and Close Malicious Process

```
1. Open Task Manager (Ctrl+Shift+Esc)
2. Sort Processes by CPU/Memory (descending)
3. Identify suspicious high-usage processes
4. Right-click → Open file location
5. Check file properties → Digital signature
6. If unsigned/unknown → Right-click → End Task
7. Manually delete file if necessary
```

#### Task 2: Disable Startup Malware

```
1. Task Manager → Startup tab
2. Look for unknown applications
3. Right-click suspicious entry → Disable
4. Open file location if available
5. Delete startup file if malicious
6. Restart system to confirm
```

#### Task 3: Monitor Performance for Attacks

```
1. Open Task Manager
2. Performance tab → Monitor CPU, Memory, Network
3. Detect unusual spikes
4. Click processes → Sort by usage
5. Investigate source of activity
6. Check Performance → Ethernet for network attacks
```

---

## Summary Table: Quick Reference

| Concept | Key Point | Security Impact |
|---------|-----------|-----------------|
| **BitLocker** | Pro+ editions only | Critical encryption protection |
| **NTFS Permissions** | 7 standard types | Access control foundation |
| **Special Permissions** | Granular control | Advanced security configuration |
| **ADS** | Hidden data streams | Malware hiding risk |
| **System32** | Core OS files | Attack surface consideration |
| **Administrator** | Full access | High privilege risk |
| **Standard User** | Limited access | Best practice for daily use |
| **lusrmgr.msc** | User management | Administrative control |
| **UAC** | Privilege elevation prompt | Malware prevention |
| **Settings/Control Panel** | Configuration center | System hardening location |
| **Task Manager** | Process monitoring | Real-time threat detection |

---

## Recommended Security Practices

### Best Practices Checklist

- [ ] Use Standard User accounts for daily tasks
- [ ] Enable BitLocker on sensitive devices
- [ ] Configure NTFS permissions with least privilege principle
- [ ] Monitor Task Manager for suspicious processes
- [ ] Keep UAC enabled (Level 3 recommended)
- [ ] Regularly audit user accounts via lusrmgr.msc
- [ ] Scan for ADS in system folders quarterly
- [ ] Apply Windows Updates promptly
- [ ] Review Startup tab for unauthorized applications
- [ ] Monitor System32 folder for file modifications

---

## Resources and Further Reading

### Official Microsoft Documentation
- [Microsoft Windows Security](https://docs.microsoft.com/en-us/windows/security/)
- [NTFS Permissions Guide](https://docs.microsoft.com/en-us/windows/security/filesystem-security)
- [BitLocker Documentation](https://docs.microsoft.com/en-us/windows/security/encryption-data-protection)

### Security Analysis Tools
- Sysinternals Suite (Process Explorer, Streams.exe)
- Wireshark (Network monitoring)
- Autoruns (Startup program analysis)
- Disk2vhd (System imaging)

---
