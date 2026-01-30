all about process

# 🧠 Windows Internals – Process (Complete Notes)

## 1️⃣ What is a Process?

A **process** is a running instance of a program. It is a container that holds all the resources required to execute a program.

Example:

* `notepad.exe` running → process

👉 **Important:** A process itself does NOT execute code — **threads do**.

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
