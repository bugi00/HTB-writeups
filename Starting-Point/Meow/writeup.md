# Meow - Hack The Box Writeup

## 1. 개요

Machine: Meow
Difficulty: Very Easy
Operating System: Linux

이 문제는 Telnet 서비스의 잘못된 인증 설정을 이용하는 문제이다.
인증 없이 접속이 가능한지 확인하는 것이 핵심이다.

---

## 2. Enumeration

열린 포트와 서비스를 확인한다.

nmap -Pn -sC -sV <TARGET_IP>

결과:

23/tcp open  telnet

Telnet 서비스가 실행 중인 것을 확인할 수 있다.
Telnet은 인증이 필요한 서비스지만 설정에 따라 인증 없이 접근이 가능할 수 있다.

---

## 3. Analysis

Telnet은 원격 접속을 위한 프로토콜이다.

특징:

* 평문 통신 사용
* 일반적으로 인증 필요
* 설정에 따라 인증 없이 접근 가능

따라서 인증 없이 접속이 가능한지 확인하는 것이 중요하다.

---

## 4. Exploitation

Telnet으로 접속을 시도한다.

telnet <TARGET_IP>

접속 후 사용자명을 입력한다.

root

비밀번호 없이 로그인에 성공한다.

---

## 5. Flag 획득

파일 목록을 확인한다.

ls

flag 파일을 확인하고 내용을 출력한다.

cat flag.txt

---

## 6. Root Cause

Telnet 서비스에서 비밀번호 없이 로그인이 가능하도록 설정되어 있다.

이로 인해 인증 우회가 발생한다.

---

## 7. 사용 명령어

nmap -Pn -sC -sV <TARGET_IP>
telnet <TARGET_IP>
ls
cat flag.txt

---

## 8. 결론

Telnet 서비스의 인증 설정이 잘못되면 시스템 전체 접근이 가능하다.
기본적인 인증 설정이 매우 중요하다.
