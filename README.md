# AZOTH-VM │ Polymorphic Forth Runtime

"Code that treats memory as a fluid medium. Runtime adaptability over static compilation."

## ⚗️ System Overview

AZOTH-VM is a minimal, stack-based Virtual Machine implementing a dialect of Forth. It is designed as a lightweight scripting layer capable of running within constrained environments (microcontrollers) or as an embedded logic engine within larger applications.

Unlike static languages (C/Rust) where the execution path is frozen at compile time, AZOTH maintains the compiler, interpreter, and dictionary in memory simultaneously. This allows for Reflexive Metaprogramming: the system can inspect its own code structure, redefine execution words at runtime, and adapt its behavior without a system reboot.

### Core Objectives

- Plasticity: Enable hot-swapping of logic in mission-critical loops (Live Coding).

- Instrumentation: Provide a REPL (Read-Eval-Print Loop) interface for deep hardware inspection and debugging.

- Polymorphism: Support self-modifying code techniques to alter execution signatures dynamically.

## 🏗️ Architecture

The VM is built upon the Indirect Threaded Code (ITC) model to maximize flexibility.

### 1. The Dictionary (Linked List)

- Structure: Code is organized as a linked list of "Words" (headers).

- Dynamic Search: Execution looks up words in the dictionary at runtime.

- Polymorphism: By manipulating the linked list pointers, words can be "shadowed" or redefined instantly. The system can switch between different implementations of a function (e.g., LOG_DATA) by simply pointing the dictionary head to a new memory address.

### 2. The Stacks

- Data Stack: Holds operands for computation.

- Return Stack: Holds execution pointers (call stack).

- Isolation: Stack overflows are contained within the VM, preventing host system crashes.

### 3. The Core (C Host)

- Portable: The inner interpreter is written in ANSI C89 for maximum portability across architectures (AVR, ARM, x86).

- FFI (Foreign Function Interface): Exposed hooks allow the Forth environment to call C functions in the host firmware (e.g., toggling GPIO, reading sensors).

---

## 🛠️ Tech Stack

- Language: ANSI C (VM Core) + Forth (Scripting).

- Memory Footprint: < 16KB Flash, < 2KB RAM (Configurable).

- Interface: Serial UART / Standard I/O.

---

## 📦 Usage Example

#### 1. Hot-Swapping Logic

Defining a function, running it, and then redefining it on the fly.
~~~
\ Define a word to read sensor data
: MONITOR-STATUS ( -- ) 
  READ-SENSOR 
  10 > IF ." WARNING: High Value" CR ELSE ." OK" CR THEN ;

\ Run it
MONITOR-STATUS  \ Output: OK

\ Redefine it dynamically (Polymorphism)
\ The system now uses this new definition for all future calls
: MONITOR-STATUS ( -- ) 
  READ-SENSOR 
  ." Raw Value: " . CR ;

MONITOR-STATUS  \ Output: Raw Value: 5
~~~

#### 2. Metaprogramming (Code creating Code)

Creating a lookup table generator.
~~~
: GENERATE-TABLE ( n -- )
  0 DO 
    I I * ,  \ Compile square of I into memory
  LOOP ;

CREATE SQUARES 10 GENERATE-TABLE
~~~

## 🔒 Security & Constraints

- Direct Memory Access: AZOTH provides PEEK and POKE capabilities. While powerful for debugging, this requires strict isolation when running alongside privileged kernels.

- Sandboxing: Ideally deployed within a secure container or strictly defined memory segment when used in networked environments.

## ⚖️ License

### MIT License

Permission is hereby granted to modify and distribute this runtime, provided the core copyright notices remain intact.
