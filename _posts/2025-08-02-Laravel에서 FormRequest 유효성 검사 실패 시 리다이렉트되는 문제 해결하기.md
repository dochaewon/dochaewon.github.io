---
title: "Laravel에서 FormRequest 유효성 검사 실패 시 '/'로 리다이렉트되는 문제 해결하기"
date: 2025-08-04
categories: [Laravel, PHP]
tags: [laravel, validation, formrequest, api, 예외처리]
---

## 🚫 문제 상황: JSON 응답 대신 '/'로 리다이렉트됨

API 요청에서 유효성 검사(FormRequest)를 사용했는데, 다음과 같은 문제가 발생한 적이 있나요?

* 유효성 검사 실패 시 JSON 에러 응답 대신 `/`로 리다이렉트됨
* 응답 상태코드가 422가 아니라 302 (Redirect)
* Postman에서는 `text/html` 응답으로 바디 없이 돌아옴

> 이런 현상은 대부분 **헤더 설정, 라우팅, 예외 처리** 문제 때문입니다.

---

## 🧩 원인 및 해결법

### 1️⃣ Accept 헤더 누락

Laravel은 요청 헤더 중 `Accept` 값을 기준으로 응답 타입을 판단합니다.

📛 문제가 되는 경우:

```http
Accept: */*
```

✅ 반드시 아래처럼 설정하세요:

```http
Accept: application/json
```

이 설정을 하지 않으면 Laravel은 **웹 브라우저 요청으로 간주**하고 리다이렉트 처리합니다.

👉 Postman에서는 Headers 탭에서 `Accept`를 직접 추가하거나, 전역 설정하세요.

---

### 2️⃣ 웹 미들웨어 그룹 사용

Laravel은 기본적으로 `web.php` 라우트 파일에서 `web` 미들웨어 그룹을 사용합니다.
이 그룹은 **세션, 쿠키, CSRF**를 포함하며 **HTML 응답을 기본값으로 설정**합니다.

📛 문제가 되는 경우:

```php
Route::post('/api/register', RegisterController::class); // web.php
```

✅ 해결 방법:

* `routes/api.php`에 등록하거나,
* 명시적으로 `middleware('api')`를 지정:

```php
Route::middleware('api')->post('/register', RegisterController::class);
```

---

### 3️⃣ Laravel 11에서 예외 핸들러 누락

Laravel 11은 기본적으로 `app/Exceptions/Handler.php` 파일이 없습니다. 따라서 예외 발생 시 자동 JSON 응답이 되지 않습니다.

✅ 해결 방법:

* `App\Exceptions\CustomHandler`를 생성하고,
* `AppServiceProvider`에서 수동 등록하세요:

```php
// App\Providers\AppServiceProvider.php
use Illuminate\Contracts\Debug\ExceptionHandler;
use App\Exceptions\CustomHandler;

public function register(): void
{
    $this->app->singleton(ExceptionHandler::class, CustomHandler::class);
}
```

그리고 핸들러에서 다음처럼 유효성 예외 처리:

```php
public function render($request, Throwable $exception)
{
    if ($exception instanceof ValidationException) {
        return response()->json([
            'message' => '입력값이 유효하지 않습니다.',
            'errors' => $exception->errors(),
        ], 422);
    }

    return parent::render($request, $exception);
}
```

---

## ✅ 결과 확인

이제 FormRequest에서 실패하더라도 아래처럼 JSON으로 응답됩니다:

```json
{
  "message": "입력값이 유효하지 않습니다.",
  "errors": {
    "email": ["The email field is required."]
  }
}
```

---

## 🧵 마무리

Laravel로 API 서버를 만들 때, `FormRequest`를 사용할 경우 리다이렉트 대신 **정확한 JSON 에러 응답**을 받으려면 다음 3가지가 핵심입니다:

1. Accept: application/json 헤더 필수
2. 반드시 `api.php` 또는 `api` 미들웨어 사용
3. Laravel 11 이상은 예외 핸들러 수동 등록 필요

이 3가지만 기억하면, 유효성 검증 오류 때문에 `/`로 리다이렉트되는 당황스러운 상황은 더 이상 없습니다!
