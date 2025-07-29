---
title: "[AWS] 'ubuntu'와 'ec2-user' 계정의 차이점은 무엇인가?"
date: 2025-07-29
categories: [Linux, AWS, EC2]
tags: [ubuntu, ec2-user, ssh, 리눅스계정, aws]
---

AWS EC2 인스턴스를 사용할 때 자주 보이는 계정 이름인 `ubuntu`와 `ec2-user`.  
둘 다 처음 로그인할 때 사용하는 기본 계정이지만, 실제로 어떤 차이가 있을까?

---

## ✅ 기본 차이 요약

| 항목 | `ubuntu` | `ec2-user` |
|------|----------|------------|
| 제공 OS | Ubuntu AMI (Canonical) | Amazon Linux / RHEL / CentOS |
| 계정 목적 | 우분투용 표준 로그인 계정 | AWS 자체 이미지용 표준 계정 |
| 홈 디렉토리 | `/home/ubuntu/` | `/home/ec2-user/` |
| 패키지 관리자 | `apt` 계열 (`apt-get`, `snap`) | `yum`, `dnf` |
| 기본 권한 | `sudo` 가능, 비번 없이 | 동일 |
| SSH 정책 | 키 기반 로그인 (비번 불가) | 동일 |

---

## ✅ 왜 계정이 다른가?

AWS에서 다양한 리눅스 배포판을 제공하기 때문에, **OS마다 보안 및 초기 환경 설정이 다르며**,  
이에 따라 사용되는 기본 계정도 다르게 설정되어 있다.

예:

- **Ubuntu 20.04/22.04**: `ubuntu` 계정 제공
- **Amazon Linux 2 / 2023**: `ec2-user` 계정 제공
- **RHEL / CentOS**: `ec2-user` 또는 `centos`
- **Fedora**: `ec2-user` 또는 `fedora`
- **Debian**: `admin`

---

## ✅ 기능적으로 같은 점

- `sudo`를 통해 루트 권한 명령어 실행 가능
- 기본적으로 `ssh` 키 기반 인증을 사용
- AWS에서 제공하는 보안 정책에 맞춰 비밀번호 로그인은 차단됨

---

## ✅ 접속과 무관한 시스템적인 차이

| 항목 | ubuntu | ec2-user |
|------|--------|----------|
| `~/.bashrc` 초기 내용 | Ubuntu 전용 프롬프트/환경 구성 | Amazon Linux 전용 구성 |
| `cloud-init` 동작 | Ubuntu cloud-init 템플릿 반영 | Amazon Linux cloud-init 템플릿 반영 |
| 서비스 기본 포트 | OS 구성에 따라 달라짐 | 다름 |
| 사용자 그룹 | `ubuntu`, `sudo` | `ec2-user`, `wheel` |

---

## ✅ 결론

`ubuntu`와 `ec2-user`는 단순히 이름만 다른 것이 아니라 **운영체제와 연동된 전용 계정**이다.

EC2 인스턴스에 접속할 때 사용자명을 구분하는 것은 단순 SSH 문제가 아니라,  
각 운영체제의 기본 구조를 이해하는 시작점이기도 하다.

> 이 둘은 "누구냐 넌?" 이 아니라 "어디서 왔니?"로 구분해야 한다.
