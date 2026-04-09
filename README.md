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
```
HTB-writeups/
├── Starting-Point/
│    ├── Tier-0/        # Completed
│    │    ├── Dancing/
│    │    ├── Explosion/
│    │    ├── Fawn/
│    │    ├── Meow/
│    │    ├── Mongod/
│    │    ├── Preignition/
│    │    ├── Redeemer/
│    │    └── Synced/
│    │
│    ├── Tier-1/        # Completed
│    │    ├── Appointment/
│    │    ├── Bike/
│    │    ├── Crocodile/
│    │    ├── Funnel/
│    │    ├── Ignition/
│    │    ├── Pennyworth/
│    │    ├── Responder/
│    │    ├── Sequel/
│    │    ├── Tactics/
│    │    └── Three/
│    │
│    └── Tier-2/        # In progress
│         ├── Archetype/
│         ├── Oopsie/
│         ├── Unified/
│         └── Vaccine/
│
├── Machines/
│    ├── Easy/          # Not started
│    ├── Medium/        # Not started
│    └── Hard/          # Not started
│
└── README.md
```

---

## Current Machines

### Starting Point - Tier 0
| Machine     | Service | Key Issue              | Writeup                                          |
|------------|--------|------------------------|--------------------------------------------------|
| Dancing    | SMB     | Null session           | ./Starting-Point/Tier-0/Dancing/writeup.md       |
| Explosion  | RDP     | Weak authentication    | ./Starting-Point/Tier-0/Explosion/writeup.md     |
| Fawn       | FTP     | Anonymous login        | ./Starting-Point/Tier-0/Fawn/writeup.md          |
| Meow       | Telnet  | No authentication      | ./Starting-Point/Tier-0/Meow/writeup.md          |
| Mongod     | MongoDB | No authentication      | ./Starting-Point/Tier-0/Mongod/writeup.md        |
| Preignition| HTTP    | Broken authentication  | ./Starting-Point/Tier-0/Preignition/writeup.md   |
| Redeemer   | Redis   | Unauthenticated access | ./Starting-Point/Tier-0/Redeemer/writeup.md      |
| Synced     | Rsync   | Anonymous access       | ./Starting-Point/Tier-0/Synced/writeup.md        |

---

### Starting Point - Tier 1
| Machine     | Service              | Key Issue                        | Writeup                                           |
|------------|---------------------|----------------------------------|---------------------------------------------------|
| Appointment| HTTP                 | SQL Injection                    | ./Starting-Point/Tier-1/Appointment/writeup.md    |
| Bike       | HTTP + Node.js       | SSTI (Handlebars)                | ./Starting-Point/Tier-1/Bike/writeup.md           |
| Crocodile  | FTP + HTTP           | Credential reuse                 | ./Starting-Point/Tier-1/Crocodile/writeup.md      |
| Funnel     | SSH + PostgreSQL     | Local port forwarding            | ./Starting-Point/Tier-1/Funnel/writeup.md         |
| Ignition   | HTTP                 | Default credentials              | ./Starting-Point/Tier-1/Ignition/writeup.md       |
| Pennyworth | HTTP + Groovy        | Jenkins script console RCE       | ./Starting-Point/Tier-1/Pennyworth/writeup.md     |
| Responder  | HTTP + NTLM          | NTLM hash capture                | ./Starting-Point/Tier-1/Responder/writeup.md      |
| Sequel     | MySQL                | Unauthenticated DB access        | ./Starting-Point/Tier-1/Sequel/writeup.md         |
| Tactics    | SMB                  | Pass-the-hash, Impacket          | ./Starting-Point/Tier-1/Tactics/writeup.md        |
| Three      | HTTP + S3            | S3 bucket misconfiguration       | ./Starting-Point/Tier-1/Three/writeup.md          |

---

### Starting Point - Tier 2
| Machine   | Service                | Key Issue                             | Writeup                                        |
|----------|------------------------|---------------------------------------|------------------------------------------------|
| Archetype | SMB + MSSQL            | xp_cmdshell, PATH hijacking           | ./Starting-Point/Tier-2/Archetype/writeup.md   |
| Oopsie    | HTTP                   | IDOR, broken access control, webshell | ./Starting-Point/Tier-2/Oopsie/writeup.md      |
| Unified   | UniFi + Log4Shell      | CVE-2021-44228, MongoDB auth          | ./Starting-Point/Tier-2/Unified/writeup.md     |
| Vaccine   | FTP + HTTP + PostgreSQL| SQL injection, webshell, SUID         | ./Starting-Point/Tier-2/Vaccine/writeup.md     |

---

## Skills
- Network scanning (nmap)
- Service enumeration
- Protocol analysis (FTP, SMB, Telnet, Redis, MongoDB, Rsync, HTTP, MySQL, PostgreSQL, MSSQL)
- Authentication bypass techniques
- Misconfiguration exploitation
- SQL injection
- Server-Side Template Injection (SSTI)
- NTLM hash capture and relay
- File upload and webshell deployment
- IDOR and broken access control
- Log4Shell (CVE-2021-44228) exploitation
- JNDI injection via LDAP
- Local port forwarding (SSH tunneling)
- PATH hijacking
- SUID privilege escalation
- MongoDB credential manipulation
- Jenkins Groovy script console RCE
- Basic post-exploitation

---

## Methodology
1. Identify open ports and services
2. Analyze service behavior and weaknesses
3. Select the most probable attack path
4. Exploit misconfiguration or vulnerability
5. Verify access through flag retrieval
6. Identify root cause

---

## Purpose
- Build a solid foundation in penetration testing
- Understand real-world misconfigurations and vulnerabilities
- Develop structured attack reasoning
- Improve technical documentation

---

## Notes
- All machines are from Hack The Box
- This repository is for educational purposes only
