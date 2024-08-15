# Hardware Breakpoint Hooking in C for Windows

## Overview

This project implements a hardware breakpoint hooking system in C, specifically designed for the Windows operating system. It enables low-level function interception by leveraging hardware breakpoints through the CPU's debug registers (Dr0-Dr3). This technique allows for precise function hooking without modifying the target function's memory, providing a non-invasive and robust method for debugging and monitoring function calls.

## Features

### 1. **Hardware Breakpoints**
   - Set and remove hardware breakpoints on functions using the debug registers (Dr0-Dr3).
   - Breakpoints are used to intercept the execution of specific functions and redirect their flow to custom detour functions.

### 2. **Detour Functions**
   - Allows for custom detour functions that execute in place of the original function when a breakpoint is hit.
   - Detour functions can manipulate the parameters and return values of the original function by directly modifying the CPU’s thread context.

### 3. **Thread Context Manipulation**
   - Access and modify the parameters and return values of the hooked functions.
   - Supports both x64 and x86 architectures by adjusting register values (e.g., `RCX`, `RDX`, `R8`, `R9` on x64).

### 4. **Critical Section Management**
   - Ensures thread safety when setting or removing breakpoints.
   - Utilizes Windows Critical Sections to protect shared resources in a multithreaded environment.

### 5. **Vectored Exception Handling**
   - Utilizes Vectored Exception Handling (VEH) to manage single-step exceptions triggered by hardware breakpoints.
   - Ensures that only the intended hardware breakpoints trigger detour function execution.

### 6. **Error Handling and Reporting**
   - Comprehensive error handling and reporting mechanisms for all critical operations.
   - Provides detailed error messages to assist with debugging and development.

## Example Usage

The project includes example functions that demonstrate how to hook commonly used Windows API functions like `MessageBoxA` and `Sleep`.

```c
VOID MessageBoxADetour(PCONTEXT pThreadCtx) {
    printf("[i] MessageBoxA's Old Parameters: \n");
    printf("\t> %s \n", (char*)GETPARM_2(pThreadCtx));
    printf("\t> %s \n", (char*)GETPARM_3(pThreadCtx));

    RETURN_VALUE(pThreadCtx, MessageBoxA(NULL, "This is the hook", "MessageBoxADetour", MB_OK | MB_ICONEXCLAMATION));
    BLOCK_REAL(pThreadCtx);
    CONTINUE_EXECUTION(pThreadCtx);
}

VOID SleepDetour(PCONTEXT pThreadCtx) {
    printf("[i] Sleep's Old Parameters: \n");
    printf("\t> %d \n", (DWORD)GETPARM_1(pThreadCtx));

    BLOCK_REAL(pThreadCtx);
    CONTINUE_EXECUTION(pThreadCtx);
}

In the main function, these hooks are installed and uninstalled, demonstrating their use:

int main() {
    // Initialize 
    if (!InitializeHardwareBPVariables())
        return -1;

    // Install Hooks
    SetHardwareBreakingPnt(MessageBoxA, MessageBoxADetour, Dr0);
    SetHardwareBreakingPnt(Sleep, SleepDetour, Dr1);

    // Trigger Hooked Functions
    MessageBoxA(NULL, "This Wont Execute", "Will it ?", MB_OK);
    Sleep(-1);

    // Uninstall Hooks
    RemoveHardwareBreakingPnt(Dr0);
    RemoveHardwareBreakingPnt(Dr1);

    // Cleanup
    UnintializeHardwareBPVariables();
    return 0;
}


Installation and Setup
Clone the repository:
git clone https://github.com/your-username/hardware-breakpoint-hooking.git
cd hardware-breakpoint-hooking
Compile the code:

Use a C compiler like GCC or MSVC to compile the code on a Windows machine.
Run the example:

Execute the compiled binary to see the hooking in action with MessageBoxA and Sleep.
Requirements
Windows Operating System (x86 or x64)
C Compiler (GCC, MSVC, etc.)
Windows SDK (for access to Windows APIs)
