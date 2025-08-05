---
title: "Laravel 11에서 authorize() 사용 시 'This action is unauthorized.' 해결하기"
date: 2025-08-01
categories: [Laravel, PHP]
tags: [laravel11, authorize, policy, 예외처리, 권한]
---

## 🧠 Policy란 무엇인가요?

Laravel에서 **Policy(정책)** 는 인증된 사용자가 특정 리소스(Post, User, Order 등)에 대해 **행동할 권한이 있는지 판단**하는 역할을 합니다.

쉽게 말하면, 컨트롤러에서는 `"이 사용자가 이 글을 수정해도 되는가?"` 같은 질문을 Policy에게 던지고, Policy는 `true/false`로 답합니다.

예를 들어 블로그 포스트의 작성자가 본인인지 확인하려면:

```php
public function update(User $user, Post $post): bool
{
    return $user->id === $post->user_id;
}
```

위 메서드를 Policy 클래스에 정의해두고, 컨트롤러에서는 이렇게 씁니다:

```php
$this->authorize('update', $post);
```

→ 이렇게 하면 Laravel이 내부적으로 `PostPolicy::update()`를 호출하여 권한을 판단합니다.

---

## ❓ 문제 상황

Laravel 11에서 다음과 같은 코드를 작성한 뒤, `authorize()`를 호출했을 때 아래와 같은 오류 메시지를 받았던 경험이 있으신가요?

```php
$this->authorize('update', $post);
```

```json
{
  "message": "This action is unauthorized."
}
```

이 에러는 **Policy가 연결되지 않았거나, 등록이 누락된 경우**에 자주 발생합니다.

---

## 🧩 원인 분석

1. **Policy 클래스가 없거나 연결되지 않음**
2. `AuthServiceProvider`에서 Policy 등록이 누락됨
3. `Post` 모델과 Policy 네이밍이 맞지 않음
4. `authorize()` 전에 `$this->authorizeResource()` 또는 `Gate::policy()` 설정 안함

---

## ✅ 해결 순서

### 1. Policy 생성

```bash
php artisan make:policy PostPolicy --model=Post
```

이렇게 하면 `App\Policies\PostPolicy`가 생성되고 `Post` 모델과 자동 연결됩니다.

### 2. AuthServiceProvider에서 등록

`app/Providers/AuthServiceProvider.php`에서 다음을 추가합니다:

```php
use App\Models\Post;
use App\Policies\PostPolicy;

protected $policies = [
    Post::class => PostPolicy::class,
];
```

Laravel 11에서도 이 방식은 여전히 유효합니다.

### 3. authorize() 사용 시 메서드 존재 확인

```php
// 컨트롤러 예시
public function update(Post $post)
{
    $this->authorize('update', $post); // PostPolicy::update() 메서드가 반드시 존재해야 함
}
```

### 4. Policy 메서드 예시

```php
public function update(User $user, Post $post): bool
{
    return $user->id === $post->user_id;
}
```

---

## 🚨 authorize() vs can()

* `$this->authorize(...)` → 실패 시 `AuthorizationException` 발생
* `Gate::allows(...)` 또는 `$user->can(...)` → `true/false` 반환

API라면 `authorize()`의 실패 응답을 예쁘게 처리하기 위해 커스텀 핸들러도 필요할 수 있습니다.

---

## 🧪 실전 테스트

1. PostPolicy 생성
2. update() 메서드 구현
3. AuthServiceProvider에 등록
4. 컨트롤러에서 `$this->authorize('update', $post);` 호출

정상 동작하면 권한이 없는 경우 아래와 같이 403 JSON 응답이 반환됩니다:

```json
{
  "message": "이 작업은 권한이 없습니다."
}
```

---

## 🧵 마무리하며

Laravel의 `authorize()`는 Policy를 기반으로 강력한 권한 관리를 가능하게 해줍니다. 하지만 **정확한 연결과 선언이 없으면 무조건 실패**하기 때문에 다음을 항상 체크하세요:

* ✅ Policy 클래스가 생성되었는가?
* ✅ AuthServiceProvider에 등록되었는가?
* ✅ authorize()에서 지정한 메서드가 존재하는가?
* ✅ 모델 ↔ Policy 간의 연결이 올바른가?

---

## 📚 관련 키워드

`#laravel11` `#policy` `#authorize` `#authorizationexception` `#권한관리`
