# Inside a Computer System

Simple, clear notes on how a computer is built and what happens when it boots — written for both a normal user and a hacker.

---

## Table of Contents

- [Core Hardware Components](#core-hardware-components)
  - [Motherboard](#motherboard)
  - [RAM](#ram)
  - [PSU](#psu)
  - [CPU](#cpu)
  - [GPU](#gpu)
  - [HDD and SSD](#hdd-and-ssd)
- [Key Terms You Must Know](#key-terms-you-must-know)
  - [UEFI](#uefi)
  - [BIOS Chip](#bios-chip)
  - [CMOS](#cmos)
  - [JTAG](#jtag)
  - [Debug Ports](#debug-ports)
- [What Happens When You Press the Power Button](#what-happens-when-you-press-the-power-button)

---

## Core Hardware Components

---

### Motherboard

**Think of it as:** The roads of a city. Every component in the computer connects to and communicates through the motherboard. Without it, nothing talks to anything else.

**Normal User**
- It is the large green circuit board inside your computer
- Every part — CPU, RAM, GPU, storage — plugs into it

**Hacker**
- Physical access to the motherboard means access to everything — BIOS chip, CMOS battery, JTAG pins, and debug ports
- This is ground zero for hardware-level attacks that survive OS reinstalls and antivirus scans

---

### RAM

**Think of it as:** Your desk. Whatever you are currently working on sits on your desk. When you go home (power off), the desk is cleared — everything is gone.

**Normal User**
- RAM holds everything the computer is currently running — open apps, browser tabs, files in use
- More RAM = more things open at once without slowdown
- It is wiped completely when the computer turns off

**Hacker**
- RAM is a goldmine — it holds passwords, encryption keys, and session tokens in plaintext while the system is running
- Cold boot attack: freeze the RAM chips after power-off, dump their contents — data lingers briefly even without power
- Fileless malware lives entirely in RAM — it never touches the disk, making it invisible to most antivirus tools
- Tool: Volatility — used to analyze RAM dumps and extract credentials

---

### PSU

**Think of it as:** The heart. It pumps power to every component in the system. If the heart stops, everything stops.

**Normal User**
- Converts wall power (AC) to the lower voltages (DC) that computer parts need
- A failing PSU causes random crashes or prevents the system from booting at all

**Hacker**
- Rarely targeted directly, but a modified PSU can act as a hidden hardware implant
- Power analysis attacks: by measuring tiny fluctuations in power draw during encryption, an attacker can mathematically extract secret keys — this is a side-channel attack

---

### CPU

**Think of it as:** The brain. Every instruction, calculation, and decision the computer makes runs through the CPU.

**Normal User**
- Executes billions of instructions per second
- Faster CPU = faster and more responsive system

**Hacker**
- Spectre and Meltdown: CPU-level vulnerabilities that let one program read memory belonging to another program or the OS — a fundamental architectural flaw
- Privilege rings: the CPU enforces layers of trust — Ring 0 is the kernel (full control), Ring 3 is where normal apps run. Most exploits aim to move from Ring 3 to Ring 0 (privilege escalation)
- Microcode: low-level CPU instructions that can be updated — a compromised microcode update is nearly undetectable

---

### GPU

**Think of it as:** A factory floor with thousands of workers doing simple tasks simultaneously. The CPU is one genius solving complex problems; the GPU is thousands of workers each doing one tiny job at the same time.

**Normal User**
- Handles all graphics rendering — everything displayed on screen goes through the GPU
- Also used for video editing, 3D work, and AI processing

**Hacker**
- Password cracking: Hashcat uses the GPU to test billions of password hashes per second — far faster than the CPU alone
- Cryptojacking: malware silently uses the victim's GPU to mine cryptocurrency
- GPU memory is not scanned by antivirus — researchers have demonstrated malware that hides and executes inside GPU memory

---

### HDD and SSD

**Think of it as:** A filing cabinet. It stores everything permanently — files, OS, programs — even when the power is off. Unlike the desk (RAM), the filing cabinet keeps its contents when you leave.

**Normal User**
- HDD: stores data on spinning magnetic disks — slower, cheaper, larger capacity
- SSD: stores data on flash memory chips — faster, more durable, more expensive
- Both retain data without power

**Hacker**
- HDD: deleted files are not actually erased immediately — they sit in unallocated space until overwritten. Tools like Autopsy and PhotoRec can recover them
- SSD: uses wear leveling (spreads writes across cells) — deleted data may still exist in cells not yet reused
- HDD firmware attacks: malware embedded in the drive's own firmware survives full OS reinstall and disk formatting (e.g., Equation Group implants)
- Bootkits: malware living in the MBR (Master Boot Record) or EFI partition loads before the OS — invisible to all OS-level security tools

---

## Key Terms You Must Know

---

### UEFI

**One line:** UEFI is the first software that runs when your computer powers on — before the OS, before everything.

**The simple version**
- When you press the power button, the CPU does not know what to do yet
- UEFI is the set of instructions stored on a chip on the motherboard that tells the CPU what to do first
- It wakes up all the hardware, checks that everything works, then finds and loads the operating system
- Think of it as the computer's startup checklist — it runs before Windows, Linux, or any OS ever loads

**Why hackers care**
- UEFI runs with more privilege than the OS — code here cannot be stopped or scanned by antivirus
- UEFI rootkits survive OS reinstalls, disk replacements, and factory resets — the only fix is reflashing the firmware chip
- Real-world examples: LoJax (first UEFI rootkit found in the wild), CosmicStrand, MosaicRegressor
- UEFI has its own network stack — vulnerabilities like LogoFAIL allow attacks before the OS ever loads

**Access method:** Press F2, F10, DEL, or ESC at startup (varies by manufacturer)

---

### BIOS Chip

**One line:** The physical chip on the motherboard where UEFI firmware is stored.

**The simple version**
- UEFI is software — the BIOS chip is the hardware it lives on
- It is a small flash memory chip soldered to the motherboard
- It keeps its contents even when the computer is off (non-volatile)
- Think of it as a USB drive that is permanently glued to the motherboard and runs before anything else

**Why hackers care**
- The chip can be read and written directly using an SPI flash programmer (e.g., CH341A with a clip)
- With physical access, an attacker can dump the firmware, modify it, and reflash it — no OS interaction required
- Extracted firmware can be reverse engineered to find hidden backdoors or vulnerabilities
- Supply chain attacks target this chip before devices reach the buyer

---

### CMOS

**One line:** A tiny piece of memory powered by a coin battery that remembers your BIOS settings when the computer is off.

**The simple version**
- Your BIOS settings (boot order, system clock, hardware config) need to be saved somewhere even when the computer is unplugged
- CMOS is that storage — a small memory area kept alive by the round coin cell battery on the motherboard
- UEFI is the firmware code. BIOS chip is where that code lives. CMOS is where your settings are saved.
- If the CMOS battery dies, your computer forgets the date and time every time it powers off

**Why hackers care**
- Removing or shorting the CMOS battery resets all BIOS settings to factory defaults — including the BIOS password
- BIOS password set to block USB boot or firmware access? Pull the CMOS battery, wait 30 seconds, reinsert — password gone
- This works on most consumer and SMB hardware with physical case access
- After a CMOS reset, the system clock resets — a forensic artifact that physical tampering occurred
- Defense: full disk encryption (BitLocker, LUKS) — even if BIOS password is bypassed and attacker boots from USB, the encrypted drive cannot be read

---

### JTAG

**One line:** A hardware debug interface that gives direct access to a chip's internals — CPU registers, memory, flash storage — all without the OS being involved.

**The simple version**
- Think of JTAG as a maintenance hatch directly into the chip
- Engineers use it during development to pause the CPU mid-execution, inspect memory, and step through code one instruction at a time
- It connects via a small set of pins (usually a 4 or 5 pin header) on the circuit board
- Most production devices still have these pins physically present — manufacturers just do not advertise them

**Why hackers care**
- JTAG access = full control over the chip with no software authentication required
- Used to extract firmware from embedded devices (routers, IP cameras, industrial systems) where no other extraction method works
- Can bypass all software-level protections — lockscreens, encrypted storage, authentication systems
- Many IoT devices ship with JTAG headers exposed on the PCB, forgotten by the manufacturer
- Tools: OpenOCD (open-source debugger), J-Link, JTAGulator (automates finding JTAG pins on unknown boards)

---

### Debug Ports

**One line:** Hardware interfaces left on devices by developers for testing — and frequently exploited by hackers to get direct low-level access.

**The simple version**
- During development, engineers need to see what is happening inside the device — so they build in debug ports
- These are physical connection points (pins or headers on the circuit board) that expose the internals of the system
- They are meant to be used in the lab and disabled or removed before the product ships — but often they are not
- The most common types:

| Port | What it does |
|---|---|
| JTAG | Full chip debug — read/write CPU, memory, flash (covered above) |
| UART | Serial console — often drops into a Linux shell or bootloader prompt |
| SWD | 2-pin ARM debug interface, similar function to JTAG |
| USB Debug | Used for Android ADB, Windows kernel debugging |

**Why hackers care**
- UART is the most commonly exploited debug port in IoT hacking
- Connect a USB-to-serial adapter (FTDI) to the UART pins on a router or camera PCB, open a terminal at 115200 baud — you often land directly in a root shell with no password required
- Debug ports bypass every software security control — no login, no authentication, no logs
- Finding them: look for unpopulated 3-4 pin headers on PCBs, probe with a multimeter or logic analyzer
- Tools: Bus Pirate, FTDI adapter, minicom, screen, PuTTY

---

## What Happens When You Press the Power Button

```
Power Button --> PSU supplies power --> UEFI starts --> POST --> Boot device selected --> Bootloader --> OS loads
```

---

### Step 1 — Press the Power Button

**What happens:** A signal goes to the PSU. The PSU starts supplying power to all components.

**Analogy:** You are waking up from sleep. The moment you breathe in, your heart starts pumping blood and your body systems come online.

**Hacker angle:**
- This is the earliest point an attacker can interfere with a system
- Evil maid attack: an attacker with brief physical access modifies hardware or boot media before the machine powers on
- Anti-forensics: some security systems are designed to wipe RAM the moment an unexpected power-on is detected

---

### Step 2 — Firmware Starts (UEFI)

**What happens:** The CPU starts executing code stored in the UEFI chip. UEFI initializes all hardware components — CPU, RAM, storage, peripherals.

**Analogy:** Your brain becomes conscious. It checks that your arms, legs, and senses are all connected and working before you try to do anything.

**Hacker angle:**
- UEFI runs before the OS — it has more privilege than anything that comes after it
- UEFI rootkits (LoJax, CosmicStrand) live here and survive OS reinstalls and disk replacements
- No UEFI password set? An attacker can walk up, change boot order, boot from USB — done in 30 seconds
- Tool to analyze UEFI firmware: chipsec, UEFITool

---

### Step 3 — POST (Power-On Self Test)

**What happens:** UEFI runs a self-check on all hardware. It confirms that the CPU, RAM, and storage are present and functioning. If something fails, it throws an error (beep codes or on-screen messages) and stops.

**Analogy:** Your body running a systems check after waking — making sure your heart, lungs, and limbs all respond before you stand up.

**Hacker angle:**
- POST runs before any OS security control exists — OS-level antivirus and firewalls are completely irrelevant here
- Hardware-level denial of service: remove or damage a component to cause POST failure — the machine will not boot regardless of what software defenses are installed
- POST error codes leak hardware configuration information useful during physical reconnaissance

---

### Step 4 — Select Boot Device

**What happens:** UEFI checks its priority list and looks for a device containing a bootable OS. It tries each device in order (e.g., USB first, then SSD) and loads from the first valid one it finds.

**Analogy:** Your brain decides where to look for your memory and personality — it checks storage locations in order of priority until it finds one that works.

**Hacker angle:**
- No UEFI password = trivial boot order change = boot from attacker USB = full access to the machine
- PXE boot (network boot): if enabled, an attacker on the same network can serve a malicious OS image over the network
- Booting Kali Linux from USB bypasses the installed OS entirely — login screen, local user accounts, and unencrypted data all become accessible
- Secure Boot is the defense: only allows bootloaders signed with trusted keys. Bypasses exist but require more effort.

---

### Step 5 — Bootloader

**What happens:** The bootloader is a small program on the boot device that copies the OS from storage into RAM and hands control to it. Once done, UEFI steps back and the OS takes over management of all hardware.

**Analogy:** The bridge between sleep and full consciousness — the part where your memories load and you become fully yourself again.

**Hacker angle:**
- Bootkits infect the bootloader — they run before the OS and can hide processes, files, and network connections from everything that loads after them, including antivirus
- GRUB (Linux bootloader): on unprotected systems, an attacker can interrupt boot and drop into a root shell directly from the bootloader — no password needed
- BCDEdit (Windows): can be used to add malicious boot entries or redirect the boot process
- TPM (Trusted Platform Module): records a cryptographic hash of each boot stage — any tampering changes the hash and can be detected. High-value target for attackers trying to plant persistent implants.
- This is the last pre-OS attack surface — once the OS loads, software defenses finally come online

---

