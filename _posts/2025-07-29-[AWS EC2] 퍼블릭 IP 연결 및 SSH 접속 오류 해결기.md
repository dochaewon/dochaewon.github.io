---
title: "[AWS EC2] 퍼블릭 IP 연결 및 SSH 접속 오류 해결기"
date: 2025-07-29
categories: [AWS, EC2, Troubleshooting]
tags: [ec2, ssh, 탄력적IP, known_hosts, aws]
---

회사에서 운영 중인 EC2 인스턴스에 퍼블릭 IP를 연결하고 SSH 접속을 시도하는 과정에서 여러 오류를 겪었다.  
이번 글에서는 실시간으로 겪은 문제 해결 과정을 기록해둔다.

---

## ✅ EC2 인스턴스에 퍼블릭 IP가 없는 이유

EC2 인스턴스 목록을 확인하다 보니, `carboom-chasalddae-crm-prod` 인스턴스에 퍼블릭 IP가 없었다.

**가능한 원인:**

- 퍼블릭 서브넷이 아님
- 인스턴스 생성 시 `퍼블릭 IP 자동 할당` 옵션이 꺼져 있었음

```
프라이빗 IP: 172.21.15.164
퍼블릭 IP: 없음
```

---

## ✅ 탄력적 IP(EIP) 수동 할당

퍼블릭 IP가 필요해서 **탄력적 IP** 를 할당하고 EC2 인스턴스에 수동으로 연결했다.

- **탄력적 IP**: `3.35.211.216`
- **인스턴스 ID**: `i-018d2b9200cf27f5b`
- **프라이빗 IP**: `172.21.15.164`

> AWS 콘솔 > EC2 > 탄력적 IP > 연결 → 문제없이 연결 완료

---

## ✅ SSH 접속 시도 → 접속 실패

```bash
ssh -i ~/.ssh/carboom.pem ubuntu@ec2-3-35-211-216.ap-northeast-2.compute.amazonaws.com
```

- 접속이 되지 않음
- 사용자명(`ubuntu` vs `ec2-user`)이나 `.pem` 키 이름 불일치 가능성 확인

> AWS EC2 콘솔 > 인스턴스 > 키 페어 이름이 현재 사용 중인 `.pem` 파일과 일치해야 함

---

## ⚠️ SSH 호스트 키 충돌 에러 발생

```text
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Host key verification failed.
```

이는 동일한 퍼블릭 IP가 과거 다른 인스턴스에 연결됐었고, `~/.ssh/known_hosts` 에 저장된 호스트 키와 현재 키가 다르기 때문에 발생하는 보안 에러다.

### 🔧 해결 방법

```bash
ssh-keygen -R ec2-3-35-211-216.ap-northeast-2.compute.amazonaws.com
ssh-keygen -R 3.35.211.216
```

해당 호스트 정보를 known_hosts 에서 삭제 후 재접속 시 정상 작동.

---

## ✅ 보안 그룹 인바운드 규칙 확인

EC2 인스턴스에 연결된 보안 그룹에서 **포트 22 (SSH)** 가 열려 있어야 한다.

```text
포트 범위: 22
프로토콜: TCP
소스: 0.0.0.0/0 또는 본인 IP
```

보안 그룹 이름: `carboom-front-services-prod-sg`

> 이미 설정되어 있었으므로 따로 수정할 필요는 없었다.

---

## ✅ 최종 접속 명령어

```bash
ssh -i ~/.ssh/carboom.pem ubuntu@ec2-3-35-211-216.ap-northeast-2.compute.amazonaws.com
```

또는,

```bash
ssh -i ~/.ssh/carboom.pem ec2-user@ec2-3-35-211-216.ap-northeast-2.compute.amazonaws.com
```

AMI 종류에 따라 사용자명을 바꿔야 접속 가능함.

---

## 📌 정리

| 항목                | 확인 및 조치 |
|---------------------|--------------|
| 퍼블릭 IP 없음       | 탄력적 IP 할당 |
| SSH 접속 실패        | 키 페어 이름, 사용자명 확인 |
| Host key 충돌       | `ssh-keygen -R` 로 삭제 |
| 포트 차단 여부       | 보안 그룹 인바운드 22번 확인 |

---

## 💬 결론

단순히 SSH가 안 된다고 해서 키만 의심하면 안 된다.  
퍼블릭 IP, 키 페어, 사용자명, 보안 그룹, known_hosts 충돌 여부까지 꼼꼼히 확인하는 습관이 중요하다.
