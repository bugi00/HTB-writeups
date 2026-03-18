# Meow - Hack The Box Writeup

## 1. Overview

Machine: Meow
Difficulty: Very Easy
Operating System: Linux

본 문제는 Telnet 서비스의 설정 취약점을 이용하여 시스템에 접근하는 과정이다.
핵심은 서비스 식별 이후 직접 접근을 시도하는 기본적인 흐름을 이해하는 것이다.

---

## 2. Enumeration

대상 시스템의 열린 포트와 실행 중인 서비스를 확인한다.

nmap -sC -sV <TARGET_IP>
![nmap result](./images/nmap.png)

결과:

23/tcp open  telnet

Telnet 서비스가 외부에 노출되어 있음을 확인할 수 있다.

---

## 3. Analysis

Telnet은 암호화되지 않은 평문 통신을 사용하는 프로토콜이다.
인증 정보가 보호되지 않으며, 보안 설정이 미흡한 경우 무단 접근이 가능하다.

이러한 특성 때문에 인증 없이 접속이 가능한지 확인하는 것이 우선적인 접근 방법이다.

---

## 4. Exploitation

Telnet 서비스를 이용하여 대상 시스템에 접속한다.

telnet <TARGET_IP>
![telnet login](./images/telnet-login.png)

접속 시 로그인 프롬프트가 출력된다.

login:

일반적으로 기본 계정 또는 비밀번호가 설정되지 않은 계정을 우선적으로 시도한다.

root
![telnet shell](./images/telnet-shell.png)

별도의 비밀번호 입력 없이 로그인에 성공한다.

---

## 5. Flag Retrieval

로그인 이후 현재 디렉토리의 파일을 확인한다.

ls

flag 파일을 확인한 후 내용을 출력한다.

cat flag.txt

---

## 6. Root Cause

Telnet 서비스가 외부에 노출되어 있으며,
root 계정에 대한 인증 절차가 존재하지 않는다.

인증 없는 원격 접속이 가능하도록 설정된 것이 근본적인 원인이다.

---

## 7. Commands Summary

nmap -sC -sV <TARGET_IP>
telnet <TARGET_IP>
ls
cat flag.txt

---

## 8. Conclusion

서비스 식별 이후 해당 서비스의 특성을 기반으로 접근 방법을 결정하는 과정이 중요하다.
Telnet과 같이 보안성이 낮은 프로토콜은 우선적으로 점검해야 한다.

