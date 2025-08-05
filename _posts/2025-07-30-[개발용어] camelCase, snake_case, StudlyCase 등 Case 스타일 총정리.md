---
layout: post
title: "[개발용어] camelCase, snake_case, StudlyCase 등 Case 스타일 총정리"
date: 2025-07-30
categories: [Development, Naming]
tags: [naming, casing, 스타일가이드, camelCase, snake_case, studlyCase]
---

프로그래밍할 때 변수명, 함수명, 클래스명을 작성하면서 자주 듣게 되는 표현들이 있다.  
예: `camelCase`, `PascalCase`, `snake_case`, `kebab-case`, `StudlyCase` 등등...

이 글에서는 대표적인 **코딩 케이스 표기법**들을 간결하게 정리하고,  
언제 어떤 상황에 쓰이는지도 함께 설명한다.

---

## ✅ 1. camelCase (카멜 케이스)

```js
let userName = "Winnie";
function getUserList() { ... }
```

- 소문자로 시작하고, 이후 단어는 대문자로 시작
- 🐫 낙타등처럼 중간이 튀어나온 느낌에서 유래
- **자바스크립트, 자바, PHP 등의 변수명/함수명에 주로 사용**
- ✅ Laravel에서도 메소드, 속성명에 자주 사용

---

## ✅ 2. PascalCase (파스칼 케이스)

```php
class UserController { }
class AccessToken {}
```

- **각 단어의 첫 글자가 대문자**, 첫 단어도 대문자
- ✅ 클래스, 타입 정의 등에서 사용됨
- StudlyCase와 혼용되어 쓰이는 경우가 많음

---

## ✅ 3. StudlyCase (스터들리 케이스)

```php
UserProfile, AccessLog, BlogPost
```

- **PascalCase와 동일**한 스타일로 간주되는 경우가 많음
- 하지만 Laravel에서는 **특히 클래스, 이벤트, 리스너, Job 이름 등에 대해 StudlyCase라는 표현을 자주 씀**
- ✅ `Str::studly('user_profile')` → `UserProfile`
- ✅ Laravel `make:model`, `make:controller` 등의 아티즌 명령어에서 자동 생성

> 📌 정리: StudlyCase ≒ PascalCase

---

## ✅ 4. snake_case (스네이크 케이스)

```python
def get_user_name(): ...
user_profile_data = {}
```

- 모든 글자를 소문자로, 단어 구분은 언더스코어 `_`
- **Python, Ruby 등에서 함수명, 변수명으로 자주 사용**
- ✅ 데이터베이스 컬럼명에도 많이 사용됨 (ex: `created_at`)

---

## ✅ 5. kebab-case (케밥 케이스)

```html
<div class="main-container"></div>
```

- 단어를 소문자로 쓰고 `-`로 연결
- **HTML/CSS class 이름이나 URL 슬러그**에 주로 사용
- 예: `my-component.vue`, `api/v1/user-profile`

---

## ✅ 6. UPPER_SNAKE_CASE (대문자 스네이크)

```js
const MAX_LIMIT = 1000;
const API_KEY = "abcd1234";
```

- 상수(constant) 값 표현 시 주로 사용
- ✅ JavaScript, C, PHP에서 글로벌 상수에 자주 사용

---

## ✅ 7. SCREAMING-KEBAB-CASE (거의 안 씀)

```text
MY-VARIABLE-NAME
```

- 대문자와 `-` 조합
- 흔치 않지만 config 파일이나 구형 시스템에서 가끔 등장

---

## 🧠 언제 어떤 스타일을 써야 할까?

| 용도             | 권장 스타일     | 비고 |
|------------------|------------------|------|
| 변수명           | `camelCase`      | JS, PHP, Java |
| 함수/메서드명    | `camelCase`      | 대부분의 언어 |
| 클래스명         | `PascalCase` 또는 `StudlyCase` | 타입/클래스명 |
| 상수             | `UPPER_SNAKE_CASE` | `const` 변수 |
| DB 컬럼/테이블명 | `snake_case`     | 대부분의 SQL DB |
| HTML 클래스명    | `kebab-case`     | CSS |
| 파일/URL 슬러그  | `kebab-case`     | Vue, REST API 경로 등 |

---

## 🔁 Laravel 기준 정리

| 항목        | 예시               | 스타일       |
|-------------|--------------------|--------------|
| 클래스명    | `UserController`   | StudlyCase (Pascal) |
| 메소드명    | `getUserList()`    | camelCase |
| 속성명      | `$userName`        | camelCase |
| 컬럼명(DB)  | `created_at`       | snake_case |
| 마이그레이션 파일 | `2025_07_30_create_users_table.php` | snake_case + 날짜 prefix |
| URL 슬러그   | `/user-profile`    | kebab-case |

---

## ✍️ 결론

이름 짓기는 코드 품질과 유지보수성을 결정짓는 중요한 요소다.  
언어/프레임워크의 컨벤션에 맞는 스타일을 익혀두고 **일관성 있게 적용하는 것**이 무엇보다 중요하다.
