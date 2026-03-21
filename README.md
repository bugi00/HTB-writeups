# HTB Writeups

This repository contains writeups for Hack The Box machines.

It includes Starting Point machines and will be expanded to higher difficulty levels.

---

## Project Overview

The goal of this repository is to understand vulnerabilities and attack paths, not just reproduce solutions.

Each writeup focuses on reasoning and follows a structured approach:

* Enumeration
* Analysis
* Exploitation
* Flag Retrieval
* Root Cause

---

## Repository Structure

HTB-writeups/
├── Starting-Point/
│    ├── Meow/
│    ├── Fawn/
│    ├── Dancing/
│    ├── Explosion/
│    ├── Preignition/
│    ├── Mongod/
│    ├── Synced/
│    └── Redeemer/
│
├── Machines/
│    ├── Easy/
│    ├── Medium/
│    └── Hard/
│
└── README.md

---

## Current Machines

### Starting Point (Tier 0)

| Machine     | Service | Key Issue              | Writeup                                 |
| ----------- | ------- | ---------------------- | --------------------------------------- |
| Meow        | Telnet  | No authentication      | ./Starting-Point/Meow/writeup.md        |
| Fawn        | FTP     | Anonymous login        | ./Starting-Point/Fawn/writeup.md        |
| Dancing     | SMB     | Null session           | ./Starting-Point/Dancing/writeup.md     |
| Explosion   | RDP     | Weak authentication    | ./Starting-Point/Explosion/writeup.md   |
| Preignition | HTTP    | Broken authentication  | ./Starting-Point/Preignition/writeup.md |
| Mongod      | MongoDB | No authentication      | ./Starting-Point/Mongod/writeup.md      |
| Synced      | Rsync   | Anonymous access       | ./Starting-Point/Synced/writeup.md      |
| Redeemer    | Redis   | Unauthenticated access | ./Starting-Point/Redeemer/writeup.md    |

---

## Skills

* Network scanning (nmap)
* Service enumeration
* Protocol analysis (FTP, SMB, Telnet, Redis, MongoDB, Rsync)
* Authentication bypass techniques
* Misconfiguration exploitation
* Basic post-exploitation

---

## Methodology

1. Identify open ports and services
2. Analyze service behavior and weaknesses
3. Select the most probable attack path
4. Exploit misconfiguration or weak authentication
5. Verify access through flag retrieval
6. Identify root cause

---

## Purpose

* Build a solid foundation in penetration testing
* Understand real-world misconfigurations
* Develop structured attack reasoning
* Improve technical documentation

---

## Notes

* All machines are from Hack The Box
* This repository is for educational purposes only
