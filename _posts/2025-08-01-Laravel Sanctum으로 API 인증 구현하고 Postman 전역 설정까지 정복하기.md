---
title: "Laravel Sanctum으로 API 인증 구현하고 Postman 전역 설정까지 정복하기"
date: 2025-08-04
categories: [Laravel, PHP]
tags: [laravel, sanctum, postman, api, 인증]
---

## 🔰 Sanctum은 무엇인가요?

**Sanctum**은 라라벨(Laravel)에서 제공하는 인증 시스템 중 하나입니다. 한국어로는 **"생텀"**, 혹은 일부 커뮤니티에서는 **"샹텀"**, "쌍텀" 등으로 발음되지만, 공식적으로는 \[/ˈsæŋktəm/]으로 영어 "생텀"에 가깝습니다.

Sanctum은 다음과 같은 시나리오를 위해 설계되었습니다:

1. **SPA (Single Page Application)** - Vue, React 등 브라우저 기반 앱과 백엔드 간 쿠키 세션 인증
2. **API 토큰 인증** - 외부 앱, 모바일, Postman 등에서 Bearer Token으로 인증
3. **기계 간 API 통신** - 서버 간 요청 등에서 간단한 토큰 인증 방식 제공

이 글에서는 **API 토큰 인증 방식**을 중심으로, Laravel Sanctum의 작동 원리부터 Postman 자동 설정까지 **현업에 바로 적용 가능한 실전 흐름**을 다룹니다.

---

## 🔐 Laravel Sanctum 인증 흐름 이해하기

Laravel Sanctum은 `HasApiTokens` 트레이트를 통해 `createToken()` 메서드로 Personal Access Token을 발급하고, `auth:sanctum` 미들웨어로 토큰을 인증합니다. 내부적으로는 `personal_access_tokens` 테이블을 사용하지만, 프로젝트 상황에 따라 직접 토큰을 생성해 사용하는 커스텀 방식도 가능하죠.

### 🌀 Sanctum 인증 흐름 요약 (PAT 테이블 사용 기준)

1. 클라이언트가 `/login` API에 email/password로 로그인 요청
2. 서버는 유효성 확인 후 `createToken()`으로 토큰 발급
3. 클라이언트는 이후 요청마다 Authorization 헤더에 `Bearer {token}` 포함
4. Laravel은 해당 토큰을 `personal_access_tokens`에서 검증
5. 유효한 토큰일 경우 인증된 사용자로 API 처리

### ⚠️ PAT 테이블 없이 사용하는 경우는?

만약 `php artisan migrate`를 통해 `personal_access_tokens` 테이블을 생성하지 않았다면:

* `createToken()` 호출 시 Laravel 내부적으로 오류가 발생하거나, 토큰이 발급되지 않음
* 이 경우 `users` 테이블에 `api_token` 컬럼을 추가해 커스텀 방식으로 토큰을 관리할 수 있습니다

```php
// 커스텀 토큰 생성 예시
$token = base64_encode(Str::random(40));
$user->api_token = $token;
$user->save();
```

→ 이후 미들웨어 또는 컨트롤러에서 `where('api_token', $request->bearerToken())`로 직접 인증합니다.

> ✅ 이 방식은 Laravel Sanctum의 정식 방식은 아니며, 테스트용 혹은 간단한 API 서버에서만 권장됩니다.

---

## 🧱 Sanctum 설치 및 구성

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate # personal_access_tokens 테이블 생성됨
```

### Kernel.php 설정

`app/Http/Kernel.php` → `api` 미들웨어 그룹에 추가

```php
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

→ 위 미들웨어는 **SPA 방식**에 필요하며, API 토큰 인증에는 생략 가능

---

## 🔑 로그인 및 토큰 발급 API 만들기

```php
Route::post('/login', function (Request $request) {
    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        return response()->json(['message' => '인증 실패'], 401);
    }

    return response()->json([
        'token' => $user->createToken('api-token')->plainTextToken,
    ]);
});
```

---

## 🧪 Postman에서 Bearer 토큰 자동 설정하기

### 1. 토큰을 환경변수로 저장

* Postman 좌측 상단 Environment 생성
* `token` 변수 추가하고 로그인 응답에서 받은 토큰 붙여넣기

### 2. Pre-request Script에 자동 헤더 삽입

```js
pm.request.headers.add({
  key: "Authorization",
  value: "Bearer {{token}}"
});
```

→ 모든 요청마다 자동으로 Bearer 토큰이 삽입됩니다

### 3. 테스트 요청 전 흐름 요약

1. `/login` 호출 후 응답에서 토큰 복사 → `token` 변수에 저장
2. 이후 API 호출은 모두 자동 인증

---

## 🔓 로그아웃 처리 및 토큰 삭제

```php
Route::post('/logout', function (Request $request) {
    $request->user()->currentAccessToken()->delete();
    return response()->json(['message' => '로그아웃 완료']);
});
```

### 모든 토큰 삭제 (전체 기기 로그아웃)

```php
$request->user()->tokens()->delete();
```

---

## ⏰ 토큰 만료 직접 구현하기

Sanctum은 기본적으로 토큰에 만료 시간이 없습니다. 직접 추가하려면:

1. `personal_access_tokens` 테이블에 `expires_at` 컬럼 추가
2. 토큰 생성 시 만료 시간 지정:

```php
$user->createToken('api-token', [], now()->addHours(1));
```

3. 커스텀 미들웨어에서 `expires_at` 체크 로직 구현

---

## 🧬 SPA + Sanctum 사용 시 흐름 요약 (쿠키 세션 기반)

```js
axios.get('/sanctum/csrf-cookie').then(() => {
  axios.post('/login', { email, password })
})
```

* Laravel은 `/sanctum/csrf-cookie` 요청에 `XSRF-TOKEN` 쿠키를 내려줌
* 이후 요청에는 `X-XSRF-TOKEN` 헤더 포함해야 로그인 성공
* `EnsureFrontendRequestsAreStateful` 미들웨어 필수

> ✅ 이 방식은 API 토큰 인증과는 별개이며, **SPA (웹 앱) 환경에서만 사용됩니다**

---

## ✅ 인증 방식 정리표

| 방식        | 방식 설명        | 주요 특징                              | 적합한 경우       |
| --------- | ------------ | ---------------------------------- | ------------ |
| API Token | Bearer 토큰    | `createToken()`, `auth:sanctum` 필요 | 모바일/외부 클라이언트 |
| SPA 세션    | 쿠키 + CSRF    | `csrf-cookie`, 미들웨어 필요             | Vue/React 웹앱 |
| 커스텀       | 사용자 지정 토큰 저장 | `users.api_token`, 미들웨어 직접 구현      | 테스트, 프로토타입   |

---

## 🧵 마무리

Sanctum은 라라벨에서 API 인증을 가장 쉽게 구현할 수 있는 방법입니다. 특히 Postman 테스트 환경이나 모바일 백엔드에서 매우 유용하게 활용되며, SPA와의 연동도 지원합니다.

이번 글에서는 다음 내용을 실전 중심으로 정리했습니다:

* Sanctum의 개념과 사용 시나리오
* PAT 테이블 없이 직접 토큰 구현 시 유의사항
* Postman에서 Bearer 토큰 자동 설정법
* 로그아웃 및 토큰 만료 처리
* SPA 연동 시 흐름

이제 Laravel에서 어떤 인증이 필요한지에 따라 적절한 방식만 골라 구현하시면 됩니다 💡
