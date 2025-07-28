---
layout: post
title: "OpenSearch란? Kibana랑 뭐가 다를까? 간단 개요 정리"
date: 2024-07-29
categories: [opensearch, devtools]
tags: [opensearch, kibana, 검색엔진, 대시보드, 실무정리]
---

처음 OpenSearch를 접하면 누구나 이렇게 생각한다:

> "Kibana랑 되게 비슷한데? 이건 뭐고 뭘 쓰면 되지?"

나 역시 처음엔 헷갈렸고, 기존에 Elasticsearch + Kibana를 쓰던 프로젝트에서 OpenSearch로 넘어오며 조금씩 정리가 되었다. 이 글에서는 **OpenSearch의 개요**, **Kibana와의 차이**, **뭘 쓰면 되는지**를 간단하게 정리해본다.

---

## 🔍 OpenSearch란?

- **Elasticsearch 오픈소스 버전에서 갈라져 나온 검색엔진**
- 아마존이 주도 (AWS OpenSearch Service와도 연결)
- **문서 기반 검색, 실시간 인덱싱, 로그 분석 등에 최적화**
- 완전한 오픈소스 (Apache 2.0 라이선스)

> Elasticsearch 7.10 이후부터 상용 라이선스로 바뀌면서,  
> 오픈소스 진영에서 분기되어 나온 것이 OpenSearch다.

---

## 📊 OpenSearch Dashboards (예전 Kibana)

- Elasticsearch의 시각화 도구인 Kibana를 오픈소스 기반으로 분기한 것
- 이름만 바뀌었지 구조나 기능은 유사함
- 흔히 **Kibana → OpenSearch Dashboards**로 바뀌었다고 보면 된다

### 대표 기능:
- 인덱스 시각화 (테이블, 차트, 맵)
- Discover로 로그/데이터 검색
- Dev Tools에서 쿼리 실행 (POST /_search 등)
- 대시보드 & Alert 관리

---

## 🆚 Kibana와 OpenSearch Dashboards 차이

| 항목 | Kibana | OpenSearch Dashboards |
|------|--------|------------------------|
| 소속 | Elastic 사 | 오픈소스 커뮤니티 (주로 AWS) |
| 라이선스 | Elastic License (비오픈) | Apache 2.0 (완전 오픈소스) |
| 기본 연결 | Elasticsearch | OpenSearch |
| 상용 서비스 | Elastic Cloud | AWS OpenSearch Service |

> **기능은 거의 유사하지만, 상용화 제한 여부와 라이선스가 가장 큰 차이**다.

---

## 🛠️ 뭘 쓰면 될까?

- **기존 Elasticsearch + Kibana 사용자** → OpenSearch + Dashboards로 넘어와도 거의 무리 없음
- **AWS 기반 로그 분석, 검색엔진 구축** → OpenSearch 쓰는 게 비용, 통합 관리 측면에서 더 유리
- 완전한 오픈소스만 써야 하는 기업/프로젝트 → OpenSearch 선택이 안정적

---

## ✅ 정리

- OpenSearch는 Elasticsearch 기반 오픈소스 검색엔진
- Kibana는 이제 OpenSearch Dashboards로 이어졌다고 보면 됨
- 쿼리 구조, 시각화 방식, 인덱싱 등 거의 동일
- 다만 **Elastic Stack은 상용**, OpenSearch는 **완전 오픈소스**

실무에서 Kibana 써본 경험이 있다면 OpenSearch는 매우 쉽게 적응 가능하다.  
나는 지금 대부분 OpenSearch + Dashboards 조합으로 사용 중이다.

---

## 🏷️ 태그
#OpenSearch #Kibana #Elasticsearch #검색엔진 #오픈소스 #시각화도구 #대시보드 #AWSOpenSearch #실무팁
