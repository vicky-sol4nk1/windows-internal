all about process

# 🧠 Windows Internals – Process (Complete Notes)

<img width="1139" height="350" alt="image" src="https://github.com/user-attachments/assets/8a284c4a-b77f-41b8-8df9-2f5ec0948b4b" />

## 1️⃣ What is a Process?

A **process** is a running instance of a program. It is a container that holds all the resources required to execute a program.

Example:

* `notepad.exe` running → process

👉 **Important:** A process itself does NOT execute code — **threads do**.

---

Nice diagram 👍
This is a **classic Windows process memory + resources layout**. I’ll explain **each component** in **simple language**, plus **why it matters (exam + security)**.

---

# 🧠 Components Shown in the Process Diagram

---

## 1️⃣ **Code (Text Section)**

### 🔹 What it is

* Contains the **actual program instructions**
* Loaded from the `.exe` file
* Usually **read-only**

### 🔹 What lives here

* Machine instructions
* Program logic

### 🔹 Why important

* Prevents accidental modification
* Malware tries to **redirect execution to heap/stack**, not here

📌 **Exam tip**: Code section ≠ execution → **threads execute**

---

## 2️⃣ **Global Variables (Data Section)**

### 🔹 What it is

* Stores **global and static variables**
* Exists for the **entire lifetime of the process**

### 🔹 Examples

```c
int counter = 0;
static int flag = 1;
```

### 🔹 Why important

* Shared across all threads
* Improper use → race conditions

📌 **Security note**: Attackers may modify globals to change behavior

---

## 3️⃣ **Process Heap**

### 🔹 What it is

* Memory for **dynamic allocation at runtime**
* Grows and shrinks as needed

### 🔹 Used for

* Objects
* Buffers
* User input
* Network data

### 🔹 Why important

* Writable
* Large
* Frequently abused

📌 **Security note**:

* Heap spraying
* Payload storage
* Exploitation targets

---

## 4️⃣ **Process Resources (Open Files, Heaps)**

### 🔹 What it is

* OS-managed resources owned by the process

### 🔹 Includes

* Open files
* Registry keys
* Mutexes
* Events
* Additional heaps

### 🔹 Why important

* Managed via **handles**
* Enforces access control

📌 **Exam tip**: Resources are accessed via **handle table**

---

## 5️⃣ **Environment Block**

### 🔹 What it is

* Stores **environment variables**
* Inherited from parent process

### 🔹 Examples

```
PATH
TEMP
USERNAME
SYSTEMROOT
```

### 🔹 Why important

* Influences program behavior
* Malware checks env vars to detect sandbox/VM

📌 **Security note**: Environment-based evasion is common

---

## 6️⃣ **Threads (Thread 1 … Thread N)**

### 🔹 What they are

* **Actual execution units**
* Each thread has:

  * Its own stack
  * CPU context

### 🔹 Shared between threads

* Code
* Heap
* Global variables
* Handles

### 🔹 Why important

* Without threads → process does nothing

📌 **Golden rule**:

> **Process = container
> Thread = execution**

---

## 🔄 How Everything Connects

```
Process
 ├─ Code        (what to run)
 ├─ Globals     (shared data)
 ├─ Heap        (dynamic memory)
 ├─ Resources   (files, handles)
 ├─ Env Block   (settings)
 └─ Threads     (run the code)
```

---







## 2️⃣ Process vs Program

| Program             | Process                    |
| ------------------- | -------------------------- |
| Static file on disk | Running instance in memory |
| `.exe` file         | Has PID, memory, threads   |

---

## 3️⃣ Core Components of a Process

### 🔹 1. Virtual Address Space

* Private memory space for the process
* Contains:

  * Code section
  * Heap
  * Stack
  * Loaded DLLs
* Isolated from other processes

---

### 🔹 2. Threads

* Smallest execution unit
* A process must have **at least one thread**
* Threads share:

  * Process memory
  * Handles

👉 **Execution happens here**

---

### 🔹 3. Executable Image (.exe)

* Program file loaded into memory
* Defines entry point for execution

---

### 🔹 4. Process Control Block (EPROCESS)
it's a data structure store the information about a process
* Kernel structure used by Windows
* Stores:

  * PID
  * Parent PID
  * Priority
  * Creation time
  * Token pointer
  * status 

---

### 🔹 5. Handle Table

* Tracks OS objects accessed by the process
* Examples:

  * Files
  * Registry keys
  * Mutexes
  * Events

---

### 🔹 6. Security Token 🔐

* Defines **identity & privileges**
* Created by **LSA (LSASS.exe)**
* Contains:

  * User SID
  * Group SIDs
  * Privileges

Types:

* Primary Token
* Impersonation Token

---

### 🔹 7. Environment Block

* Stores environment variables
* Inherited from parent process

Examples:

* PATH
* TEMP
* USERNAME

---

### 🔹 8. Loaded DLLs

* Shared libraries required by process
* Examples:

  * `ntdll.dll`
  * `kernel32.dll`

---

### 🔹 9. Job Object (Optional)

* Used to group and control processes
* Can limit:

  * CPU
  * Memory
  * Runtime

Used by:

* Sandboxes
* Containers
* Malware

---

## 4️⃣ Process Creation Flow (Simplified)

```
User starts program
   ↓
CreateProcess()
   ↓
Windows loads executable
   ↓
LSA attaches security token
   ↓
Initial thread created
```

---

## 5️⃣ Process States (Status)

### 🔹 Running

* Actively executing on CPU

### 🔹 Ready

* Waiting for CPU time

### 🔹 Waiting / Blocked

* Waiting for I/O or event waiting for uploading file ,

### 🔹 Suspended

* Execution paused

### 🔹 Terminated

* Process has exited

👉 Process state depends on **thread states**

---

## 6️⃣ Parent–Child Relationship

* Every process (except System) has a parent
* Child process inherits:

  * Security token
  * Environment variables

Example:

```
explorer.exe → cmd.exe
```

---

## 7️⃣ User Mode vs Kernel Mode

| User Mode             | Kernel Mode          |
| --------------------- | -------------------- |
| Limited access        | Full hardware access |
| Applications run here | OS core runs here    |
| Safer                 | Powerful             |

---

## 8️⃣ Process Inspection Tools

### 🔹 Built-in

* Task Manager
* PowerShell: `Get-Process`

### 🔹 Sysinternals

* Process Explorer
* Process Monitor

---

## 9️⃣ Security & Malware Perspective 🔥

* Token stealing → privilege escalation
* Process injection → malicious code execution
* Hollowing → replace legit process memory
* Suspended processes → evasion

---


Alright, this is **MITRE ATT&CK + Windows Internals gold** 🥇
I’ll explain **what it is**, **why attackers use it**, and a **realistic scenario** — **no commands, no how-to abuse**, just solid understanding (perfect for TryHackMe, exams, blue team).

---

# 🧠 T1055 – Process Injection (Parent Technique)

## 🔹 What is Process Injection?

**Process Injection** means:

> Attacker runs **malicious code inside a legitimate process** instead of creating a new obvious one.

🎯 Goal:

* Stealth
* Evasion
* Privilege abuse

👉 Instead of `evil.exe`, malware hides inside `explorer.exe`, `svchost.exe`, etc.

---

## 🔥 Why attackers love it

* Antivirus trusts legit processes
* Blends into normal activity
* Harder to detect than new processes

---

## 🧪 General Scenario

1. Malware already running (initial access)
2. Finds a trusted process
3. Injects code into it
4. Legit process now executes attacker code

---

---

# 🧬 T1055.012 – **Process Hollowing**

## 🔹 What is Process Hollowing?

A special type of injection where:

> A **legitimate process is started**, then its **original code is removed**, and **malicious code is placed instead**.

💀 The process looks legit, but its soul is gone.

---

## 🧠 How it works (conceptually)

* Start legit process (e.g. `svchost.exe`)
* Suspend it
* Remove original memory
* Insert malicious payload
* Resume process

🧠 Result:

```
svchost.exe (name) ❌
malware.exe (code) ✅
```

---

## 🎯 Why attackers use hollowing

* File name looks trusted
* Parent-child relationship looks normal
* Bypasses simple AV rules

---

## 🧪 Realistic Scenario

💼 Corporate environment:

* User opens phishing email
* Dropper launches `svchost.exe`
* Hollowing replaces its memory
* Malware communicates with C2
* Blue team sees:

  * svchost.exe making suspicious network calls

🚩 Red flag:

* Legit process doing **non-legit behavior**

---

## 🔍 Blue Team Detection Clues

* Mismatch between:

  * Process name
  * Loaded memory sections
* Abnormal parent process
* Suspended → resumed processes

---

---

# 🎭 T1055.013 – **Process Masquerading**

⚠️ This is **often confused with hollowing** — but it’s different.

---

## 🔹 What is Process Masquerading?

> Malware **pretends to be a legitimate process by name or path**, but it is actually a malicious executable.

❗ No injection needed.

---

## 🧠 Key idea

* Looks legit
* Actually fake

Examples:

```
svch0st.exe   (zero instead of o)
explorer .exe (extra space)
C:\Windows\svchost.exe (wrong path)
```

---

## 🎯 Why attackers use masquerading

* Trick users
* Trick admins
* Trick basic monitoring tools

---

## 🧪 Realistic Scenario

🧑‍💻 User downloads cracked software:

* Malware saved as `chrome_update.exe`
* Icon copied from Chrome
* User runs it
* Process looks “normal” in Task Manager
* Malware runs freely

🚩 Red flag:

* Process path mismatch
* Unsigned binary
* Wrong parent process

---

## 🔍 Blue Team Detection Clues

* Legit name, wrong location
* Suspicious spelling
* No digital signature
* Unexpected startup behavior

---

# 🆚 Quick Comparison (Very Important)

| Technique            | Injection? | Legit Process Used | Code Replaced |
| -------------------- | ---------- | ------------------ | ------------- |
| Process Injection    | ✅          | Yes                | No            |
| Process Hollowing    | ✅          | Yes                | ✅ Yes         |
| Process Masquerading | ❌          | Looks legit        | N/A           |

---

