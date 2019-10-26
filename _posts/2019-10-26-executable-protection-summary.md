---
layout: post
title: Executable Protection - Summary
author: Xiang Ye
description: The way how to protect executable
categories: [Reverse Engineering]
---

```
NOTE: This is not a tutorial -- it is a memo. I won't list detail implementation of the techniques used in these articles, instead, I want to record why I choose those techniques. Also, I won't repeat those basic concepts -- I assume you guys already knows those.
```

- [1. Motivations](#1-motivations)
    - [1.1 A Real Case](#11-a-real-case)
- [2. What To Protect Against?](#2-what-to-protect-against)
    - [2.1 Static Analysis](#21-static-analysis)
        - [2.1.1 Tools](#211-tools)
        - [2.1.2 Code Disassembling](#212-code-disassembling)
        - [2.1.3 Readonly Data Analysis](#213-readonly-data-analysis)
        - [2.1.4 Import Functions Analysis](#214-import-functions-analysis)
        - [2.1.5 Export Functions Analysis](#215-export-functions-analysis)
    - [2.2 Debugging](#22-debugging)
        - [2.2.1 Tools](#221-tools)
    - [2.3 Patch](#23-patch)
- [3. Challenges](#3-challenges)
- [4. Articles](#4-articles)


### 1. Motivations

There are many motivations to crack an executable. For example, to pirate the software, to steal data from an application, to monitor user/application's behavior or to provide fake data via the application. That's why executable protection becomes important (although many software companies don't realize this).

#### 1.1 A Real Case

I guess game companies can feel the pain most. There are so many exploits and bots created for games. Although game companies work hard and take many approaches to prevent exploits and bots, the exploit's authors can always find new ways to cheat. They put game executable under the microscope and cut it into pieces to figure out what it does, what it uses or what it sends to the server. After they understand how the game executable works, they can either patch the executable to make it works as they expected or they build a fake game client working as bots.

### 2. What To Protect Against?

There are many methods can be used to crack executable, Here I only discuss those work on executable directly. Others (like network traffic monitoring, etc.) are not in scope.

#### 2.1 Static Analysis

Static analysis might be the second action that should be taken when cracking an executable (the first action is to play the executable to get familiar with it). It is a very important action because it can give crackers most of the useful information about the executable. So the protection needs to find ways to hide information from static analysis.

##### 2.1.1 *Tools*
- Executable Viewer (Like PE Studio, Depends, CFF Explorer, etc.)
- Disassembler (IDA Pro)
- Hex Editor (Ultra Editor, 101 Editor, etc.)

##### 2.1.2 *Code Disassembling*

Disassembled code helps hackers to understand how the executable works. From the code, they can figure out how executable encrypt its data, how executable verify user credential, how executable verify the serial number, etc.

##### 2.1.3 *Readonly Data Analysis*

Readonly data (especially strings) can provide really useful information which might surprise you. When I reverse engineering an executable, I always go over strings, find those with keywords, check their `x-reference` to see which functions use them, and then I can focus on those key functions. Sometimes, those well-defined strings are like lightening at dark night. Another example is the prime number or default keys used by crypto library -- there are tools to search those data in executable to figure out what kind of crypto algorithms are used.

##### 2.1.4 *Import Functions Analysis*

When an important API is in the import table, hackers can easily know how it is used. For example, when `CryptCreateHash` is used, you know immediately that WinCrypt APIs are used to generate the hash. By tacking the function calls this API, you should be able to find why a hash is generated and how it is used (maybe for password?).

##### 2.1.5 *Export Functions Analysis*

This is useful when analyzing a dynamic library. A meaningful function can tell hackers a lot of things and make analysis easier.

#### 2.2 Debugging

Static analysis cannot provide al the information (due to obfuscation, packer, anti-disassemble or some data only exists at run-time). That's why runtime debugging is required to figure out what exactly the executable does. Normally after static analysis, we know several important functions and run the debugger to analyze how those functions work step by step.

##### 2.2.1 *Tools*
- Debugger (WinDbg, x32dbg/x64dbg, etc.)
- Memory Viewer/Editor (Cheat Engine, etc.)
- Useful plug-ins (PE dumper, fixer, etc.)

#### 2.3 Patch

After analyze and understand how executable works, hackers can patch it to change its behavior or data. For example, nop-out security checks, bypassing license check, sending fake data, etc.

### 3. Challenges

There is no silver bullet. When an executable is distributed, end-users own it physically, and they can do whatever they want on the executable. All the protection methods can only make it harder to analyze and crack executables, but cannot prevent it from being analyzed and cracked completely. In theory, any executable can be cracked --- the difference is how much cost is required to do this.

There are several things should be understood:

- Any executable can be analyzed and cracked at different costs.
- The meaning of executable protect is to make analysis and cracking harder.
- Single protection method is always easy to crack -- different methods should be used together in a proper way and in many places.
- Security has a cost, either the performance or the ability of easy-to-maintain.

### 4. Articles

- [Executable Protection - Summary](/executable-protection-summary/)
- [Executable Protection - Anti Disassembling](/executable-protection-anti-disassembling/)
- [Executable Protection - Data Obfuscation](/executable-protection-data-obfuscation/)
- [Executable Protection - Import and Export Functions](/executable-protection-import-and-export-functions/})
