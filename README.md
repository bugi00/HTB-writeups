# HTB Writeups

Hack The Box 머신 라이트업 모음. **재현보다 사고 과정**에 초점을 둔다 — 어떤 명령을 쓰는지가 아니라 왜 그 판단을 내렸는지를 기록한다.

각 라이트업은 다음 구조를 따른다: 열거 → 분석 → 익스플로잇 → 플래그 획득 → 근본 원인.

Starting Point와 retired된 Easy·Medium 라이트업을 공개한다. **아직 retire되지 않았거나 공개하면 안 되는 머신의 상세 풀이는 개인 Notion에서 관리**하며 유출 방지를 위해 저장소에 포함하지 않는다. 아래 비공개 목록에는 이름, OS, 난이도만 표기하고 상세 내용은 요청 시 공유한다.

---

## 저장소 구성

```
HTB-writeups/
├── Starting-Point/
│    ├── Tier-0/
│    ├── Tier-1/
│    └── Tier-2/
├── Easy/
├── Medium/
└── README.md
```

---

## 진행 현황

| 난이도          | 완료(solved) | 공개 라이트업(이 저장소) | 비공개(Notion) |
|-----------------|--------------|--------------------------|----------------|
| Starting Point  | 25           | 25                       | —              |
| Easy            | 8            | 6                        | 2              |
| Medium          | 6            | 2                        | 4              |
| Hard            | 2            | —                        | 2              |
| **합계**        | **41**       | **33**                   | **8**          |

---

## 공개 라이트업 — Retired (Easy / Medium)

| 머신        | 난이도 | OS      | 핵심 기법                                                        | 라이트업                          |
|------------|--------|---------|------------------------------------------------------------------|----------------------------------|
| CCTV        | Easy   | Linux   | ZoneMinder, SQLi (CVE-2024-51482), capability 악용, RCE (CVE-2025-60787) | [writeup](./Easy/CCTV/writeup.md) |
| Facts       | Easy   | Linux   | 웹 열거, AWS S3 설정 오류                                         | [writeup](./Easy/Facts/writeup.md)       |
| Forest      | Easy   | Windows | AD 열거, AS-REP roasting, DCSync                                  | [writeup](./Easy/Forest/writeup.md)      |
| Nocturnal   | Easy   | Linux   | IDOR, ISPConfig RCE (CVE-2023-46818)                             | [writeup](./Easy/Nocturnal/writeup.md)   |
| Planning    | Easy   | Linux   | Grafana RCE (CVE-2024-9264), cron 권한 상승                       | [writeup](./Easy/Planning/writeup.md)    |
| Support     | Easy   | Windows | LDAP 자격증명 추출, RBCD                                         | [writeup](./Easy/Support/writeup.md)     |
| Environment | Medium | Linux   | Laravel 익스플로잇, sudo BASH_ENV 권한 상승                       | [writeup](./Medium/Environment/writeup.md) |
| Voleur      | Medium | Windows | AD Kerberos 악용, DPAPI, Backup Operators                        | [writeup](./Medium/Voleur/writeup.md)    |

---

## 공개 라이트업 — Starting Point

### Tier 0

| 머신        | 서비스   | 핵심 이슈              | 라이트업                                          |
|------------|----------|------------------------|--------------------------------------------------|
| Dancing     | SMB      | Null session           | [writeup](./Starting-Point/Tier-0/Dancing/writeup.md)       |
| Explosion   | RDP      | 취약한 인증            | [writeup](./Starting-Point/Tier-0/Explosion/writeup.md)     |
| Fawn        | FTP      | 익명 로그인            | [writeup](./Starting-Point/Tier-0/Fawn/writeup.md)          |
| Meow        | Telnet   | 인증 없음              | [writeup](./Starting-Point/Tier-0/Meow/writeup.md)          |
| Mongod      | MongoDB  | 인증 없음              | [writeup](./Starting-Point/Tier-0/Mongod/writeup.md)        |
| Preignition | HTTP     | 인증 우회              | [writeup](./Starting-Point/Tier-0/Preignition/writeup.md)   |
| Redeemer    | Redis    | 미인증 접근            | [writeup](./Starting-Point/Tier-0/Redeemer/writeup.md)      |
| Synced      | Rsync    | 익명 접근              | [writeup](./Starting-Point/Tier-0/Synced/writeup.md)        |

### Tier 1

| 머신        | 서비스           | 핵심 이슈                  | 라이트업                                           |
|------------|------------------|----------------------------|---------------------------------------------------|
| Appointment | HTTP             | SQL Injection              | [writeup](./Starting-Point/Tier-1/Appointment/writeup.md)    |
| Bike        | HTTP + Node.js   | SSTI (Handlebars)          | [writeup](./Starting-Point/Tier-1/Bike/writeup.md)           |
| Crocodile   | FTP + HTTP       | 자격증명 재사용            | [writeup](./Starting-Point/Tier-1/Crocodile/writeup.md)      |
| Funnel      | SSH + PostgreSQL | 로컬 포트 포워딩           | [writeup](./Starting-Point/Tier-1/Funnel/writeup.md)         |
| Ignition    | HTTP             | 기본 자격증명              | [writeup](./Starting-Point/Tier-1/Ignition/writeup.md)       |
| Pennyworth  | HTTP + Groovy    | Jenkins 스크립트 콘솔 RCE  | [writeup](./Starting-Point/Tier-1/Pennyworth/writeup.md)     |
| Responder   | HTTP + NTLM      | NTLM 해시 캡처             | [writeup](./Starting-Point/Tier-1/Responder/writeup.md)      |
| Sequel      | MySQL            | 미인증 DB 접근             | [writeup](./Starting-Point/Tier-1/Sequel/writeup.md)         |
| Tactics     | SMB              | Pass-the-hash, Impacket    | [writeup](./Starting-Point/Tier-1/Tactics/writeup.md)        |
| Three       | HTTP + S3        | S3 버킷 설정 오류          | [writeup](./Starting-Point/Tier-1/Three/writeup.md)          |

### Tier 2

| 머신      | 서비스                  | 핵심 이슈                                                              | 라이트업                                        |
|----------|-------------------------|-----------------------------------------------------------------------|------------------------------------------------|
| Archetype | SMB + MSSQL             | xp_cmdshell, PATH 하이재킹                                            | [writeup](./Starting-Point/Tier-2/Archetype/writeup.md)   |
| Base      | HTTP                    | Vim 스왑 노출, strcmp() 타입 혼동, 파일 업로드 RCE, sudo 설정 오류      | [writeup](./Starting-Point/Tier-2/Base/writeup.md)        |
| Included  | HTTP + TFTP             | LFI, TFTP 업로드, lxd 그룹 권한 상승                                   | [writeup](./Starting-Point/Tier-2/Included/writeup.md)    |
| Markup    | HTTP + SSH              | XXE 인젝션, SSH 키 노출, 파일 권한 설정 오류                           | [writeup](./Starting-Point/Tier-2/Markup/writeup.md)      |
| Oopsie    | HTTP                    | IDOR, 접근 제어 미흡, 웹셸                                            | [writeup](./Starting-Point/Tier-2/Oopsie/writeup.md)      |
| Unified   | UniFi + Log4Shell       | CVE-2021-44228, MongoDB 인증                                          | [writeup](./Starting-Point/Tier-2/Unified/writeup.md)     |
| Vaccine   | FTP + HTTP + PostgreSQL | SQL 인젝션, 웹셸, SUID                                                | [writeup](./Starting-Point/Tier-2/Vaccine/writeup.md)     |

---

## 풀이 완료 — 비공개 (Notion)

아직 retire되지 않았거나 공개하면 안 되는 머신의 상세 풀이는 비공개로 유지한다. 기법은 Notion에 정리되어 있으며 요청 시 공유 가능하다.

| 머신       | 난이도 | OS      | 주제                              |
|-----------|--------|---------|-----------------------------------|
| Silentium  | Easy   | Linux   | Flowise RCE, 컨테이너 탈출, Gogs RCE |
| Reactor    | Easy   | Linux   | 웹 익스플로잇, Node.js inspector  |
| DevHub     | Medium | Linux   | MCP 서버 익스플로잇, hidden admin API |
| Helix      | Medium | Linux   | Apache NiFi RCE, OPC-UA           |
| Logging    | Medium | Windows | AD, ADCS, WSUS 스푸핑             |
| Checkpoint | Medium | Windows | AD, BadSuccessor, Volatility      |
| Pirate     | Hard   | Windows | AD 위임 체인, gMSA, SPN 하이재킹  |
| Nimbus     | Hard   | Linux   | 클라우드/AWS, SSRF, IAM 악용      |

---

## 보유 역량

**네트워크 / 서비스 열거**
- nmap, 서비스 핑거프린팅, vhost/서브도메인 퍼징(ffuf, gobuster, feroxbuster)
- 프로토콜 분석: FTP, SMB, Telnet, Redis, MongoDB, Rsync, HTTP, MySQL, PostgreSQL, MSSQL, LDAP, Kerberos

**웹 익스플로잇**
- SQL 인젝션, SSTI, XXE, IDOR / 접근 제어 우회
- 파일 업로드 및 웹셸 배포, PHP 타입 혼동(strcmp 우회)
- SSRF, LFI, 소스코드 내 자격증명 추출

**액티브 디렉터리(AD)**
- AS-REP roasting, Kerberoasting, DCSync
- Pass-the-hash, NTLM 캡처 및 릴레이
- 제약 위임 / 리소스 기반 제약 위임(RBCD), SPN 조작
- BloodHound 기반 공격 경로 분석

**권한 상승**
- SUID / sudo 설정 오류, PATH 하이재킹, capability 악용, lxd 그룹 악용
- 로컬 포트 포워딩 및 피벗(SSH 터널링, ligolo-ng)
- 컨테이너 탈출, cron / 예약 작업 악용

**주요 CVE / 기법**
- Log4Shell (CVE-2021-44228), LDAP를 통한 JNDI 인젝션
- ISPConfig RCE (CVE-2023-46818), Grafana RCE (CVE-2024-9264)
- Jenkins Groovy 스크립트 콘솔 RCE, Vim 스왑 파일 분석

---

## 방법론

1. 열린 포트와 서비스 식별
2. 서비스 동작과 약점 분석
3. 가장 가능성 높은 공격 경로 선택
4. 설정 오류 또는 취약점 익스플로잇
5. 플래그 획득으로 접근 검증
6. 근본 원인 식별

---

## 목적

- 침투 테스트의 탄탄한 기초 확립
- 실제 환경의 설정 오류와 취약점 이해
- 구조적인 공격 사고력 개발
- 기술 문서화 역량 향상

---

## 비고

- 모든 머신은 Hack The Box에서 제공된다.
- 공개 라이트업은 Starting Point 및 retired 머신에 한정한다. active 머신 풀이는 HTB 정책에 따라 Notion에서 비공개로 관리한다.
- 교육 목적으로만 사용한다.
