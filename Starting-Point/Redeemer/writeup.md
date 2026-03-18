# Redeemer

## 개요
이 문제는 기본 포트 스캔으로는 확인되지 않는 서비스를 탐지하고, Redis 서버에 인증 없이 접근하여 메모리에 저장된 데이터를 획득하는 과정에 초점을 둔다. 핵심은 포트 스캔 범위를 확장하고, Redis의 기본 설정 취약점을 이해하는 것이다.

---

## 대상 정보
- Target IP: <TARGET_IP>
- OS: Linux
- Service: Redis

---

## 1. 포트 스캔

기본 nmap 스캔은 상위 1000개의 포트만 검사하기 때문에 일부 서비스는 탐지되지 않을 수 있다. 따라서 전체 포트를 대상으로 스캔을 수행한다.

nmap -p- --min-rate 5000 <TARGET_IP>

![nmap](images/nmap-full.png)

스캔 결과, 6379 포트에서 Redis 서비스가 실행 중인 것을 확인할 수 있다.

---

## 2. Redis 서비스 접근

Redis는 전용 CLI 도구를 통해 접근할 수 있으며, 인증이 설정되어 있지 않은 경우 외부에서도 접근이 가능하다.

redis-cli -h <TARGET_IP>

![redis connect](images/redis-connect.png)

접속에 성공하면 즉시 명령어를 실행할 수 있는 상태가 된다.

---

## 3. 서버 정보 확인

서버의 상태 및 설정을 확인하기 위해 다음 명령어를 사용한다.

INFO

![redis info version](images/redis-info-1.png)

위 결과를 통해 Redis 버전 및 실행 환경을 확인할 수 있다.

또한 Keyspace 정보를 통해 데이터 존재 여부를 확인할 수 있다.

![redis info keyspace](images/redis-info-2.png)

db0에 4개의 key가 존재하는 것을 확인할 수 있다.

---

## 4. 데이터베이스 탐색

Redis는 여러 개의 논리적 데이터베이스를 지원하며, 기본적으로 DB 0이 사용된다. Keyspace에서 db0에 데이터가 존재하는 것이 확인되었으므로 해당 DB를 선택한다.

SELECT 0

데이터 개수를 확인한다.

DBSIZE

---

## 5. 키 확인 및 데이터 추출

데이터베이스 내 key 목록을 확인한다.

KEYS *

이후, flag 값을 조회한다.

GET flag

![redis flag](images/redis-flag.png)

이를 통해 메모리에 저장된 flag 값을 획득할 수 있다.

---

## 핵심 정리

- 기본 포트 스캔은 모든 서비스를 발견하지 못할 수 있다
- 전체 포트 스캔을 통해 숨겨진 서비스를 발견할 수 있다
- Redis는 인증 없이 접근 가능한 경우가 존재한다
- 메모리 기반 데이터베이스는 민감한 정보를 그대로 노출할 수 있다
- 서비스 발견 후 직접 접근하여 확인하는 과정이 중요하다
