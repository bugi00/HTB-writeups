
# Funnel

**Difficulty:** Very Easy
**OS:** Linux
**Tags:** FTP Anonymous Login, Credential Leak, SSH Port Forwarding, PostgreSQL

---

## Summary

FTP 익명 로그인으로 내부 메일 백업 파일을 획득하고, 거기서 기본 패스워드를 추출해 SSH 접속에 성공한다. 내부에서만 접근 가능한 PostgreSQL 포트를 SSH 로컬 포트 포워딩으로 노출시킨 뒤, 데이터베이스에서 플래그를 획득하는 머신이다.

---

## Enumeration

### Nmap

```bash
nmap -sC -sV $IP
```

![nmap-full-1](./images/nmap-full-1.png)
![nmap-full-2](./images/nmap-full-2.png)

열려 있는 포트는 두 개다.

- **21/tcp** — vsftpd 3.0.3
- **22/tcp** — OpenSSH 8.2p1 (Ubuntu)

FTP가 열려 있고 vsftpd이므로 익명 로그인 가능 여부를 먼저 확인한다.

---

## Foothold

### FTP Anonymous Login

```bash
ftp $IP
# Name: anonymous
# Password: (빈칸 엔터)
```

![ftp-anonymous-login](./images/ftp-anonymous-login.png)

`230 Login successful` 응답으로 익명 로그인이 허용된 상태임을 확인한다.

### FTP 디렉토리 탐색

```bash
ftp> ls
```

![ftp-ls](./images/ftp-ls.png)

루트에 `mail_backup` 디렉토리가 존재한다. 이름만으로도 내부 메일 백업임을 추측할 수 있다.

```bash
ftp> cd mail_backup
ftp> ls
```

![ftp-mail-backup](./images/ftp-mail-backup.png)

두 개의 파일이 있다.

- `password_policy.pdf` — 회사 패스워드 정책 문서
- `welcome_28112022` — 신규 입사자 환영 메일로 추정

### 파일 다운로드

```bash
ftp> get password_policy.pdf
```

![ftp-get-policy](./images/ftp-get-policy.png)

```bash
ftp> get welcome_28112022
```

![ftp-get-welcome](./images/ftp-get-welcome.png)

### 파일 내용 분석

**password_policy.pdf**

![password-policy](./images/password-policy.png)

패스워드 정책 문서 하단에 중요한 내용이 있다.

> "Default passwords — such as those created for new users — must be changed as quickly as possible. For example the default password of **funnel123#!#** must be changed immediately."

기본 패스워드가 문서에 평문으로 노출되어 있다. `funnel123#!#`이 신규 계정의 초기 패스워드임을 알 수 있다.

**welcome_28112022**

![welcome-email](./images/welcome-email.png)

환영 메일의 수신자 목록에서 유저명을 추출할 수 있다.

- optimus
- albert
- andreas
- **christine**
- maria

패스워드 정책을 즉시 변경하라고 명시했지만, 실제로 변경하지 않은 계정이 있을 가능성이 높다. 수집한 유저명 전체에 대해 기본 패스워드로 SSH 접속을 시도한다.

### SSH 접속

여러 계정 중 `christine`이 기본 패스워드를 변경하지 않은 것을 확인한다.

```bash
ssh christine@$IP
# password: funnel123#!#
```

![ssh-login](./images/ssh-login.png)

`Welcome to Ubuntu 20.04.5 LTS` 메시지와 함께 접속에 성공한다.

---

## Privilege Escalation / Internal Reconnaissance

### 내부 포트 확인

SSH 접속 후 내부에서 열려 있는 포트를 확인한다. nmap에서 보이지 않던 서비스가 로컬에 바인딩되어 있을 수 있기 때문이다.

```bash
ss -tlnp
```

![ss-tlnp](./images/ss-tlnp.png)

주목할 포트는 두 개다.

- `127.0.0.1:5432` — PostgreSQL이 루프백에만 바인딩되어 있어 외부에서 직접 접근 불가
- `127.0.0.1:38369` — 용도 불명

PostgreSQL(`5432`)이 내부에서만 동작 중이므로, SSH 로컬 포트 포워딩으로 로컬 머신에 터널을 만들어 접근한다.

---

## Flag

### VPN 연결 확인

포트 포워딩은 HTB VPN이 연결된 상태의 로컬 머신(맥북)에서 실행해야 한다.

![vpn-connect](./images/vpn-connect.png)

`Initialization Sequence Completed` 메시지로 VPN 연결 상태를 확인한다.

### SSH 로컬 포트 포워딩

```bash
ssh -L 1234:localhost:5432 christine@$IP
```

![ssh-tunnel](./images/ssh-tunnel.png)

이 명령은 로컬 머신의 `1234` 포트로 들어오는 트래픽을 SSH 터널을 통해 타깃의 `127.0.0.1:5432`로 전달한다. 즉, 로컬에서 `localhost:1234`에 접속하면 타깃의 PostgreSQL에 접근할 수 있다.

### PostgreSQL 접속

```bash
psql -U christine -h localhost -p 1234
```

![psql-connect](./images/psql-connect.png)

SSH와 동일한 크리덴셜(`christine` / `funnel123#!#`)로 PostgreSQL 접속에 성공한다.

### 데이터베이스 탐색

```sql
\l
```

![psql-list-db](./images/psql-list-db.png)

데이터베이스 목록에서 `secrets`가 눈에 띈다. 기본 제공 DB(`postgres`, `template0`, `template1`)와 달리 사용자가 직접 만든 DB이므로 우선 탐색 대상이다.

```sql
\c secrets
\dt
```

![psql-secrets](./images/psql-secrets.png)

`secrets` DB에 `flag` 테이블이 존재한다.

```sql
SELECT * FROM flag;
```

![flag](./images/flag.png)

플래그를 획득한다.

---

## Attack Chain

```
FTP Anonymous Login
        |
        v
mail_backup/password_policy.pdf  →  기본 패스워드 확인 (funnel123#!#)
mail_backup/welcome_28112022     →  유저명 목록 확인
        |
        v
SSH (christine / funnel123#!#)
        |
        v
ss -tlnp  →  127.0.0.1:5432 (PostgreSQL 내부 바인딩 확인)
        |
        v
SSH Local Port Forwarding (-L 1234:localhost:5432)
        |
        v
psql → secrets DB → flag 테이블
        |
        v
      FLAG
```

---

## Key Takeaways

- FTP 익명 로그인은 항상 초반 체크 항목에 포함해야 한다. 내부 문서가 노출되면 크리덴셜로 이어지는 경우가 많다.
- 패스워드 정책 문서에 예시로 적힌 기본 패스워드가 실제로 사용 중인 계정이 있었다. 문서 자체가 공격 벡터가 된 케이스다.
- `ss -tlnp`는 루프백에 바인딩된 숨겨진 서비스를 찾는 핵심 명령이다. nmap 외부 스캔에서는 보이지 않는다.
- SSH 로컬 포트 포워딩(`-L`)은 내부 서비스를 로컬로 끌어오는 가장 깔끔한 방법이다. 별도 도구 없이 SSH 하나로 처리된다.
