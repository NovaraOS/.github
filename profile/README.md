# NovaraOS

**A Research Architecture for a Verified Consumer Desktop**

> **⚠️ STATUS: CONCEPT / ARCHITECTURE PROPOSAL**
>
> **This is a theoretical framework.** There is currently **no functioning operating system**, ISO, or active repository.
> This document outlines a research proposal to combine the **seL4 microkernel** with a **native NT kernel personality** to create a crash-resistant, "console-grade" PC experience.

---

## 1. Core Philosophy: "The Architecture of Trust"

Modern OS security relies on "Access Control Lists" (asking a software guard for permission). novaraOS proposes a **Capability-Based Security Model** (having a physical key).

* **The Difference:** In standard OSs, root can access everything. In novaraOS, if an app lacks the specific capability key for a memory address, it is **mathematically impossible** for it to access it.
* **The Goal:** To replace "Anti-Cheat Scanning" (invasive surveillance) with "Platform Attestation" (mathematical proof of environment integrity).

---

## 2. The "Dual-Track" Integrity System

To solve the conflict between "Open Source Freedom" and "Competitive Anti-Cheat," novaraOS proposes two distinct boot modes.

| Feature | **Open Mode (Dev / Kernel Hacking)** | **Verified Mode (Standard / Competitive)** |
| :--- | :--- | :--- |
| **Boot Chain** | Unlocked / Self-Signed | **Secure Boot + TPM 2.0** |
| **Kernel State** | Modifiable (Root Access) | **Immutable / Signed** |
| **Target Use** | Kernel Dev, OS Modding | **Gaming, Modding, Emulation, Banking** |
| **App Support** | Standard Apps, Mods, Emulators | **All Above + Apps Requiring "Verified OS"** |
| **Anti-Cheat** | Flagged as "Untrusted" | **Flagged as "Verified Platform"** |

* **Clarification:** **Verified Mode** does *not* lock down user-space. You can still install mods (e.g., Assetto Corsa Content Manager), run emulators, and use ReShade. The restriction only prevents modification of the **Kernel** and **OS Core**, which satisfies Anti-Cheat requirements without blocking game mods.

---

## 3. Technical Architecture

### The Foundation: seL4 & User-Space Drivers

* **Kernel:** **seL4**. The only general-purpose OS kernel with a formal mathematical proof of implementation correctness.
* **Fault Isolation:** All drivers (GPU, Network, Audio) run in **User Space**.
    * *Concept:* If a graphics driver crashes, the OS detects the fault and restarts the driver process in milliseconds. The system does not panic; the screen simply flickers.

### The Compatibility Layer (Native NT Personality)
* **Pure NT Implementation:** We do not use Wine, Proton, or Linux translation layers.
* **Architecture:** The OS implements the **Windows NT Executive** (NTOSKRNL) directly in user space on top of seL4 (based on the NeptuneOS architecture).
* **Goal:** To run Windows binaries (`.exe`) natively by servicing their NT syscalls directly, without the overhead or translation layers of a POSIX host.

---

## 4. Engineering Challenges

This concept faces significant hurdles that currently prevent it from being a reality. This project aims to document and research solutions for these specific problems:

### 1. The "NT Personality" Difficulty
Replicating the Windows NT Executive (the kernel-level API) in user space is a monumental task.
* **The Problem:** Most Windows applications rely on undocumented or complex NT system calls.
* **Current Reality:** The upstream NeptuneOS project is in early stages. It cannot currently run complex GUI applications or modern games.

### 2. The Driver Gap
Microkernels do not have the vast hardware support of the Linux kernel.
* **Proposed Solution:** Utilization of the Genode **DDE (Device Driver Environment)** to wrap Linux drivers.
* **Limitation:** This introduces latency and does not cover proprietary hardware (e.g., specific RGB controllers, obscure sound cards).

### 3. Gaming Performance
* **The Problem:** Translating DirectX calls → NT Personality → Genode → Hardware introduces too much latency for AAA gaming.
* **Research Area:** Exploring "Hybrid Virtualization" where the OS runs securely, but games run in a highly optimized, sandboxed VM with GPU passthrough.

---

## 5. User Experience (UX) Architecture

### "Just-in-Time" Permissions
Inspired by mobile OS security but adapted for desktop workflows.
* **Default State:** Deny All.
* **The Prompt:** When an app attempts to access a resource (Microphone, File System, Game Memory) for the first time, the kernel halts execution and prompts the user via a **System Pop-up**.
* **The Enforcement:** If denied, the kernel feeds "Null Data" to the application. The app cannot bypass this as it lacks the memory capability to read the real hardware.

### The Unified Settings App
A single system application managing all configurations, eliminating the need to edit text files manually.
* **Layer 1 (Normal Mode):** Simple GUI toggles (Wi-Fi, Resolution, Dark Mode).
* **Layer 2 (Expert Mode):** Granular control (Driver rollback, Registry tweaks).
* **Layer 3 (Declarative Mode):** The GUI generates a **NixOS-style configuration file**. Users can export this file to replicate their exact OS state on another machine instantly.

---

## 6. Installation Strategy

* **Standard Install:** Deploys pre-compiled binaries for immediate use (~5 min).
* **Optimized Install:** A source-based installation that compiles the OS and subsystems specifically for the host CPU architecture (e.g., `-march=native`), theoretically maximizing performance (~45 min).

---

## 📄 License & Status

* **Project Status:** **Pre-Alpha / Conceptual Research.**
* **Intended License:** GPLv2 (Kernel/Drivers) / MIT (UI Components).
* **Contributions:** Currently seeking theoretical discussions on **seL4 driver interfaces** and **TPM 2.0 attestation workflows**.

---

## 🙏 Credits & Acknowledgments

This research concept stands on the shoulders of giants. We acknowledge the foundational work of:

* **[seL4 Foundation](https://sel4.systems/)**: For the world's only formally verified microkernel.
* **[Genode Labs](https://genode.org/)**: For the robust OS framework and driver environments.
* **[NeptuneOS Team](https://github.com/cl91/NeptuneOS)**: For the pioneering work on implementing the Windows NT personality on microkernels.
* **[Qt Project](https://www.qt.io/)**: For the user interface technology.
