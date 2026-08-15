---
title: "Simple DLL injector with GUI (dll-inj)"
date: 2023-09-15
tags: ["in development", "challenge"]
series: "0x13hrafnulf's Challenges"
categories: ["Cybersecurity"]
---

*in development* [repository](https://github.com/0x5bjorn/dll-inj)

Simple DLL injector with GUI written in C++ for educational purposes only

![demo](/images/posts-media/dll-inj/demo.gif)

## Code organization

- **Application** - application and app window management
- **Injection** - DLL injection implemention using Windows API
- **Proc** - process and process modules querying handler
- **GUI** - GUI management using [Dear ImGui](https://github.com/ocornut/imgui)
- **OpenDialogBox** - file selection dialog box using Windows API

## DLL injection

One of the most known *code injection* techniques in Windows environments is **Dynamic Link Library (DLL) injection**. The core idea is to execute arbitrary external code in the context of running processes. This can be done by misusing legitimate Windows API functionality and its designed features. Injection is used for the following list of malicious operations:

- *Data Theft* - injected code has access to the memory and data available to the target process
- *Evasion Tactics* - injected code is hidden within "trusted" process, making it diffucult to detect by process-based monitoring
- *Privilege Escalation* - injected code is executed with elevated permissions

This project is a simple implementation of the most common and classic DLL injection technique using `CreateRemoteThread` function from Windows API. This function allows us to create thread in remote target process, where it will load malicious DLL upon execution. The whole process consists of the following steps:

### Step 1. Target Process Access

The first step is to open handle to target process with required permissions. This is done with the help of `OpenProcess` function by providing specific access permissions for thread creation and virutal memory operations, or `PROCESS_ALL_ACCESS`.

### Step 2. Memory Allocation and Writing

A thread is basically a unit of execution within the process which only operates within its *virtual address space*. Every process has its own *virtual address space* (for isolation and security reasons), which is a memory allocated for that process by OS to work with. Since a thread created in the target process will load DLL, it needs to know its path. So the DLL path needs to be in the process memory. For that, `VirtualAllocEx` and `WriteProcessMemory`, to allocate memory in remote process and to write data into allocated memory correspondingly, are used by providing the process handle acquired with appropriate access rights.

### Step 3. Remote Thread Creation and Library Loading

The final step is to create a thread using `CreateRemoteThread` which expects process handle, an address of a function to be executed and the additional function arguments. The provided function must exist in or at least be accessible to the remote process. In this case, the address of the `LoadLibraryA` function with the DLL path as its argument needs to be provided for the thread to load DLL. This function is defined in `Kernel32` library (one of the essential libraries mapped into every process memory in Windows OS), which luckily is mapped similarly in each process memory while OS is running. Therefore, the address of `LoadLibraryA` in *injector* process and target process are identical.

Once all the steps are done, the code located in the DLL’s `DllMain` will be executed in the address space of the target process.

<!-- ## Dynamic data updates
*Coming soon...* -->

## Improvements

- Implement thread pool for thread reuse (currently a simple thread creation and deletion mechanism is utilized)

## References

- Windows process and its modules(dll) listing - <https://learn.microsoft.com/en-us/windows/win32/toolhelp/taking-a-snapshot-and-viewing-processes>
- Native Windows File Selection prompt - <https://learn.microsoft.com/en-us/windows/win32/learnwin32/example--the-open-dialog-box>
- Process injection -
  - <https://www.ired.team/offensive-security/code-injection-process-injection/dll-injection>
  - <https://ctfcracker.gitbook.io/process-injection/process-injection-part-2>
  - <https://tbhaxor.com/createremotethread-process-injection/>
- ImGUI - <https://github.com/ocornut/imgui>
