Dancing - Hack The Box Writeup

1. 개요

Machine: Dancing  
Difficulty: Very Easy  
Operating System: Windows  

이 문제는 SMB 서비스에서 null session이 허용된 설정을 이용하여  
인증 없이 공유 디렉토리에 접근하고 파일을 획득하는 문제이다.

핵심은 SMB 서비스가 보였을 때,  
복잡한 공격보다 “인증 없이 접근 가능한지”를 먼저 확인하는 것이다.


2. Enumeration

대상 시스템의 열린 포트와 실행 중인 서비스를 확인한다.

nmap -Pn -sC -sV <TARGET_IP>

<img src="images/nmap.png" alt="nmap"/>

결과:

445/tcp open  microsoft-ds  

445번 포트에서 SMB 서비스가 실행 중인 것을 확인할 수 있다.

SMB는 네트워크 파일 공유 서비스이기 때문에,  
접속만 가능하다면 바로 파일 접근으로 이어질 수 있다.

또한 설정이 잘못된 경우  
인증 없이 접근(null session)이 가능한 경우가 많다.

따라서 다음 단계는 추가 스캔이 아니라  
SMB 접근 가능 여부를 바로 확인하는 것이다.


3. Analysis

SMB(Server Message Block)의 특징은 다음과 같다.

- 네트워크 공유 디렉토리 접근 가능  
- 일반적으로 사용자 인증 필요  
- 설정 오류 시 인증 없이 접근 가능 (null session)

이 문제에서는 Starting Point 난이도이기 때문에  
복잡한 취약점보다 설정 오류를 먼저 의심하는 것이 합리적이다.

즉, 가장 먼저 해야 할 것은  
“인증 없이 접근 가능한지 확인”이다.


4. Exploitation

SMB 공유 목록을 확인한다.

smbclient -L //<TARGET_IP> -N

<img src="images/smb-shares.png" alt="smb shares"/>

옵션 설명:

-L : 공유 목록 조회  
-N : 비밀번호 없이 접속 (null session 시도)

여기서 중요한 점은  
계정 공격을 하기 전에 null session을 먼저 시도했다는 것이다.

결과에서 여러 share가 확인된다.

그 중 WorkShares share가  
인증 없이 접근 가능한 것을 확인할 수 있다.

해당 share에 접속한다.

smbclient //<TARGET_IP>/WorkShares -N

<img src="images/smb-login.png" alt="smb login"/>

접속에 성공하면 SMB 쉘에 진입한다.

이 시점에서 이미 인증 없이 파일 시스템 접근이 가능한 상태이다.


5. Flag 획득

파일 목록을 확인한다.

ls

<img src="images/smb-ls.png" alt="smb ls"/>

디렉토리를 탐색한 후 flag 파일을 확인한다.

flag 파일을 다운로드한다.

get flag.txt

이를 통해 flag를 획득할 수 있다.

여기서 중요한 점은  
추가적인 exploit 없이 파일 접근이 가능했다는 것이다.


6. Root Cause

이 문제의 근본 원인은 다음과 같다.

- SMB 서비스가 외부에 노출되어 있다  
- null session 접근이 허용되어 있다  
- 인증 없이 공유 디렉토리 접근이 가능하다  

즉, 접근 제어가 제대로 적용되지 않은 상태이다.

이 설정은 공격자가 별도의 취약점 분석 없이도  
즉시 내부 파일을 열람할 수 있도록 만든다.


7. 사용 명령어

nmap -Pn -sC -sV <TARGET_IP>  

smbclient -L //<TARGET_IP> -N  

smbclient //<TARGET_IP>/WorkShares -N  

ls  

get flag.txt  


8. 결론

이 문제는 서비스 식별 이후  
해당 서비스의 특성을 기반으로  
가장 가능성 높은 약점을 먼저 확인하는 것이 중요하다는 것을 보여준다.

SMB가 열려 있다면:

- 인증 없이 접근 가능한지 확인  
- 접근 가능한 share 확인  
- 파일 탐색 및 다운로드  

이 순서로 진행하는 것이 가장 효율적이다.

특히 이 문제에서는 다음 판단이 핵심이었다.

445 포트에서 SMB 서비스 확인  
파일 접근 서비스임을 인지  
null session 가능성 확인  
인증 없이 share 접근 성공  
flag 획득  

실제 환경에서는 다음과 같은 대응이 필요하다.

- null session 비활성화  
- SMB 접근 제어 강화  
- 외부 노출 최소화  
- 인증 정책 적용  

이 문제는 단순하지만,  
잘못된 설정 하나로 전체 시스템이 노출될 수 있다는 점에서 중요한 사례이다.
