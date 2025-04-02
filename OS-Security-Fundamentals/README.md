# 🔐 OS-Security-Fundamentals

This folder is a **organization space for understanding the internal structure of the operating system from a security perspective**.
It is not simply a review of OS concepts, but rather an analysis and application of the infrastructure required for implementing a security system.

---

## 🎯 Purpose

- Reconstruct the core components of the operating system from a **system security perspective**
- **Analysis of the causes of security vulnerabilities such as kernel, memory, and system calls**
- Securing a foundation for future system security paper reviews and practice

---

## 📁 Directory Structure

| Folder                  | Description                                                           |
| ----------------------- | --------------------------------------------------------------------- |
| `Kernel-Memory/`        | Kernel address space, system call interface, memory isolation         |
| `Privilege-Escalation/` | Analysis of privilege escalation vulnerability cases and principles   |
| `Access-Control/`       | Analysis of Linux DAC, MAC, Capability models                         |
| `Exploit-Primitives/`   | Buffer Overflow, Use-after-Free, etc. Security Vulnerabilities Basics |
| `Syscall-Audit/`        | System call tracing and security monitoring techniques                |

> Each directory consists of concept organization + example code + image-based learning flow.

---

## 🧩 Recommended learning flow

1. `Kernel-Memory` → Get a feel for the system structure
2. `Privilege-Escalation` → Understand system privilege flow and vulnerabilities
3. `Access-Control` → Interpret Linux-based access control
4. `Exploit-Primitives` → Learn major attack techniques
5. `Syscall-Audit` → Security monitoring and policy application

---

## 🔄 Future additional plans

- Actual vulnerability analysis (e.g. Dirty COW, Ret2usr)
- QEMU-based kernel debugging setup tutorial
- OS customization project with security policy applied
