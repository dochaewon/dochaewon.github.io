---
layout: post
title: "서버에서 외부 URL 호출되는지 확인하는 3가지 방법 (curl, ss, tcpdump)"
date: 2025-07-24
categories: [linux, troubleshooting]
tags: [curl, tcpdump, ss, 서버네트워크, 오픈서치, 실무팁]
---

서버에서 특정 외부 URL(OpenSearch, 외부 API 등)에 **정말 요청이 나가고 있는지 확인**하고 싶을 때가 있다.  
코드에서는 호출했지만 **실제 네트워크 요청이 발생하는지**, **방화벽이나 보안그룹에 막힌 건 아닌지** 궁금할 때 아래 세 가지 방법을 사용하면 실시간으로 확인할 수 있다.

---

## 1️⃣ `curl` 명령어로 직접 요청

가장 간단한 방법은 직접 `curl`로 호출해보는 것이다.(실제 가장 많이 사용하는 방법임.)

```bash
curl -X GET "https://search-carboom-search-engine-test-xxxx.ap-northeast-2.es.amazonaws.com" -i
```

### ✅ 기대 결과
- **2xx / 4xx 응답이 오면 통신은 된 것** → 호출 성공
- **timeout** 또는 DNS 에러면 → 방화벽, 보안그룹, VPC 설정 확인 필요

> 참고: 인증 오류(403, 401)는 괜찮음. 연결이 되었다는 뜻이므로, **“호출은 됐다”**고 판단할 수 있다.

---

## 2️⃣ `ss` 또는 `netstat` 로 HTTPS 연결 추적

> 이 방법은 **실행 중인 프로세스가 외부와 연결을 맺고 있는 상태**에서만 확인 가능하다.  
> 즉, 백그라운드 프로세스가 지속적으로 요청을 보내고 있는 상황에 적합하다.

### `ss` 사용 예시

```bash
sudo ss -tnp | grep 443
```

- 결과가 없다면 현재 연결 중인 HTTPS 요청이 없는 상태일 수 있다.

### `netstat`은 기본 설치되어 있지 않음

```bash
sudo apt install net-tools
netstat -tnp | grep 443
```

---

## 3️⃣ `tcpdump`로 실시간 패킷 감시

가장 강력한 방법은 네트워크 패킷 자체를 확인하는 `tcpdump`다.

```bash
sudo apt install tcpdump
sudo tcpdump -i any host search-carboom-search-engine-test-xxxx.ap-northeast-2.es.amazonaws.com
```

또는 IP로 확인하려면 먼저 도메인을 IP로 변환:

```bash
dig +short search-carboom-search-engine-test-xxxx.ap-northeast-2.es.amazonaws.com
```

그리고:

```bash
sudo tcpdump -i any host [IP주소]
```

### 🔍 주요 포인트
- HTTPS라 패킷 내용을 볼 수는 없지만, **통신 발생 여부는 확인 가능**
- 요청이 나가고 있는지, 어디에서 나가는지 확인 가능

---

## 🔚 마무리

| 목적 | 방법 |
|------|------|
| 직접 연결 테스트 | `curl` |
| 실시간 연결 확인 | `ss`, `netstat` *(단, 연결 유지 상태에서만)* |
| 패킷 감시 | `tcpdump` *(가장 확실함)* |

단발성 요청이라면 `curl`,
지속적인 연결 추적이라면 `tcpdump`가 가장 확실하고 실전에서 자주 사용된다.

---

## 🏷️ 태그
#curl #tcpdump #ss #netstat #서버디버깅 #네트워크확인 #OpenSearch #실무팁 #리눅스명령어 #깃허브블로그
