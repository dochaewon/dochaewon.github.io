---
title: "Laravel 파사드(Facade)란? 서비스 컨테이너와의 관계까지 제대로 정리해보기"
date: 2025-07-23
tags: [Laravel, Facade, LaravelFacade, ServiceContainer, PHP]
---

## 🎯 들어가며

Laravel을 쓰다 보면 자주 마주치는 코드가 있다.

```php
Cache::put('key', 'value', 300);
Log::info('Something happened!');
```

이런 문법을 처음 보면 "정적(static) 클래스인가?"라고 생각하기 쉽다.  
하지만 실제로는 Laravel의 **파사드(Facade)** 기능이 숨어 있다.

이번 글에서는 **Laravel 파사드의 개념, 내부 동작 원리, 서비스 컨테이너와의 관계, 그리고 실전 사용 팁**까지 정리해본다.

---

## 🧱 파사드란?

Laravel 공식 문서의 정의는 다음과 같다:

> “파사드는 서비스 컨테이너에 등록된 클래스를 정적 인터페이스처럼 사용할 수 있게 해주는 클래스”

즉, 실제 객체는 동적으로 주입되지만, 우리는 마치 static 메서드를 쓰듯 간편하게 사용할 수 있다.

---

## ⚙️ 내부 동작 원리

예를 들어 `Cache::put()`을 호출하면 실제로는 다음과 같은 흐름이 일어난다:

1. `Cache`는 `Illuminate\Support\Facades\Cache` 클래스이다.
2. 이 클래스는 `Facade` 추상 클래스를 상속받는다.
3. 내부적으로 `getFacadeAccessor()`를 통해 서비스 컨테이너의 키를 가져온다.

```php
// Illuminate\Support\Facades\Cache
protected static function getFacadeAccessor()
{
    return 'cache';  // 서비스 컨테이너의 key
}
```

4. Laravel은 `app('cache')`로 실제 구현체를 resolve 한다.
5. 즉, 결국엔 `app()->make('cache')->put(...)`으로 실행된다.

---

## 🧪 직접 구현해보기

```php
// app/Services/MessageService.php
class MessageService {
    public function hello() {
        return "Hello from service!";
    }
}
```

```php
// app/Facades/MessageFacade.php
use Illuminate\Support\Facades\Facade;

class MessageFacade extends Facade {
    protected static function getFacadeAccessor() {
        return 'message';
    }
}
```

```php
// AppServiceProvider.php
public function register()
{
    $this->app->singleton('message', function () {
        return new \App\Services\MessageService();
    });
}
```

이제 어디서든 `MessageFacade::hello();`로 호출 가능하다.

---

## 💡 파사드 vs 의존성 주입 비교

| 항목 | 파사드 | 의존성 주입 |
|------|--------|----------------|
| 문법 | 정적처럼 보임 | 일반 클래스 주입 |
| 테스트 | Mock 힘듦 (별도 처리 필요) | Mocking 쉬움 |
| 가독성 | 매우 좋음 | 상황에 따라 다름 |
| 유연성 | 낮음 | 높음 |
| 추천 용도 | 로깅, 캐시, 큐 등 Laravel 시스템 서비스 | 도메인 서비스, 유닛 테스트 대상 |

---

## ✅ 언제 파사드를 쓰면 좋은가?

- Laravel 내부 기능을 간결하게 호출할 때
- 반복 호출이 많고 간단한 로직일 때 (ex: `Log`, `Cache`, `Queue`, `Route`)
- 별도의 인스턴스를 관리하지 않아도 될 때

---

## 🔥 주의할 점

- 너무 남용하면 **정적 의존**이 강해져 테스트가 어려워진다
- `Mockery::mock('alias')` 또는 `Facades::shouldReceive()` 등을 사용해야 한다
- 도메인 서비스나 상태를 갖는 서비스에는 **의존성 주입이 더 적합**

---

## ✍️ 마무리

Laravel의 파사드는 **정적처럼 보이지만 동적이다.**  
서비스 컨테이너와 긴밀하게 연결되어 있으며, 내부적으로는 의존성 주입의 한 방식이다.  
적절히 활용하면 코드가 간결해지고, Laravel의 아름다움을 제대로 느낄 수 있다.

---
