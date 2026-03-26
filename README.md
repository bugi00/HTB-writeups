# HTB Writeups

This repository contains writeups for Hack The Box machines.

It includes Starting Point machines and will be expanded to higher difficulty levels.

---

## Project Overview

The goal of this repository is to understand vulnerabilities and attack paths, not just reproduce solutions.

Each writeup focuses on reasoning and follows a structured approach:

- Enumeration
- Analysis
- Exploitation
- Flag Retrieval
- Root Cause

---

## Repository Structure

## Repository Structure

HTB-writeups/
├── Starting-Point/
│    ├── Tier-0/
│    │    ├── Meow/
│    │    ├── Fawn/
│    │    ├── Dancing/
│    │    ├── Explosion/
│    │    ├── Preignition/
│    │    ├── Mongod/
│    │    ├── Synced/
│    │    └── Redeemer/
│    │
│    ├── Tier-1/
│    │    └── Appointment/
│    │
│    └── Tier-2/
│         └── (in progress)
│
├── Machines/
│    ├── Easy/
│    ├── Medium/
│    └── Hard/
│
└── README.md

---

## Current Machines

### Starting Point - Tier 0

| Machine     | Service | Key Issue              | Writeup                                          |
|------------|--------|------------------------|--------------------------------------------------|
| Meow       | Telnet | No authentication      | ./Starting-Point/Tier-0/Meow/writeup.md          |
| Fawn       | FTP    | Anonymous login        | ./Starting-Point/Tier-0/Fawn/writeup.md          |
| Dancing    | SMB    | Null session           | ./Starting-Point/Tier-0/Dancing/writeup.md       |
| Explosion  | RDP    | Weak authentication    | ./Starting-Point/Tier-0/Explosion/writeup.md     |
| Preignition| HTTP   | Broken authentication  | ./Starting-Point/Tier-0/Preignition/writeup.md   |
| Mongod     | MongoDB| No authentication      | ./Starting-Point/Tier-0/Mongod/writeup.md        |
| Synced     | Rsync  | Anonymous access       | ./Starting-Point/Tier-0/Synced/writeup.md        |
| Redeemer   | Redis  | Unauthenticated access | ./Starting-Point/Tier-0/Redeemer/writeup.md      |

---

### Starting Point - Tier 1

| Machine     | Service | Key Issue     | Writeup                                          |
|------------|--------|--------------|--------------------------------------------------|
| Appointment| HTTP   | SQL Injection| ./Starting-Point/Tier-1/Appointment/writeup.md   |

---

### Starting Point - Tier 2

In progress.

---

## Skills

- Network scanning (nmap)
- Service enumeration
- Protocol analysis (FTP, SMB, Telnet, Redis, MongoDB, Rsync, HTTP)
- Authentication bypass techniques
- Misconfiguration exploitation
- Basic post-exploitation

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

- Build a solid foundation in penetration testing
- Understand real-world misconfigurations
- Develop structured attack reasoning
- Improve technical documentation

---

## Notes

- All machines are from Hack The Box
- This repository is for educational purposes only
