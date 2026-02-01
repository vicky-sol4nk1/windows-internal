1. What is Win32?

Win32 is the main programming interface (API) used to build native Windows applications.
It allows software to communicate directly with the Windows operating system.

Application → Win32 API → Windows OS

Win32 API = the friendly menu that normal programs use to ask Windows to do things.

You write a program (Notepad, Chrome, game, calculator…)
You want to do something useful → open file, show window, play sound, read keyboard, draw something on screen, create folder, etc.
You don't talk directly to hardware or deep OS parts → instead you call ready-made functions like CreateFile(), MessageBox(), CreateWindow(), WriteFile(), etc.

[/images/wind32.png
](https://github.com/vicky-sol4nk1/windows-internal/blob/main/images/win32.png)

| Area          | Example                        |
| ------------- | ------------------------------ |
| Windows & GUI | Create windows, buttons, menus |
| Memory        | Allocate and manage memory     |
| Files         | Read and write files           |
| Processes     | Create and control processes   |
| Threads       | Multi-tasking inside a process |
| Input         | Keyboard and mouse             |
| Networking    | Windows sockets (Winsock)      |
| Security      | Access tokens, permissions     |


| Layer                          | Where is it?          | Can touch hardware directly? | Can crash whole computer?  | Examples                                  | Runs as…                    |
| ------------------------------ | --------------------- | ---------------------------- | -------------------------- | ----------------------------------------- | --------------------------- |
| **User Application**           | Top                   | ❌ No                         | ❌ No (only itself crashes) | Your `.exe` program                       | Normal application          |
| **Win32 API**                  | Still top (User Mode) | ❌ No                         | ❌ No                       | `kernel32.dll`, `user32.dll`, `gdi32.dll` | API libraries               |
| **System Calls**               | Transition point      | ❌ No                         | ❌ No                       | `syscall` instruction                     | Mode switch (User → Kernel) |
| **Kernel**                     | Bottom                | ✅ Yes (full access)          | ✅ Yes (Blue Screen)        | `ntoskrnl.exe`, device drivers            | Core OS                     |
| **Physical Memory / Hardware** | Bottom-most           | —                            | —                          | RAM, CPU, SSD, screen, keyboard           | Real hardware               |
