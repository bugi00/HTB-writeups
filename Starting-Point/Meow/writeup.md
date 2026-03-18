# Meow - Hack The Box Writeup

## 1. 개요

Machine: Meow  
Difficulty: Very Easy  
Operating System: Linux  

이 문제는 Telnet 서비스의 잘못된 인증 설정을 이용하여 시스템에 접근하는 문제이다.  
핵심은 서비스 식별 이후 인증 없이 접속이 가능한지 확인하는 것이다.

---

## 2. Enumeration

대상 시스템의 열린 포트와 실행 중인 서비스를 확인한다.

nmap -Pn -sC -sV <TARGET_IP>

![nmap](./images/nmap.png)

결과:

23/tcp open  telnet  

23번 포트에서 Telnet 서비스가 실행 중인 것을 확인할 수 있다.  
Telnet은 원격 접속 프로토콜이므로, 서비스 식별 이후 직접 접속을 시도하는 것이 중요하다.

---

## 3. Analysis

Telnet은 원격 접속을 위한 프로토콜이다.

주요 특징은 다음과 같다.

- 평문 통신을 사용한다.
- 일반적으로 사용자 인증이 필요하다.
- 설정이 잘못된 경우 인증 없이 접근이 가능할 수 있다.

따라서 Telnet 서비스가 확인되면, 인증이 정상적으로 적용되어 있는지 확인해야 한다.  
이 문제에서는 인증 절차가 미흡할 가능성을 우선적으로 고려하고 직접 접속을 시도한다.

---

## 4. Exploitation

Telnet 서비스를 이용하여 대상 시스템에 접속한다.

telnet <TARGET_IP>

![telnet login](./images/telnet-login.png)

접속 후 로그인 프롬프트가 출력된다.

login:

일반적으로 기본 계정 또는 인증 절차가 약한 계정을 우선적으로 시도할 수 있다.  
이 문제에서는 root 계정으로 로그인을 시도한다.

root

![telnet shell](./images/telnet-shell.png)

별도의 비밀번호 입력 없이 로그인에 성공하며 쉘에 접근할 수 있다.  
즉, 인증 없이 원격 시스템 접근이 가능한 상태이다.

---

## 5. Flag 획득

로그인 이후 현재 디렉토리의 파일을 확인한다.

ls

flag 파일을 확인한 후 내용을 출력한다.

cat flag.txt

이를 통해 최종적으로 flag를 획득할 수 있다.

---

## 6. Root Cause

Telnet 서비스가 외부에 노출되어 있으며,  
root 계정에 대해 정상적인 인증 절차가 적용되지 않고 있다.

그 결과 비밀번호 없이 로그인할 수 있으며,  
인증 없는 원격 접속이 가능하도록 설정된 것이 근본적인 원인이다.

---

## 7. 사용 명령어

nmap -Pn -sC -sV <TARGET_IP>  
telnet <TARGET_IP>  
ls  
cat flag.txt  

---

## 8. 결론

이 문제는 Telnet 서비스의 잘못된 인증 설정이 얼마나 치명적인 결과로 이어질 수 있는지 보여준다.  
서비스 식별 이후 해당 프로토콜의 특성을 이해하고,  
인증 우회 가능성을 빠르게 점검하는 것이 핵심이다.

특히 Telnet과 같이 보안성이 낮은 프로토콜은  
외부에 노출되지 않도록 제한하고,  
정상적인 인증 절차가 반드시 적용되도록 관리해야 한다.
