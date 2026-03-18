Dancing - Hack The Box Writeup

1. 개요

Machine: Dancing  
Difficulty: Very Easy  
Operating System: Windows  

이 문제는 SMB 서비스에서 잘못된 접근 제어 설정(null session)을 이용하여
인증 없이 공유 디렉토리에 접근하고 파일을 획득하는 문제이다.

핵심은 단순히 SMB에 접속하는 것이 아니라,

왜 SMB에서 “인증 없이 접근 가능한지”를 먼저 의심했고  
왜 null session을 바로 시도했는지를 이해하는 것이다.

---

2. Enumeration

대상 시스템의 열린 포트와 실행 중인 서비스를 확인한다.

nmap -Pn -sC -sV <TARGET_IP>

결과:

445/tcp open  microsoft-ds  

SMB 서비스가 실행 중이며, Windows 기반 시스템임을 확인할 수 있다.

여기서 SMB에 주목한 이유는 다음과 같다.

- SMB는 네트워크 파일 공유 서비스이다.
- 인증이 필요하지만, 설정에 따라 인증 없이 접근(null session)이 가능하다.
- 잘못된 설정일 경우 내부 파일 접근으로 바로 이어진다.

즉, 웹처럼 추가 분석이 필요한 서비스가 아니라  
“접속 가능 여부 자체가 공격 성공 여부를 결정하는 서비스”이다.

따라서 다음 단계는 취약점 탐색이 아니라,

👉 “인증 없이 접근 가능한지 확인”이다.

---

3. Analysis

SMB(Server Message Block)의 핵심 특징:

- 네트워크 공유 디렉토리 접근 가능
- 일반적으로 사용자 인증 필요
- 설정 오류 시 null session 허용 가능

이 문제에서의 판단 흐름은 다음과 같다.

1. SMB는 파일 접근 서비스다  
2. 인증이 약하면 즉시 데이터 접근 가능  
3. Starting Point 환경 → 복잡한 exploit보다 설정 취약점 가능성 높음  

따라서 가장 합리적인 전략은:

👉 인증 없이 share 접근 가능한지 먼저 확인

이 단계에서 공격 방향은 이미 확정된다.

---

4. Exploitation

SMB 공유 목록을 확인한다.

smbclient -L //<TARGET_IP> -N

- `-L` : 공유 목록 조회  
- `-N` : 비밀번호 없이 접속 (null session 시도)

이 옵션 선택 자체가 중요하다.

👉 이유:

- 인증을 요구하는지 확인하기 위한 테스트
- 불필요한 계정 추측보다 훨씬 효율적인 1차 검증

결과에서 여러 share가 확인된다.

여기서 핵심 판단:

- 어떤 share가 접근 가능한가?
- 인증 요구 없이 들어갈 수 있는가?

WorkShares share가 인증 없이 접근 가능함을 확인한다.

즉,

👉 이미 내부 파일 접근 권한이 확보된 상태이다.

---

해당 share에 접속한다.

smbclient //<TARGET_IP>/WorkShares -N

접속 성공 시 SMB 쉘에 진입한다.

이 시점에서 확인할 수 있는 사실:

- 인증 없이 파일 시스템 접근 가능
- 내부 데이터 열람 가능

즉, 접근 제어가 완전히 실패한 상태이다.

---

5. Flag 획득

파일 목록을 확인한다.

ls

여기서 중요한 판단:

- 이미 권한 확보 완료 상태
- 추가 exploit 필요 없음

따라서 전략은 단순하다:

👉 “탐색 → 파일 식별 → 다운로드”

디렉토리를 탐색한다.

cd <directory_name>

flag 파일을 확인한 후 다운로드한다.

get flag.txt

이로써 flag를 획득할 수 있다.

핵심은 flag 자체가 아니라,

👉 인증 없이 내부 파일 접근이 가능했다는 점이다.

---

6. Root Cause

이 문제의 근본 원인은 다음과 같다.

- SMB 서비스가 외부에 노출되어 있다
- null session 접근이 허용되어 있다
- 인증 없이 공유 디렉토리 접근 가능하다

즉,

👉 접근 제어가 완전히 실패한 상태

이 설정은 공격자가 별도의 취약점 분석 없이도  
즉시 내부 데이터 접근 → 다운로드를 가능하게 만든다.

---

7. 사용 명령어

nmap -Pn -sC -sV <TARGET_IP>  

smbclient -L //<TARGET_IP> -N  

smbclient //<TARGET_IP>/WorkShares -N  

ls  

cd <directory_name>  

get flag.txt  

---

8. 결론

이 문제는 서비스 식별 이후,

👉 서비스의 “기능”을 기반으로 공격 방향을 빠르게 결정하는 것이 핵심이다.

SMB가 보이면 해야 할 판단:

- 파일 접근 서비스인가? → YES  
- 인증이 필요한가? → 확인 필요  
- null session 가능성 있는가? → 높음  

따라서:

👉 “계정 공격보다 null session이 우선”

이 문제에서 핵심 판단 흐름:

- 445 포트 → SMB 확인  
- SMB는 파일 접근 서비스  
- Starting Point → 설정 취약점 가능성 높음  
- null session 시도  
- 인증 없이 share 접근 성공  
- 파일 탐색 → flag 획득  

---

실제 환경에서의 대응:

- null session 비활성화
- SMB 접근 제어 강화
- 외부 노출 최소화
- 인증 정책 적용

---

이 문제의 본질은 “취약한 서비스 설정”이다.

SMB 자체가 문제가 아니라,

👉 인증 없는 접근이 가능하도록 방치된 설정이 문제이다.

단순하지만 실제 환경에서도 매우 자주 발생하는 치명적인 보안 실수다.
