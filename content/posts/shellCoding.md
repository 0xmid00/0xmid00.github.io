+++
title = 'Shellcoding from Scratch'
date = 2024-06-29T20:02:06+01:00
draft = false
description = "A beginner's guide to shellcoding"
slug = ""
authors = ["Ahmed Boureghida"]
tags = ["Binary Exploitation", "Shellcoding"]
categories = ["Binary Exploitation"]
externalLink = ""
series = []
+++

# Shellcoding from Scratch

This post serves as my notes for the ECPPT course preparation.


- **Shellcode**: Small piece of code injected into vulnerable applications to gain control.
- **Execution Methods**:
    - **EIP Manipulation**: Change the instruction pointer to execute shellcode.
    - **SEH Frames**: Use exception handling to run shellcode.

## Types of Shellcode

- **Local Shellcode**: Increases privileges on the local machine (privilege escalation).
- **Remote Shellcode**: Sent through the network to exploit remote systems.
    - **Connect-back**: Attacker receives a connection from the victim.
    - **Bind Shell**: Attacker connects to a specific port on the victim.
    - **Socket Reuse**: Uses existing connections to communicate.

## Staged Shellcode

- **Stage 1**: Small initial code that downloads/executed larger payload (Stage 2).
    - **Egg-hunt**: Finds the larger payload in memory.
    - **Omelet**: Combines multiple smaller payloads.

## Null-Free and Encoded Shellcode

- **Null-Free**: Must not contain `\x00` (NULL bytes), as they terminate strings.
- **Encoding**: Techniques like alphanumeric encoding bypass restrictions.


## Creating Basic Shellcode

1. **Example**: Open Windows Calculator.
2. **Steps**:
    1. Write a C program to test shellcode.
    2. Inject shellcode and check execution.
    
```c
char code[] = "shellcode here";
int main() {
    int (*func)();
    func = (int (*)()) code;
    (int)(*func)();
}
```

## Creating Advanced Shellcode from Scratch

### What is Shellcode?
- **Shellcode**: A small piece of code used to exploit vulnerabilities and execute commands on a target system, such as opening a command prompt.

### Key Components
- **`cmd`**: Command to open the command prompt.
- **`open`**: Parameter that tells Windows to open the specified program.
- **`ShellExecute`**: A function in `Shell32.dll` that runs programs or opens files.

### Endianness
- **Little Endian**:
  - In little endian systems (like most x86 architectures), bytes are stored with the least significant byte first.
  - Example: The hex `636d6400` would be stored in memory as `\x00\x64\x6d\x63`.

### Finding `ShellExecute` Address
- Use tools like **arwin** to find the address:
  ```bash
  arwin.exe Shell32.dll ShellExecute
  ```
- Example address: `0x762BD970`.

### Converting Strings to Hexadecimal
- Use Python with the `pwntools` library to convert strings to hex:
  ```python
  from pwn import *

  cmd = "cmd\0"
  open_cmd = "open\0"

  cmd_hex = enhex(cmd.encode())
  open_hex = enhex(open_cmd.encode())

  print(f"cmd hex: {cmd_hex}")
  print(f"open hex: {open_hex}")
  ```

- **Output**:
  - `"cmd\0"`: `636d6400`
  - `"open\0"`: `6f70656e00`

### Order of Storing
- When storing hexadecimal values in shellcode, reverse the byte order:
  - `"cmd\0"`: `"\x00\x64\x6d\x63"` (from `636d6400`)
  - `"open"`: `"\x6e\x65\x70\x6f"` (from `6f70656e`)
  - We did not add `\x00` because `"\x68"` instruction (PUSH) only copies 4 bytes, fitting "open" perfectly without the null terminator.

### Explanation of Memory Allocation
- **Memory Blocks**:
  - The `PUSH` instruction works with 4-byte blocks.
  - `"open"` fits into one 4-byte block (`\x6f\x70\x65\x6e`), so no need for an additional null byte.
  - `"cmd\0"` uses `\x00` to complete 4 bytes: (`\x00\x64\x6d\x63`).

### Shellcode Creation Steps

1. **Push Strings to the Stack**:
   - Push strings in reverse order due to the stack's LIFO (Last In, First Out) nature.
   ```assembly
   "\x68\x00\x64\x6d\x63"  // PUSH (0x68) "cmd\0" (hex: 636d64\x00) in reverse order
   "\x68\x6e\x65\x70\x6f"  // PUSH (0x68) "open" (hex: 6f70656e) in reverse order
   ```

2. **Store Pointers**:
   - Save the memory addresses of the strings in registers (`EBX` for `"cmd"` and `ECX` for `"open"`).
   ```assembly
   "\x8B\xDC"  // MOV EBX, ESP (0x8B) (hex: DC)
   "\x8B\xCC"  // MOV ECX, ESP (0x8B) (hex: CC)
   ```

3. **Push Additional Arguments**:
   - Push parameters for `ShellExecute`:
     - `3`: Maximizes the window.
     - `0`: NULL values for unused parameters.
   ```assembly
   "\x6A\x03"  // PUSH 3 (hex: 6A) (3: 0x03)
   "\x33\xC0"  // XOR EAX, EAX (hex: 33C0)
   "\x50"      // PUSH EAX (hex: 50) (NULL: 0x00)
   "\x50"      // PUSH EAX (hex: 50) (NULL: 0x00)
   "\x53"      // PUSH EBX (hex: 53) (pointer to "cmd")
   "\x51"      // PUSH ECX (hex: 51) (pointer to "open")
   "\x50"      // PUSH EAX (hex: 50) (NULL: 0x00)
   ```

4. **Call `ShellExecute`**:
   - Move the address of `ShellExecute` into a register (`EAX`) and call it.
   ```assembly
   "\xB8\x70\xD9\x2B\x76"  // MOV EAX, 0x762BD970 (hex: B8 70 D9 2B 76)
   "\xFF\xD0"              // CALL EAX (hex: FF D0)
   ```

### Complete Shellcode
- This shellcode opens the command prompt using `ShellExecute`:
```c
char code[] =
    "\x68\x00\x64\x6d\x63"  // PUSH (0x68) "cmd\0" (hex: 636d64\x00) in reverse order
    "\x8B\xDC"              // MOV EBX, ESP (0x8B) (hex: DC)
    "\x6A\x00"              // PUSH 0 (hex: 6A) (NULL terminator)
    "\x68\x6e\x65\x70\x6f"  // PUSH (0x68) "open" (hex: 6f70656e) in reverse order
    "\x8B\xCC"              // MOV ECX, ESP (0x8B) (hex: CC)
    "\x6A\x03"              // PUSH 3 (hex: 6A) (3: 0x03)
    "\x33\xC0"              // XOR EAX, EAX (hex: 33C0)
    "\x50"                  // PUSH EAX (hex: 50) (NULL: 0x00)
    "\x50"                  // PUSH EAX (hex: 50) (NULL: 0x00)
    "\x53"                  // PUSH EBX (hex: 53) (pointer to "cmd")
    "\x51"                  // PUSH ECX (hex: 51) (pointer to "open")
    "\x50"                  // PUSH EAX (hex: 50) (NULL: 0x00)
    "\xB8\x70\xD9\x2B\x76"  // MOV EAX, 0x762BD970 (hex: B8 70 D9 2B 76)
    "\xFF\xD0";             // CALL EAX (hex: FF D0)
```

### Testing the Shellcode
- Use a simple C program to test the shellcode:
```c
#include <stdio.h>

int main() {
    char code[] =
        "\x68\x00\x64\x6d\x63"  // PUSH (0x68) "cmd\0" (hex: 636d64\x00) in reverse order
        "\x8B\xDC"              // MOV EBX, ESP (0x8B) (hex: DC)
        "\x6A\x00"              // PUSH 0 (hex: 6A) (NULL terminator)
        "\x68\x6e\x65\x70\x6f"  // PUSH (0x68) "open" (hex: 6f70656e) in reverse order
        "\x8B\xCC"              // MOV ECX, ESP (0x8B) (hex: CC)
        "\x6A\x03"              // PUSH 3 (hex: 6A) (3: 0x03)
        "\x33\xC0"              // XOR EAX, EAX (hex: 33C0)
        "\x50"                  // PUSH EAX (hex: 50) (NULL: 0x00)
        "\x50"                  // PUSH EAX (hex: 50) (NULL: 0x00)
        "\x53"                  // PUSH EBX (hex: 53) (pointer to "cmd")
        "\x51"                  // PUSH ECX (hex: 51) (pointer to "open")
        "\x50"                  // PUSH EAX (hex: 50) (NULL: 0x00)
        "\xB8\x70\xD9\x2B\x76"  // MOV EAX, 0x762BD970 (hex: B8 70 D9 2B 76)
        "\xFF\xD0";             // CALL EAX (hex: FF D0)

    int (*func)();
    func = (int (*)()) code;
    (int)(*func)();

    return 0;
}

```

### Detailed Explanations:
- **Null Terminator (`\x00`)**: Indicates the end of a string in memory, preventing overflow.
- **Memory Blocks**: `PUSH` instruction works with 4-byte blocks. If the string fits perfectly (like `"open"`), no extra null byte is needed.
- **MOV Instruction**: Moves data from one location to another, used to store pointers to strings.
- **CALL Instruction**: Executes the function at the specified memory address, effectively running the shellcode.

### Little Endian vs. LIFO

- **Little Endian**:
  - In little endian systems, bytes are stored with the least significant byte first. For example, the string "cmd\0" (hex `636d6400`) is stored in memory as `\x00\x64\x6d\x63`.
  - This byte order is determined by the system architecture.

- **LIFO (Last In, First Out)**:
  - The stack operates on a LIFO basis, meaning the last item pushed is the first to be popped. This affects the sequence of execution but does not change the byte order.

### PUSH Instruction
- The `PUSH` instruction adds data to the stack in the order it is given. The bytes are stored in little endian format because of the system architecture, not because of the `PUSH` operation itself.

---
