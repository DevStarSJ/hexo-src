---
title: Oracle hint
date: 2016-02-26 02:00:00
categories:
- Database
- Oracle
tags:
- Database
- Oracle
- SQLP
---

* `LEADING(테이블)` : 해당 Table 부터 읽음
* `QB_NAME(이름)` : 해당 query block에 이름을 지어준다. 그래서 다른 곳에서 이름@테이블명 으로 해당 테이블을 지정 할 수 있다.
* JOIN 방법
  - `USE_NL(테이블)` : outer table을 테이블(inner table)과 Nested Loop Join 방식으로 JOIN을 시도
* Semi Join : 1 row만 JOIN에 성공하면 더이상 inner쪽을 보지 않고 outer의 다음 row를 찾음
  - `NL_SJ`, `HASH_SJ`, `MERGE_SJ`
* Anti Join : 일치하지 않는 data를 추출 (NOT IN, NOT EXISTS, MINUS)
  - `NL_AJ`, `HASH_AJ`, `MERGE_AJ`
* Subquery Unnesting 관련
  - `UNNEST` : 풀어서 JOIN 방식으로 유도
  - `NO_UNNEST` : 풀지말고 Filter 방식으로 최적화 유도
* View Merging
  - `NO_MERGE(테이블)` : main query 와 inline view를 JOIN으로 풀지말고 inline view를 먼저 실행
  - `MERGE(테이블)` : main query와 inline view를 JOIN으로 풀어서 최적화를 시도
* `PUSH_PRED(인라인뷰)` : (Push Predicate, 조건절 Push) main query에서 먼저 filtering하여 그 결과를 inline view의 filter 조건으로 넣어라.
* `PUSH_SUBQ` : (Push Subquery) 실행계획상 가능한 앞 단계에서 subquery filtering을 하여 main query의 범위를 줄이고자 할때, NO_UNNEST와 같이 사용되어야 함
* Concatenation (주로 OR절 처리)
  - `USE_CONCAT` : UNION ALL로 표현
  - `NO_EXPAND` : 나누지 말고 그대로 실행
* `FULL(테이블)` : 해당 테이블을 full scan 한다.
* `PARALLEL(테이블 프로세스수)` : 해당 테이블을 명시한 프로레스수 만큼 병렬로 처리한다.
* `APPEND` : INSERT를 APPEND 모드로 수행
