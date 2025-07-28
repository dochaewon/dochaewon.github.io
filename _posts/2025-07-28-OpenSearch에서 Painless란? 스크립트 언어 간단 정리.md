---
layout: post
title: "OpenSearch에서 Painless란? 스크립트 언어 간단 정리"
date: 2025-07-28
categories: [opensearch, scripting]
tags: [painless, opensearch, elasticsearch, 스크립트, 검색엔진, 실무팁]
---

OpenSearch에서 쿼리를 작성하다 보면 다음과 같은 코드를 자주 보게 된다:

```json
"script": {
  "lang": "painless",
  "source": "doc['price'].value > 10000"
}
```

여기서 나오는 `"lang": "painless"`는 대체 뭘까?

처음 보면 생소할 수 있지만, OpenSearch(또는 Elasticsearch)의 **기본 내장 스크립트 언어**이다.

---

## 🧠 Painless란?

- OpenSearch와 Elasticsearch에서 사용하는 **스크립트 언어**
- **빠르고 안전하게 실행되도록 설계**된 Java 기반 DSL
- 필드 계산, 조건 필터링, 스크립트 정렬 등에 사용됨
- 이름처럼 “고통 없이(painless)” 쓰라는 의미로 만들어짐

> Elasticsearch 5.x 버전부터 기본 언어로 채택되어 OpenSearch에도 그대로 이어짐

---

## 🔍 언제 쓰이나?

| 용도 | 예시 |
|------|------|
| 조건부 필터 | price > 10000 인 문서만 필터링 |
| 커스텀 스코어 계산 | 조회수와 좋아요 수를 가중치로 계산 |
| 집계 후 계산 | 평균값 기준 비교, 비율 계산 등 |
| 스크립트 기반 정렬 | 특정 조건을 반영한 정렬 로직 구현 |

---

## 🧪 예시 쿼리들

### 1. 조건부 필터
```json
{
  "query": {
    "bool": {
      "filter": {
        "script": {
          "script": {
            "source": "doc['price'].value > 10000",
            "lang": "painless"
          }
        }
      }
    }
  }
}
```

### 2. 문서 점수 커스터마이징
```json
{
  "query": {
    "function_score": {
      "script_score": {
        "script": {
          "source": "doc['view_count'].value * 0.7 + doc['like_count'].value * 1.5",
          "lang": "painless"
        }
      }
    }
  }
}
```

---

## ⚠️ 주의사항

- `doc['field']` 접근만 가능 (`_source` 사용 ❌)
- 타입 체크 안 하면 null/에러 발생 가능
- 복잡한 계산 로직은 성능 저하를 유발할 수 있음
- 가능한 한 **집계나 매핑 레벨에서 처리 후 사용하는 게 더 안전**

---

## ✅ 정리

- `painless`는 OpenSearch의 내장 스크립트 언어로,  
  조건 필터, 계산, 정렬, 점수 조정 등 다양하게 활용된다.
- 기본적으로 안전하고 빠르지만, **무분별하게 사용 시 성능 저하 가능**하므로  
  가볍고 명확한 계산에만 사용하는 것이 좋다.

> 실무에서는 특히 `script_score`, `bucket_script`, `scripted_metric` 등과 함께 자주 쓰인다.

---

## 🏷️ 태그
#OpenSearch #Painless #Elasticsearch #스크립트 #검색엔진 #쿼리 #DevTools #실무팁 #FunctionScore #ScriptScore
