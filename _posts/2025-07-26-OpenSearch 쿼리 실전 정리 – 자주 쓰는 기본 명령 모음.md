---
layout: post
title: "OpenSearch 쿼리 실전 정리 – 자주 쓰는 기본 명령 모음"
date: 2025-07-26
categories: [opensearch, search]
tags: [opensearch, 쿼리, 실무팁, 검색엔진, 데이터삭제, kibana]
---

OpenSearch를 실무에서 사용하다 보면 **자주 쓰는 쿼리**들이 있다.  
특히 데이터를 직접 조회하거나 수정, 삭제하는 기본 쿼리는 처음 접하면 낯설지만,  
한 번 익혀두면 Kibana 콘솔이나 API 호출 시 매우 유용하다.

최근 나는 아래와 같은 쿼리를 처음 사용하게 됐다:

---

## 🗑️ 문서 삭제 – `DELETE /index/_doc/id`

```http
DELETE /dev_v1_car_option_descriptions/_doc/1138_5229
```

- 특정 문서 ID로 바로 삭제할 수 있음
- `_doc`은 타입이며, OpenSearch 7.x 이후는 사실상 의미 없음
- Kibana Dev Tools에서 바로 실행 가능

```bash
# curl로 호출 시 예시
curl -X DELETE "https://your-domain/index/_doc/1138_5229" -u user:pass
```

---

## 🔍 문서 조회 – `GET /index/_doc/id`

```http
GET /dev_v1_car_option_descriptions/_doc/1138_5229
```

- 삭제 전에 존재 여부 확인할 때 자주 사용
- `_source` 안에 실제 데이터가 있음

---

## 🔎 조건 검색 – `POST /index/_search` + query DSL

```json
POST /dev_v1_car_option_descriptions/_search
{
  "query": {
    "match": {
      "car_full_name": "쏘렌토"
    }
  }
}
```

- `match`, `term`, `bool` 조합은 실무에서 매우 자주 사용
- 검색 결과의 `_id`를 활용해 수정/삭제 가능

---

## ✏️ 문서 수정 – `POST /index/_update/id`

```http
POST /dev_v1_car_option_descriptions/_update/1138_5229
{
  "doc": {
    "is_like": true
  }
}
```

- 기존 문서의 일부 필드만 갱신
- `doc_as_upsert: true`를 넣으면 없을 시 insert도 가능

---

## 🧹 전체 삭제 (주의!) – `DELETE /index`

```http
DELETE /dev_v1_car_option_descriptions
```

- 인덱스 전체를 날려버리므로 **절대 조심**
- 실수 방지를 위해 Kibana 콘솔에서는 보안 설정이 막혀 있을 수도 있음

---

## 🛠 기타 자주 쓰는 툴

| 목적 | 쿼리 |
|------|------|
| 문서 수 카운트 | `GET /index/_count` |
| 인덱스 매핑 보기 | `GET /index/_mapping` |
| 인덱스 샘플 조회 | `GET /index/_search?size=1` |

---

## ✅ 정리

OpenSearch 쿼리는 기본만 익혀도 대부분의 검색, 수정, 삭제 작업을 커버할 수 있다.  
특히 Kibana Dev Tools나 API 테스트 환경에서 빠르게 써보면서 익히는 게 가장 좋다.

내가 실무에서 가장 자주 쓰는 건 아래 세 가지:

- `GET /index/_doc/id` → 존재 확인
- `POST /index/_update/id` → 필드 일부만 수정
- `DELETE /index/_doc/id` → 단일 문서 삭제

이제부터는 매번 검색하지 않아도 된다 🙂

---

## 🏷️ 태그
#OpenSearch #쿼리기초 #문서삭제 #Kibana #DevTools #검색엔진 #ElasticSearch #API활용 #인덱스조회 #실무팁
