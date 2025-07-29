---
title: "[Laravel] Supervisor로 여러 Queue(worker) 병렬 실행하기"
date: 2025-07-29
categories: [Laravel, Queue, Production]
tags: [laravel, queue, supervisor, artisan, 큐관리, 병렬처리]
---

운영 중인 Laravel 프로젝트에서 큐를 통해 비동기 작업을 처리하고 있다.  
그 중 특정 큐(`notice_inventory`)는 이미 Supervisor로 관리되고 있었고,  
새롭게 `user_push_log`라는 큐도 병렬로 실행할 필요가 생겼다.

이 글에서는 **Supervisor로 Laravel 큐를 병렬로 다중 운영하는 구성**을 실전 예제로 정리한다.

---

## ✅ 기본 환경

- Laravel 10.x
- PHP 8.x
- Queue Driver: `database`
- Supervisor로 `queue:work` 관리 중
- 기존 큐: `notice_inventory`
- 새로 추가할 큐: `user_push_log`

---

## ✅ 기존 Supervisor 설정 확인

```bash
# 위치: /etc/supervisor/conf.d/laravel-queue-worker.conf

[program:laravel-queue-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/kbwiserent-crm/artisan queue:work --queue=notice_inventory --sleep=3 --tries=3
autostart=true
autorestart=true
user=ubuntu
numprocs=8
redirect_stderr=true
stdout_logfile=/var/www/kbwiserent-crm/storage/logs/worker.log
````

* `notice_inventory` 큐 전용 워커
* 8개의 프로세스로 병렬 실행
* 정상 작동 중

---

## ✅ 추가할 큐: `user_push_log`

이제 `user_push_log` 큐를 따로 관리하는 Worker 설정을 만든다.

### 📄 1. Supervisor 설정 파일 생성

```bash
sudo nano /etc/supervisor/conf.d/laravel-user-push-log.conf
```

내용:

```ini
[program:laravel-user-push-log]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/kbwiserent-crm/artisan queue:work --queue=user_push_log --sleep=3 --tries=3
autostart=true
autorestart=true
user=ubuntu
numprocs=4
redirect_stderr=true
stdout_logfile=/var/www/kbwiserent-crm/storage/logs/worker-user-push-log.log
stopwaitsecs=3600
```

> * `numprocs=4`: 동시에 4개의 워커가 실행됨
> * `stdout_logfile`: 로그를 별도로 기록 (디버깅 시 유용)

---

## ✅ Supervisor에 설정 반영

```bash
# 설정 재읽기
sudo supervisorctl reread

# 새 설정 등록
sudo supervisorctl update

# user_push_log 워커 시작
sudo supervisorctl start laravel-user-push-log:*
```

---

## ✅ 상태 확인

```bash
sudo supervisorctl status
```

예시 출력:

```
laravel-queue-worker_00     RUNNING
laravel-queue-worker_01     RUNNING
...
laravel-user-push-log_00    RUNNING
laravel-user-push-log_01    RUNNING
...
```

---

## ✅ 전체 구조 요약

| 큐 이름               | Supervisor 설정 파일             | 프로세스 수 | 처리 대상     |
| ------------------ | ---------------------------- | ------ | --------- |
| `notice_inventory` | `laravel-queue-worker.conf`  | 8개     | 재고 알림     |
| `user_push_log`    | `laravel-user-push-log.conf` | 4개     | 사용자 푸시 로그 |

---

## ⚠️ 왜 큐를 나눠야 하는가?

Laravel에서 다음처럼 큐를 여러 개 지정할 수 있다:

```bash
php artisan queue:work --queue=notice_inventory,user_push_log
```

하지만 이 방식은 `notice_inventory` 큐가 비워지지 않으면 `user_push_log`는 대기 상태가 되어 **우선순위 역전(blocking)** 이 발생한다.

따라서 **큐마다 Supervisor 워커를 분리**하면 다음과 같은 장점이 있다:

* **큐별 병렬 처리량 조절 가능**
* **우선순위에 따른 리소스 분배 가능**
* **디버깅 및 로그 관리 간편**
* **하나의 큐 장애가 다른 큐에 영향 주지 않음**

---

## 🧼 추가 팁

* `.env`에서 큐 이름을 관리하도록 구성하면 배포 시 유연성 증가
* `queue:restart`를 배포 스크립트에 추가해 코드 변경 후에도 워커 갱신 가능
* `failed_jobs` 테이블 모니터링 + Slack 등 알림 연동 추천
* 로그가 쌓이면 cron + `find`로 정리

```bash
find /var/www/kbwiserent-crm/storage/logs/ -name "*.log" -mtime +7 -delete
```

---

## ✅ 결론

Laravel에서 큐를 운영환경에서 제대로 활용하려면 **Supervisor 설정을 큐별로 분리**하는 것이 가장 안정적인 구조다.
하나의 워커에 여러 큐를 섞는 것보다, 역할별로 리소스를 할당하고 모니터링하는 방식이 훨씬 유연하고 효율적이다.

> 큐는 줄세우기보다 분산이 핵심이다. 그리고 분산에는 Supervisor가 필수다.
