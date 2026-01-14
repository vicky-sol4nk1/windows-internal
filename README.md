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


