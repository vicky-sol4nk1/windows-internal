# windows-internal


Windows Internals refers to the deep, low-level functioning of the Microsoft Windows operating system — how the kernel, processes, threads, memory manager, I/O manager, registry, object manager, sessions, tokens, handles, system calls (syscalls), drivers, ETW, sessions, Virtual Trust Levels (VTLs / VBS), and many other core components actually work under the hood.

The classic (and still best) book for this is Windows Internals by Mark Russinovich (now in 7th edition, split into Part 1 & Part 2).

Below is a **clear, structured explanation of Windows OS Architecture**, explained from **top to bottom**, the way it’s taught in **Windows Internals, OS exams, and cybersecurity**.

---

# Windows OS Architecture (Overview)

Windows follows a **layered and hybrid architecture**, divided mainly into **User Mode** and **Kernel Mode**, to provide **security, stability, and performance**.

---

## High-Level Architecture Diagram (Textual)

```
+-----------------------------+
|        User Applications    |
|  (Chrome, CMD, PowerShell)  |
+-----------------------------+
|     User Mode Subsystems    |
|  Win32, .NET, POSIX (old)   |
+-----------------------------+
|        System Call Layer    |
|          (ntdll.dll)        |
+=============================+
|        Kernel Mode          |
+-----------------------------+
| Executive | Kernel | HAL    |
| Drivers   | Memory | I/O    |
+-----------------------------+
|          Hardware           |
+-----------------------------+
```

---

## 1. User Mode

User mode is where **applications run safely** with limited privileges.

### Components in User Mode

#### 1️⃣ User Applications

* GUI apps (Notepad, Browser)
* Console apps (CMD, PowerShell)
* Services (some run in user mode)

👉 Cannot access hardware directly.

---

#### 2️⃣ Environment Subsystems

These provide **APIs** to applications.

* **Win32 Subsystem** (Most important)

  * `kernel32.dll`
  * `user32.dll`
  * `gdi32.dll`
* **.NET Framework**
* **WOW64** (32-bit apps on 64-bit Windows)

👉 Win32 is the **primary Windows programming interface**.

---

#### 3️⃣ Subsystem DLLs

* Translate **API calls → system calls**
* Communicate with kernel

Example:

```
CreateFile() → kernel32.dll → ntdll.dll → Kernel
```

---

## 2. System Call Interface (Boundary)

### ntdll.dll (Very Important)

* Contains **system call stubs**
* Switches CPU from **User Mode → Kernel Mode**
* Uses CPU instructions:

  * `syscall` (x64)
  * `sysenter` (x86)

This is the **security boundary**.

---

## 3. Kernel Mode

Kernel mode has **full control** of the system.

---

### 3.1 Windows Executive

High-level OS services:

| Component                  | Function                           |
| -------------------------- | ---------------------------------- |
| Object Manager             | Handles objects (files, processes) |
| Memory Manager             | Virtual memory, paging             |
| Process Manager            | Process & thread creation          |
| I/O Manager                | Device & file I/O                  |
| Security Reference Monitor | Access control                     |
| Cache Manager              | Disk caching                       |
| Plug and Play Manager      | Device detection                   |
| Power Manager              | Power states                       |

---

### 3.2 Kernel (Low-Level Core)

Handles:

* Thread scheduling
* Interrupt handling
* Exception handling
* Context switching
* Synchronization

👉 Works **below** the Executive.

---

### 3.3 Device Drivers

* Run in kernel mode
* Interface between OS and hardware
* Examples:

  * Disk drivers
  * Network drivers
  * USB drivers

⚠️ Buggy drivers = **BSOD**

---

### 3.4 HAL (Hardware Abstraction Layer)

* `hal.dll`
* Hides hardware differences from kernel
* Allows Windows to run on different CPUs/platforms

---

## 4. Hardware Layer

Includes:

* CPU
* RAM
* Disk
* Network card
* GPU
* USB devices

Only **kernel-mode code** can access hardware directly.

---

## 5. Why This Architecture Is Important

### Stability

* App crash ≠ OS crash

### Security

* Malware in user mode is limited
* Kernel exploits are harder

### Performance

* Direct hardware access only when required

---

## 6. Architecture from Security / Pentesting View

### Attack Surfaces

* Drivers
* IOCTL handlers
* System calls
* Token privileges

### Privilege Levels

```
User Mode → Kernel Mode → SYSTEM
```

---



---

## 8. Summary Table

| Layer        | Runs In     | Purpose              |
| ------------ | ----------- | -------------------- |
| Applications | User Mode   | User tasks           |
| Subsystems   | User Mode   | APIs                 |
| System Calls | Boundary    | Mode switch          |
| Executive    | Kernel Mode | OS services          |
| Kernel       | Kernel Mode | CPU control          |
| Drivers      | Kernel Mode | Hardware access      |
| HAL          | Kernel Mode | Hardware abstraction |

---



Here are the **most important Windows directories and files** that SOC analysts, threat hunters, DFIR investigators, and red teamers/pentesters should know well. These paths are standard on **Windows 10 / 11 / Server 2022 / 2025** (as of 2026).

I've focused on locations frequently involved in **persistence**, **execution**, **credential access**, **defense evasion**, **forensics**, and **living-off-the-land** attacks.

### Core System Directories & Files

| Full Path                                      | What it contains / Purpose                                                                 | Security Relevance (Blue / Red Team) |
|------------------------------------------------|---------------------------------------------------------------------------------------------|--------------------------------------|
| `C:\Windows`                                   | Main OS installation folder — core system files, resources, drivers, configs               | Baseline for legitimate files; attackers drop files here for masquerading |
| `C:\Windows\System32`                          | Critical 64-bit system executables, DLLs, drivers (e.g. `cmd.exe`, `powershell.exe`, `svchost.exe`, `lsass.exe`) | Heavily monitored; common LOLBins location; DLL hijacking target |
| `C:\Windows\SysWOW64`                          | 32-bit system files on 64-bit Windows (redirects 32-bit apps)                              | Often abused for 32-bit LOLBins or sideloading |
| `C:\Windows\System32\config`                   | Registry hive files: `SYSTEM`, `SOFTWARE`, `SAM`, `SECURITY`, `DEFAULT`                    | Critical for forensics (offline hive analysis); attackers target for credential dumping |
| `C:\Windows\System32\config\SAM`               | Local user account hashes (NTLM) — protected by LSASS                                      | Credential access target (e.g., secretsdump, Mimikatz) |
| `C:\Windows\System32\config\SYSTEM`            | Hardware & service configuration hive                                                      | Autostart locations, mounted devices forensics |
| `C:\Windows\System32\Tasks`                    | Scheduled Tasks definitions (.job files are legacy; modern = XML in this folder)           | Very common persistence (Schtasks /schtasks.exe) |
| `C:\Windows\Temp`                              | System-wide temporary files                                                                | Common staging/drop location; often cleaned but abused |
| `C:\Windows\Prefetch`                          | .pf files — prefetch cache for faster app launch                                           | Forensic gold: execution evidence (even if file deleted) |
| `C:\Windows\Minidump` or `C:\Windows\MEMORY.DMP` | Crash dump files (if enabled)                                                            | Memory forensics (lsass dump, kernel crash analysis) |

### User-Specific & Roaming Data

| Full Path                                              | What it contains / Purpose                                                                 | Security Relevance |
|--------------------------------------------------------|---------------------------------------------------------------------------------------------|--------------------|
| `C:\Users\<username>`                                  | User profile root (Desktop, Documents, Downloads, etc.)                                    | User artifacts, downloads staging |
| `C:\Users\<username>\AppData\Roaming`                  | Roaming profile data — app settings that roam with user (e.g., browser profiles)           | Classic persistence (startup folders, .lnk, scripts) |
| `C:\Users\<username>\AppData\Local`                    | Local (non-roaming) app data — caches, temp files, large data                             | Browser cache, tool artifacts, evasion staging |
| `C:\Users\<username>\AppData\Local\Temp`               | Per-user temporary files (very common drop/staging location)                               | Extremely abused by malware/dropper (often %TEMP%) |
| `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup` | Per-user startup folder — .lnk / .exe / scripts run at login                     | Classic persistence (easy to detect/abuse) |
| `C:\Users\<All Users>\Desktop` or symlink equivalents | Legacy All Users desktop (now junction to public)                                          | Rare but still checked for persistence |

### Shared / All-Users Locations

| Full Path                                      | What it contains / Purpose                                                                 | Security Relevance |
|------------------------------------------------|---------------------------------------------------------------------------------------------|--------------------|
| `C:\ProgramData`                               | Application data shared by **all users** (hidden by default)                               | Very common persistence (scheduled tasks, configs, binaries) |
| `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup` | All-users startup folder — items run for every user at login                    | High-value persistence location |
| `C:\Program Files`                             | 64-bit installed applications                                                              | Legitimate; sideloading / DLL search order abuse |
| `C:\Program Files (x86)`                       | 32-bit installed applications                                                              | Same as above — frequent target for DLL hijacking |
| `C:\Program Files\Common Files`                | Shared libraries/components for multiple apps                                              | Less common but used in some persistence |

### Other High-Interest Forensic / Attack Locations

- `C:\$Recycle.Bin` — Deleted files (SIDs subfolders) → forensic recovery
- `C:\PerfLogs` — Performance logs (sometimes abused)
- `C:\Windows\System32\spool\drivers\x64\3` — Print spooler drivers (Driver persistence / PrintNightmare-style attacks)
- `C:\Windows\System32\wbem\Repository` — WMI repository → persistence via WMI event subscriptions
- `%PUBLIC%` → `C:\Users\Public` — Public downloads / desktop (easy drop location)


