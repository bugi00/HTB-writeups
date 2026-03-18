# HTB Writeups - BUGI00

## About

This repository documents my journey through Hack The Box Labs.

Rather than simply solving machines, the focus is on understanding:

* Why a specific enumeration step is performed
* How exposed services lead to attack opportunities
* How misconfigurations can be leveraged for access

All writeups are written in Korean and emphasize reproducibility and reasoning.

---

##  Objective

* Build a solid foundation in penetration testing
* Develop an enumeration-first mindset
* Understand core network services (Telnet, FTP, SMB)
* Identify and exploit common misconfigurations

---

##  Structure

```
HTB-writeups/
 ├── README.md
 ├── Meow/
 │   └── writeup.md
 ├── Fawn/
 │   └── writeup.md
 ├── Dancing/
 │   └── writeup.md
```

---

## Tools & Reasoning

* nmap
  → Identify open ports and running services (starting point of all attacks)

* telnet / ftp / smbclient
  → Direct interaction with services to validate authentication and access control weaknesses

These tools are used not just for execution, but for verifying assumptions during enumeration.

---

## Methodology

All machines follow a consistent approach:

1. Enumeration (nmap, service detection)
2. Service Analysis (protocol behavior understanding)
3. Access Attempt (anonymous / weak authentication)
4. Exploitation (misconfiguration abuse)
5. Flag Retrieval

---

## Notes

* Flags are intentionally excluded
* Focus is on process, not answers
* Each step includes reasoning for reproducibility

---

## Root Cause Summary

* Meow → Telnet allows plaintext authentication
* Fawn → FTP allows anonymous login
* Dancing → SMB allows null session access

---

## Next Learning Path

* Redeemer → Database interaction (Redis)
* Explosion → Windows authentication (RDP)
* Preignition → Web enumeration basics

The goal is to gradually move from simple misconfigurations to multi-step exploitation.

---

## Progress

| Machine  | Difficulty | Status |
| -------- | ---------- | ------ |
| Meow     | Very Easy  | ✅      |
| Fawn     | Very Easy  | ✅      |
| Dancing  | Very Easy  | ✅      |
| Redeemer | Very Easy  | ⏳      |

---
