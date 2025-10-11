---
title: "[데이터베이스] 문제 해설 #13 - 데이터 저장 구조"
date: 2025-01-25 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [data-storage, file-organization, buffer-management, free-space-management, table-partitioning, multitable-clustering]
pin: false
math: false
mermaid: false
---

## 개요

이 포스트는 **데이터 저장 구조(Data Storage Structures)** 연습 문제 해설입니다. 데이터베이스 시스템이 물리적으로 데이터를 저장하고 관리하는 방법, 파일 조직, 레코드 구조, 버퍼 관리, 그리고 저장 공간 최적화 기법을 다룹니다.

**다루는 주제:**
- 레코드 삭제 기법 비교
- 자유 공간 관리 (Free Space Management)
- 다중 테이블 클러스터링 (Multitable Clustering)
- 비트맵 기반 자유 공간 맵
- 버퍼 관리와 해시 테이블
- 테이블 파티셔닝 (Table Partitioning)
- 버퍼 교체 정책 (LRU vs MRU)
- PostgreSQL의 버퍼 관리 전략

---

## 문제 13.1: 레코드 삭제 기법 비교

### 문제
Figure 13.3의 파일에서 레코드 5를 삭제한다고 가정합니다. 삭제를 구현하는 다음 기법들의 상대적 장단점을 비교하시오:

**(a)** 레코드 6을 레코드 5의 공간으로 이동하고, 레코드 7을 레코드 6의 공간으로 이동
**(b)** 레코드 7을 레코드 5의 공간으로 이동
**(c)** 레코드 5를 삭제됨으로 표시하고, 레코드를 이동하지 않음

### 해답

#### (a) 순차적 이동 (Sequential Move)

**방법:**
```
초기: [레코드1][레코드2][레코드3][레코드4][레코드5][레코드6][레코드7]
삭제 후: [레코드1][레코드2][레코드3][레코드4][레코드6][레코드7][빈공간]
```

**장점:**
- 가장 직관적인 접근법
- 파일의 순서 유지
- 연속적인 빈 공간 확보 (끝에 모임)
- 단편화 없음

**단점:**
- **가장 많은 레코드 이동** 필요
- **가장 많은 디스크 접근** 발생
- 시간 복잡도: O(n) - n은 삭제 지점 이후 레코드 수
- 대량 삭제 시 성능 저하 심각

**적용 시나리오:**
- 파일이 작고 삭제가 드문 경우
- 순차 접근이 중요한 경우
- 순서가 중요한 애플리케이션 (로그 파일 등)

#### (b) 마지막 레코드 이동 (Last Record Move)

**방법:**
```
초기: [레코드1][레코드2][레코드3][레코드4][레코드5][레코드6][레코드7]
삭제 후: [레코드1][레코드2][레코드3][레코드4][레코드7][레코드6][빈공간]
```

**장점:**
- 더 적은 레코드 이동 (1개만)
- 더 적은 디스크 접근
- 시간 복잡도: O(1)
- 빠른 삭제 연산

**단점:**
- **파일의 순서가 파괴됨**
- 인덱스 업데이트 필요 (레코드 7의 위치 변경)
- 순차 스캔 성능 저하 가능
- 논리적 순서와 물리적 순서 불일치

**적용 시나리오:**
- 순서가 중요하지 않은 경우
- 인덱스를 통한 접근이 주된 경우
- 빠른 삭제가 필요한 경우

#### (c) 논리적 삭제 (Logical Deletion)

**방법:**
```
초기: [레코드1][레코드2][레코드3][레코드4][레코드5][레코드6][레코드7]
삭제 후: [레코드1][레코드2][레코드3][레코드4][DELETED][레코드6][레코드7]
```

**구현:**
- 레코드에 삭제 플래그 추가
- 또는 별도의 비트맵 사용

**장점:**
- **순서 보존**
- **레코드 이동 없음** (I/O 최소화)
- 시간 복잡도: O(1)
- 삭제 취소 가능 (복구 용이)
- 동시성 제어 간단

**단점:**
- **자유 공간 추적 오버헤드**
- 파일 내 "구멍(holes)" 증가
- 시간이 지나면 단편화 심각
- 주기적인 압축(compaction) 필요
- 연속된 자유 공간 확보 어려움
- 스캔 시 삭제된 레코드도 검사 필요

**적용 시나리오:**
- 삭제가 빈번한 경우
- 트랜잭션 복구가 필요한 경우
- 동시 접근이 많은 경우

#### 실무 권장사항

**대부분의 상용 DBMS 선택: (c) + 주기적 압축**

**하이브리드 접근:**
```
1. 일반 삭제: 논리적 삭제 사용
2. 단편화 임계값 도달 시: 압축 수행
3. 압축 중: 시스템 부하가 낮을 때 실행
```

**성능 비교:**

| 기법 | 삭제 시간 | I/O 비용 | 순서 유지 | 단편화 | 적합한 경우 |
|------|----------|---------|----------|--------|------------|
| (a) 순차 이동 | O(n) | 높음 | ✓ | 없음 | 작은 파일, 드문 삭제 |
| (b) 마지막 이동 | O(1) | 낮음 | ✗ | 없음 | 순서 무관, 빠른 삭제 |
| (c) 논리 삭제 | O(1) | 최소 | ✓ | 있음 | 빈번한 삭제, 복구 필요 |

---

## 문제 13.2: 자유 리스트를 사용한 레코드 관리

### 문제
Figure 13.4의 파일 구조에서 다음 단계를 거친 후의 구조를 보이시오:

**(a)** Insert (24556, Turnamian, Finance, 98000)
**(b)** Delete record 2
**(c)** Insert (34556, Thompson, Music, 67000)

### 초기 상태 분석

**Figure 13.4 구조 (가정):**
```
header → 4
record 0: 10101, Srinivasan, Comp. Sci., 65000
record 1: (사용 중)
record 2: 15151, Mozart, Music, 40000
record 3: 22222, Einstein, Physics, 95000
record 4: → 6 (자유 리스트)
record 5: 33456, Gold, Physics, 87000
record 6: (자유)
record 7: 58583, Califieri, History, 62000
record 8: 76543, Singh, Finance, 80000
record 9: 76766, Crick, Biology, 72000
record 10: 83821, Brandt, Comp. Sci., 92000
record 11: 98345, Kim, Elec. Eng., 80000
```

**자유 리스트 구조:**
- header → 4 → 6 → NULL
- "→ i"는 레코드 i를 가리키는 포인터

### (a) Insert (24556, Turnamian, Finance, 98000)

**과정:**
1. 헤더가 가리키는 첫 자유 레코드 찾기: record 4
2. record 4에 새 데이터 삽입
3. 헤더를 다음 자유 레코드로 업데이트: header → 6

**결과:**

| 위치 | ID | Name | Department | Salary |
|------|-----|------|------------|--------|
| header | → 4 | | | |
| record 0 | 10101 | Srinivasan | Comp. Sci. | 65000 |
| record 1 | 24556 | Turnamian | Finance | 98000 |
| record 2 | 15151 | Mozart | Music | 40000 |
| record 3 | 22222 | Einstein | Physics | 95000 |
| record 4 | → 6 | | | |
| record 5 | 33456 | Gold | Physics | 87000 |
| record 6 | | | | |
| record 7 | 58583 | Califieri | History | 62000 |
| record 8 | 76543 | Singh | Finance | 80000 |
| record 9 | 76766 | Crick | Biology | 72000 |
| record 10 | 83821 | Brandt | Comp. Sci. | 92000 |
| record 11 | 98345 | Kim | Elec. Eng. | 80000 |

**Figure 13.101** - 삽입 후 상태

**자유 리스트:** header → 4 → 6 → NULL

### (b) Delete record 2

**과정:**
1. record 2의 데이터 제거
2. record 2를 자유 리스트의 앞에 추가
3. 기존 헤더 값을 record 2에 저장
4. 헤더를 record 2로 업데이트

**결과:**

| 위치 | ID | Name | Department | Salary |
|------|-----|------|------------|--------|
| header | → 2 | | | |
| record 0 | 10101 | Srinivasan | Comp. Sci. | 65000 |
| record 1 | 24556 | Turnamian | Finance | 98000 |
| record 2 | → 4 | | | |
| record 3 | 22222 | Einstein | Physics | 95000 |
| record 4 | → 6 | | | |
| record 5 | 33456 | Gold | Physics | 87000 |
| record 6 | | | | |
| record 7 | 58583 | Califieri | History | 62000 |
| record 8 | 76543 | Singh | Finance | 80000 |
| record 9 | 76766 | Crick | Biology | 72000 |
| record 10 | 83821 | Brandt | Comp. Sci. | 92000 |
| record 11 | 98345 | Kim | Elec. Eng. | 80000 |

**Figure 13.102** - 삭제 후 상태

**자유 리스트:** header → 2 → 4 → 6 → NULL

**참고:** 자유 리스트는 다른 순서도 가능 (예: header → 4 → 2 → 6)

### (c) Insert (34556, Thompson, Music, 67000)

**과정:**
1. 헤더가 가리키는 첫 자유 레코드 찾기: record 2
2. record 2에 새 데이터 삽입
3. 헤더를 다음 자유 레코드로 업데이트: header → 4

**결과:**

| 위치 | ID | Name | Department | Salary |
|------|-----|------|------------|--------|
| header | → 4 | | | |
| record 0 | 10101 | Srinivasan | Comp. Sci. | 65000 |
| record 1 | 24556 | Turnamian | Finance | 98000 |
| record 2 | 34556 | Thompson | Music | 67000 |
| record 3 | 22222 | Einstein | Physics | 95000 |
| record 4 | → 6 | | | |
| record 5 | 33456 | Gold | Physics | 87000 |
| record 6 | | | | |
| record 7 | 58583 | Califieri | History | 62000 |
| record 8 | 76543 | Singh | Finance | 80000 |
| record 9 | 76766 | Crick | Biology | 72000 |
| record 10 | 83821 | Brandt | Comp. Sci. | 92000 |
| record 11 | 98345 | Kim | Elec. Eng. | 80000 |

**Figure 13.103** - 최종 상태

**자유 리스트:** header → 4 → 6 → NULL

### 자유 리스트 관리 알고리즘

**삽입 알고리즘:**
```
procedure INSERT(record):
    if header == NULL:
        // 파일 확장 필요
        extend_file()

    free_slot = header
    header = free_slot.next
    free_slot.data = record
    free_slot.deleted_flag = false
```

**삭제 알고리즘:**
```
procedure DELETE(record_position):
    record = get_record(record_position)
    record.next = header
    header = record_position
    record.deleted_flag = true
```

### 자유 리스트의 장단점

**장점:**
- O(1) 삽입/삭제
- 동적 공간 재사용
- 구현 단순

**단점:**
- 포인터 저장 공간 필요
- 캐시 지역성 낮음
- 단편화 가능

---

## 문제 13.3: 다중 테이블 클러스터링

### 문제
관계 `section`과 `takes`를 고려하시오. 각각 5명의 학생을 가진 3개의 섹션으로 구성된 예제 인스턴스를 제시하고, 다중 테이블 클러스터링을 사용하는 파일 구조를 보이시오.

### 예제 데이터

#### section 관계 (3개 섹션)

| course_id | sec_id | semester | year | building | room |
|-----------|--------|----------|------|----------|------|
| BIO-301 | 1 | Summer | 2010 | Painter | 514 |
| CS-101 | 1 | Fall | 2009 | Packard | 101 |
| CS-347 | 1 | Fall | 2009 | Taylor | 3128 |

#### takes 관계 (각 섹션당 5명)

| ID | course_id | sec_id | semester | year | grade |
|----|-----------|--------|----------|------|-------|
| 00128 | CS-101 | 1 | Fall | 2009 | A |
| 12345 | CS-101 | 1 | Fall | 2009 | C |
| 45678 | CS-101 | 1 | Fall | 2009 | F |
| 54321 | CS-101 | 1 | Fall | 2009 | A |
| 76543 | CS-101 | 1 | Fall | 2009 | A |
| 00128 | CS-347 | 1 | Fall | 2009 | A |
| 12345 | CS-347 | 1 | Fall | 2009 | A |
| 23856 | CS-347 | 1 | Fall | 2009 | A |
| 54321 | CS-347 | 1 | Fall | 2009 | A |
| 76543 | CS-347 | 1 | Fall | 2009 | A |
| 17968 | BIO-301 | 1 | Summer | 2010 | null |
| 59762 | BIO-301 | 1 | Summer | 2010 | null |
| 78546 | BIO-301 | 1 | Summer | 2010 | null |
| 89729 | BIO-301 | 1 | Summer | 2010 | null |
| 98988 | BIO-301 | 1 | Summer | 2010 | null |

### 다중 테이블 클러스터링 구조

**개념:**
- 조인이 자주 발생하는 테이블들을 물리적으로 인접하게 저장
- section과 takes는 (course_id, sec_id, semester, year)로 조인됨
- 각 section 레코드 다음에 해당하는 takes 레코드들을 저장

**클러스터된 파일 구조:**

```
┌─────────────────────────────────────────────────────────────┐
│ BIO-301, 1, Summer, 2010, Painter, 514                      │  ← section
├─────────────────────────────────────────────────────────────┤
│   17968, BIO-301, 1, Summer, 2010, null                     │  ← takes
│   59762, BIO-301, 1, Summer, 2010, null                     │
│   78546, BIO-301, 1, Summer, 2010, null                     │
│   89729, BIO-301, 1, Summer, 2010, null                     │
│   98988, BIO-301, 1, Summer, 2010, null                     │
├─────────────────────────────────────────────────────────────┤
│ CS-101, 1, Fall, 2009, Packard, 101                         │  ← section
├─────────────────────────────────────────────────────────────┤
│   00128, CS-101, 1, Fall, 2009, A                           │  ← takes
│   12345, CS-101, 1, Fall, 2009, C                           │
│   45678, CS-101, 1, Fall, 2009, F                           │
│   54321, CS-101, 1, Fall, 2009, A                           │
│   76543, CS-101, 1, Fall, 2009, A                           │
├─────────────────────────────────────────────────────────────┤
│ CS-347, 1, Fall, 2009, Taylor, 3128                         │  ← section
├─────────────────────────────────────────────────────────────┤
│   00128, CS-347, 1, Fall, 2009, A                           │  ← takes
│   12345, CS-347, 1, Fall, 2009, A                           │
│   23856, CS-347, 1, Fall, 2009, A                           │
│   54321, CS-347, 1, Fall, 2009, A                           │
│   76543, CS-347, 1, Fall, 2009, A                           │
└─────────────────────────────────────────────────────────────┘
```

**Figure 13.105** - 다중 테이블 클러스터링 예시

### 다중 테이블 클러스터링의 장점

**1. 조인 성능 향상:**
```sql
SELECT *
FROM section NATURAL JOIN takes
WHERE course_id = 'CS-101';
```
- section 레코드와 관련 takes 레코드들이 인접
- 디스크 I/O 최소화
- 캐시 히트율 증가

**2. 범위 쿼리 최적화:**
```sql
SELECT student_id
FROM section NATURAL JOIN takes
WHERE semester = 'Fall' AND year = 2009;
```
- 관련 데이터를 연속적으로 읽을 수 있음

**3. 저장 공간 절약:**
- 조인 키 중복 저장 최소화
- section의 (course_id, sec_id, semester, year) 한 번만 저장

### 다중 테이블 클러스터링의 단점

**1. 삽입 복잡도 증가:**
- takes 레코드 삽입 시 올바른 section 블록 찾기 필요
- 블록이 가득 차면 오버플로 처리 필요

**2. 삭제 단편화:**
- takes 레코드 삭제 시 section 블록 내 빈 공간 발생

**3. 비조인 쿼리 성능 저하:**
```sql
SELECT * FROM takes WHERE grade = 'A';
```
- takes만 스캔하는 쿼리는 section 레코드도 읽어야 함
- 불필요한 I/O 발생

### 실무 적용 가이드

**적합한 경우:**
- 조인이 매우 빈번한 테이블 쌍
- 1:N 관계 (section:takes)
- 조인 키가 안정적 (자주 변경되지 않음)
- 읽기 중심 워크로드

**부적합한 경우:**
- 독립적인 쿼리가 많은 경우
- M:N 관계
- 삽입/삭제가 빈번한 경우
- 작은 테이블

---

## 문제 13.4: 비트맵 자유 공간 맵

### 문제
블록당 2비트를 사용하는 비트맵 자유 공간 맵을 고려하시오:
- 00: 0-30% 사용
- 01: 30-60% 사용
- 10: 60-90% 사용
- 11: 90% 이상 사용

**(a)** 1바이트 대신 2비트를 사용하는 두 가지 이점과 한 가지 단점
**(b)** 레코드 삽입/삭제 시 비트맵 업데이트 방법
**(c)** 자유 리스트 대비 비트맵 기법의 이점

### (a) 2비트 사용의 장단점

#### 이점 1: 메모리 효율성

**1바이트 방식:**
- 블록당 8비트 (1바이트) 사용
- 256단계의 정밀도
- 예: 1,000,000 블록 = 1MB

**2비트 방식:**
- 블록당 2비트만 사용
- 4단계의 정밀도
- 예: 1,000,000 블록 = 250KB (75% 절약)

**실무 이점:**
- 전체 비트맵을 메모리에 유지 가능
- 대용량 파일도 빠른 자유 공간 검색
- 캐시 효율성 증가

#### 이점 2: 업데이트 빈도 감소

**예제 시나리오:**

**블록 상태: 50% 사용 (01)**

| 작업 | 사용률 변화 | 1바이트 | 2비트 |
|------|-------------|---------|-------|
| 5% 레코드 삽입 | 50% → 55% | 업데이트 | 업데이트 안 함 (여전히 01) |
| 10% 레코드 삽입 | 55% → 65% | 업데이트 | 업데이트 (01 → 10) |
| 3% 레코드 삭제 | 50% → 47% | 업데이트 | 업데이트 안 함 |

**통계:**
- 많은 삽입/삭제가 경계를 넘지 않음
- 비트맵 업데이트 빈도 약 70% 감소
- 디스크 I/O 감소

#### 단점: 정확도 감소

**문제 상황:**

**시나리오 1: 공간 낭비**
```
블록 A: 31% 사용 → 비트맵 01
블록 B: 59% 사용 → 비트맵 01
```
- 100 바이트 레코드 삽입 필요
- 블록 A를 선택했지만 충분한 공간 없음
- 블록 B를 다시 검색해야 함
- **불필요한 I/O 발생**

**시나리오 2: 검색 비용 증가**
```
비트맵: 01인 블록 100개 발견
실제 사용률: 30~60% 범위에 분산
```
- 적절한 블록 찾기 위해 여러 블록 검사 필요
- 평균 검색 시간 증가

**시나리오 3: 단편화 증가**
- 정확한 자유 공간을 모르므로
- 작은 레코드가 큰 공간 차지
- 큰 레코드를 위한 연속 공간 부족

### (b) 비트맵 업데이트 방법

#### 삽입 시 업데이트

**알고리즘:**
```
procedure INSERT_UPDATE(block_id, record_size):
    old_usage = block_usage[block_id]
    new_usage = old_usage + record_size

    old_level = get_level(old_usage)
    new_level = get_level(new_usage)

    if old_level != new_level:
        bitmap[block_id] = new_level
        // 비트맵 페이지를 dirty로 표시
        mark_dirty(bitmap_page)

    block_usage[block_id] = new_usage
```

**get_level 함수:**
```
function get_level(usage_percent):
    if usage_percent < 30:
        return 00  // 0-30%
    else if usage_percent < 60:
        return 01  // 30-60%
    else if usage_percent < 90:
        return 10  // 60-90%
    else:
        return 11  // 90% 이상
```

**예제:**
```
블록 5: 현재 55% 사용 (01)
레코드 삽입: 10% 공간 사용

새 사용률: 65%
새 레벨: 10 (60-90%)
동작: 비트맵 업데이트 (01 → 10)
```

#### 삭제 시 업데이트

**알고리즘:**
```
procedure DELETE_UPDATE(block_id, record_size):
    old_usage = block_usage[block_id]
    new_usage = old_usage - record_size

    old_level = get_level(old_usage)
    new_level = get_level(new_usage)

    if old_level != new_level:
        bitmap[block_id] = new_level
        mark_dirty(bitmap_page)

    block_usage[block_id] = new_usage
```

**예제:**
```
블록 8: 현재 62% 사용 (10)
레코드 삭제: 5% 공간 확보

새 사용률: 57%
새 레벨: 01 (30-60%)
동작: 비트맵 업데이트 (10 → 01)
```

#### 최적화: 경계 근처 처리

**문제:**
- 사용률이 경계 근처에서 반복적으로 변경되면 업데이트 빈번

**해결책: 히스테리시스 (Hysteresis)**
```
function get_level_with_hysteresis(old_level, new_usage):
    // 상향 이동: 경계 +2%에서만
    // 하향 이동: 경계 -2%에서만

    if old_level == 00 and new_usage > 32:
        return 01
    if old_level == 01 and new_usage < 28:
        return 00
    if old_level == 01 and new_usage > 62:
        return 10
    // ... 기타 경우들
```

### (c) 비트맵 vs 자유 리스트

#### 자유 공간 검색

**자유 리스트:**
```
procedure FIND_FREE_SPACE_LIST(required_size):
    current = free_list_head
    while current != NULL:
        if current.size >= required_size:
            return current
        current = current.next
    return NULL  // 공간 없음
```

**문제점:**
- 선형 검색: O(n)
- 적절한 크기 찾기 위해 여러 노드 검사
- 각 노드 접근 시 디스크 I/O 가능
- 큰 레코드의 경우 많은 검색 필요

**비트맵:**
```
procedure FIND_FREE_SPACE_BITMAP(required_size):
    required_level = calculate_required_level(required_size)

    // 한 페이지의 비트맵으로 수천 개 블록 커버
    for bitmap_page in bitmap:
        for block in bitmap_page:
            if bitmap[block] <= required_level:
                return block
    return NULL
```

**장점:**
- 한 페이지로 많은 블록 정보 확인
- 메모리 내에서 빠른 검색
- I/O 최소화

**예제:**
```
4KB 비트맵 페이지 = 4096 바이트 = 32,768 비트
2비트/블록 → 16,384 블록 정보

하나의 I/O로 16,384개 블록의 자유 공간 정보 획득!
```

#### 자유 공간 업데이트

**자유 리스트:**
```
procedure UPDATE_FREE_LIST(block_id, operation):
    if operation == DELETE:
        // 리스트에 노드 추가
        new_node.id = block_id
        new_node.next = free_list_head
        free_list_head = new_node
        // 디스크 I/O 발생

    if operation == INSERT:
        // 리스트에서 노드 제거
        // 디스크 I/O 발생
```

**문제점:**
- 매 삽입/삭제마다 자유 리스트 수정
- 포인터 업데이트 필요
- 디스크 I/O 빈번

**비트맵:**
```
procedure UPDATE_BITMAP(block_id, new_level):
    old_level = bitmap[block_id]

    if old_level != new_level:
        bitmap[block_id] = new_level
        // 비트맵 페이지만 dirty 표시
        // 실제 디스크 쓰기는 나중에 일괄 처리
```

**장점:**
- 레벨 변경 시에만 업데이트 (약 30%)
- 비트맵 페이지 단위로 일괄 처리
- 디스크 I/O 대폭 감소

#### 대량 블록 할당

**시나리오: 100개의 연속 블록 필요**

**자유 리스트:**
```
1. 100개의 연속 자유 블록 찾기 → 매우 어려움
2. 여러 개의 작은 자유 블록들을 연결 → 단편화
3. 많은 노드 검색 필요 → O(n)
```

**비트맵:**
```
1. 비트맵을 스캔하여 00이 100개 연속된 구간 찾기
2. 비트 연산으로 빠른 검색:
   - 00000000 패턴 찾기 (4개 연속 자유 블록)
   - 비트마스크 연산으로 고속 처리
3. 한 번의 비트맵 스캔으로 전체 파악
```

#### 성능 비교

| 작업 | 자유 리스트 | 비트맵 | 비트맵 이점 |
|------|-------------|--------|-------------|
| 작은 블록 검색 | O(n) 리스트 순회 | O(1) 비트 확인 | 10-100배 빠름 |
| 큰 블록 검색 | O(n²) 연속 검색 | O(n) 비트맵 스캔 | 100-1000배 빠름 |
| 삽입 업데이트 | 항상 I/O | 30% 경우만 I/O | 70% I/O 감소 |
| 삭제 업데이트 | 항상 I/O | 30% 경우만 I/O | 70% I/O 감소 |
| 메모리 사용 | 노드당 8-16바이트 | 블록당 2비트 | 32-64배 절약 |

#### 실무 권장사항

**비트맵 사용:**
- 대용량 데이터베이스 (>1GB)
- 빈번한 삽입/삭제
- 대량 블록 할당 필요
- 메모리 효율 중요

**자유 리스트 사용:**
- 소규모 데이터베이스 (<100MB)
- 단순한 구현 필요
- 정확한 자유 공간 크기 중요

---

## 문제 13.5: 버퍼에서 블록 찾기

### 문제
블록이 버퍼에 있는지, 있다면 어디에 있는지 빠르게 찾는 것이 중요합니다. 데이터베이스 버퍼 크기가 매우 클 때, 어떤 (메모리 내) 자료구조를 사용하겠습니까?

### 해답: 해시 테이블

#### 설계

**해시 테이블 구조:**
```
hash_table[HASH_SIZE]
    key: (file_id, block_number)
    value: buffer_frame_pointer
```

**해시 함수:**
```c
int hash_function(int file_id, int block_number) {
    // 곱셈 해시 (빠르고 효율적)
    return ((file_id * 31) + block_number) % HASH_SIZE;
}
```

#### 주요 연산

**블록 검색:**
```
procedure FIND_BLOCK(file_id, block_number):
    hash_value = hash_function(file_id, block_number)
    bucket = hash_table[hash_value]

    // 체이닝으로 충돌 해결
    for entry in bucket:
        if entry.file_id == file_id and
           entry.block_number == block_number:
            return entry.buffer_frame

    return NULL  // 버퍼에 없음
```

**시간 복잡도: O(1) 평균**

**블록 삽입:**
```
procedure INSERT_BLOCK(file_id, block_number, buffer_frame):
    hash_value = hash_function(file_id, block_number)
    bucket = hash_table[hash_value]

    new_entry = {file_id, block_number, buffer_frame}
    bucket.add(new_entry)
```

**블록 제거:**
```
procedure REMOVE_BLOCK(file_id, block_number):
    hash_value = hash_function(file_id, block_number)
    bucket = hash_table[hash_value]

    for entry in bucket:
        if entry.file_id == file_id and
           entry.block_number == block_number:
            bucket.remove(entry)
            return
```

#### 크기 결정

**해시 테이블 크기:**
- 버퍼 프레임 수의 2-3배
- 소수(prime number) 선택 (충돌 감소)
- 예: 10,000 프레임 → 20,011 (소수)

**부하율 (Load Factor):**
```
α = n / m
n = 버퍼 프레임 수
m = 해시 테이블 크기

목표: α < 0.75 (좋은 성능 유지)
```

#### 대안: 다른 자료구조

**트리 기반 (Red-Black Tree, AVL Tree):**
- 시간 복잡도: O(log n)
- 순서 있는 순회 가능
- 구현 복잡
- 해시보다 느림

**이진 검색 배열:**
- 시간 복잡도: O(log n) 검색, O(n) 삽입/삭제
- 메모리 효율적
- 동적 변경에 비효율적

**해시 테이블이 최선인 이유:**
- O(1) 평균 시간
- 간단한 구현
- 메모리 효율적
- 동적 변경 용이

---

## 문제 13.6: 테이블 파티셔닝

### 문제
대학교에 수년간 축적된 매우 많은 `takes` 레코드가 있다고 가정합니다. `takes` 관계에서 테이블 파티셔닝을 어떻게 할 수 있는지, 어떤 이점을 제공하는지 설명하시오. 또한 이 기법의 잠재적 단점도 설명하시오.

### 파티셔닝 전략

#### 파티션 키 선택

**최적 파티션 키: (year, semester)**

**이유:**
- 시간 기반 접근 패턴 (최근 데이터 접근 빈번)
- 명확한 분할 경계
- 쿼리 술어에 자주 사용
- 균등한 분포

#### 파티션 구조

**예제: 10년간 데이터**
```
Partition 1: takes_2015_spring
Partition 2: takes_2015_fall
Partition 3: takes_2016_spring
Partition 4: takes_2016_fall
...
Partition 19: takes_2024_spring
Partition 20: takes_2024_fall
```

### 이점

#### 1. 계층적 저장소 활용

**저장소 계층:**
```
┌─────────────────────────────────────────┐
│ SSD (빠름, 비쌈)                         │
│ - 2024 Fall, 2024 Spring (최근 1년)     │
│ - 빈번한 접근                            │
│ - 빠른 응답 시간                         │
├─────────────────────────────────────────┤
│ HDD (보통, 저렴)                         │
│ - 2020-2023 (2-5년 전)                  │
│ - 가끔 접근                              │
│ - 중간 성능                              │
├─────────────────────────────────────────┤
│ 아카이브/테이프 (느림, 매우 저렴)        │
│ - 2015-2019 (5년 이상)                  │
│ - 드물게 접근                            │
│ - 규정 준수 목적                         │
└─────────────────────────────────────────┘
```

**비용 절감:**
- 전체 데이터를 SSD에 저장: $10,000
- 파티셔닝 후: SSD $1,000 + HDD $500 = $1,500
- **절감: $8,500 (85%)**

#### 2. 쿼리 성능 향상 (Partition Pruning)

**쿼리 1: 특정 년도**
```sql
SELECT *
FROM takes
WHERE year = 2024 AND semester = 'Fall';
```

**파티셔닝 없이:**
- 전체 테이블 스캔: 10,000,000 레코드
- 읽기 시간: 10초

**파티셔닝 있음:**
- takes_2024_fall만 스캔: 500,000 레코드
- 읽기 시간: 0.5초
- **성능 향상: 20배**

**쿼리 2: 최근 학기**
```sql
SELECT student_id, AVG(grade_point)
FROM takes
WHERE year >= 2023
GROUP BY student_id;
```

**파티셔닝 효과:**
- 2023-2024 파티션만 접근 (4개)
- 나머지 16개 파티션 무시
- **I/O 80% 감소**

#### 3. 유지보수 간편화

**인덱스 재구축:**
```sql
-- 파티셔닝 없이: 전체 테이블
REBUILD INDEX takes_idx;  -- 2시간

-- 파티셔닝 있음: 파티션별
REBUILD INDEX takes_2024_fall_idx;  -- 6분
```

**파티션 삭제:**
```sql
-- 10년 전 데이터 삭제
DROP PARTITION takes_2015_spring;  -- 즉시
-- vs
DELETE FROM takes WHERE year = 2015 AND semester = 'Spring';  -- 1시간+
```

#### 4. 백업 및 복구 최적화

**증분 백업:**
```bash
# 변경된 파티션만 백업
backup takes_2024_fall  # 최근 학기만
# vs
backup takes            # 전체 10GB
```

**빠른 복구:**
- 특정 파티션만 복구 가능
- 복구 시간 단축 (전체 → 부분)

#### 5. 병렬 처리

**병렬 쿼리:**
```sql
SELECT semester, COUNT(*)
FROM takes
WHERE year BETWEEN 2022 AND 2024
GROUP BY semester;
```

- 각 파티션을 병렬로 스캔
- 6개 파티션 → 6개 스레드
- **실행 시간: 1/6**

### 단점

#### 1. 다중 파티션 쿼리 오버헤드

**문제 쿼리:**
```sql
-- 모든 년도에서 'A' 학점 검색
SELECT *
FROM takes
WHERE grade = 'A';
```

**오버헤드:**
- 20개 파티션 모두 열기
- 각 파티션의 인덱스 접근
- 결과 병합
- **성능: 파티셔닝 없는 경우보다 10-15% 느림**

**이유:**
- 파티션 간 전환 비용
- 메타데이터 오버헤드
- 여러 파일 열기/닫기

#### 2. 파티션 간 조인 비용

**쿼리:**
```sql
SELECT t1.student_id
FROM takes t1, takes t2
WHERE t1.student_id = t2.student_id
  AND t1.year = 2023
  AND t2.year = 2024;
```

**비용:**
- 서로 다른 파티션 간 조인
- 파티션 간 데이터 이동 필요
- 캐시 효율 감소

#### 3. 불균등한 파티션 크기

**문제:**
```
Summer 2024: 5,000 레코드
Fall 2024:   50,000 레코드 (10배)
```

- 일부 파티션에 부하 집중
- 병렬 처리 불균형
- 핫스팟 발생

#### 4. 관리 복잡도

**작업:**
- 파티션 추가/삭제 스케줄링
- 파티션별 인덱스 관리
- 파티션 통계 업데이트
- 파티션 간 밸런싱

### 실무 권장사항

**파티셔닝 적용 기준:**
```
✓ 테이블 크기 > 10GB
✓ 시간 기반 쿼리 > 70%
✓ 이력 데이터
✓ 명확한 분할 기준
✓ 삭제/아카이빙 필요

✗ 작은 테이블 (<1GB)
✗ 다양한 쿼리 패턴
✗ 파티션 간 조인 빈번
```

**파티션 크기 권장:**
- 최소: 100MB
- 최적: 1-10GB
- 최대: 100GB

**파티션 개수 권장:**
- 최소: 4개
- 최적: 10-50개
- 최대: 1000개 (관리 부담)

---

## 문제 13.7: LRU vs MRU 버퍼 교체 정책

### 문제
각 상황에서 관계 대수 표현식과 쿼리 처리 전략의 예를 제시하시오:

**(a)** MRU가 LRU보다 바람직한 경우
**(b)** LRU가 MRU보다 바람직한 경우

### (a) MRU가 LRU보다 바람직한 경우

#### 시나리오: 중첩 루프 조인 (Nested-Loop Join)

**쿼리:**
```sql
SELECT *
FROM R1, R2
WHERE R1.id = R2.id;
```

**처리 전략:**
```
for each tuple t2 in R2:
    for each block b1 in R1:
        for each tuple t1 in b1:
            if t1.id == t2.id:
                output (t1, t2)
```

#### 버퍼 접근 패턴

**R1 블록 순서:**
```
반복 1 (R2의 첫 튜플):
    접근: B1 → B2 → B3 → B4 → B5

반복 2 (R2의 두 번째 튜플):
    접근: B1 → B2 → B3 → B4 → B5

...
```

#### LRU의 문제

**버퍼 상태 (3 프레임):**
```
반복 1 끝:
    [B3] [B4] [B5]  ← B1, B2는 LRU로 교체됨

반복 2 시작:
    B1 필요 → 버퍼 미스!
    B1 읽기 → B3 교체 (가장 오래 사용 안 함)

반복 2:
    [B1] [B4] [B5]
    B2 필요 → 버퍼 미스!
    B2 읽기 → B4 교체
```

**결과:**
- 매 반복마다 R1의 앞부분 블록들 재읽기
- 불필요한 I/O 폭증

#### MRU의 해결책

**버퍼 상태:**
```
반복 1 끝:
    [B1] [B2] [B3]  ← B5가 MRU로 교체됨 (가장 최근 사용)

반복 2 시작:
    B1 필요 → 버퍼 히트!

계속:
    [B1] [B2] [B3] 유지
    B4, B5만 반복적으로 교체
```

**결과:**
- R1의 초기 블록들 버퍼에 유지
- I/O 최소화

#### 성능 비교

**설정:**
- R1: 1000 블록
- R2: 10000 튜플
- 버퍼: 100 프레임

| 정책 | I/O 횟수 | 설명 |
|------|----------|------|
| LRU | 1000 × 10000 = 10,000,000 | 매 반복마다 R1 전체 읽기 |
| MRU | 1000 + (10000 × 10) = 101,000 | 초기 + 끝 블록들만 반복 |
| **개선** | **99배** | |

### (b) LRU가 MRU보다 바람직한 경우

#### 시나리오: 정렬 병합 조인 (Sort-Merge Join)

**쿼리:**
```sql
SELECT *
FROM R1, R2
WHERE R1.id = R2.id;
```

**처리 전략:**
```
1. R1과 R2를 조인 속성으로 정렬
2. 동시에 스캔하면서 매칭

while not_end_of_R1 and not_end_of_R2:
    if R1.current.id == R2.current.id:
        output matching tuples
        advance both
    else if R1.current.id < R2.current.id:
        advance R1
    else:
        advance R2
```

#### 중복 값 처리

**문제:**
```
R1: [1, 1, 1, 2, 3, 3, ...]
R2: [1, 2, 2, 2, 3, 4, ...]
```

- R1의 id=1 처리 중
- R2의 id=1을 3번 반복 읽어야 함
- **백트래킹(back-up) 필요**

#### LRU의 장점

**버퍼 상태:**
```
R1 블록 B1 (id=1들):
    읽기 → 사용 → 버퍼에 유지 (최근 사용)

R2 블록 B2 (id=1):
    읽기 → R1 B1과 조인 → 버퍼에 유지

백트래킹:
    R2의 id=1로 돌아가기
    B2가 여전히 버퍼에 있음! (LRU로 유지)
```

**결과:**
- 백트래킹 시 버퍼 히트
- 불필요한 I/O 없음

#### MRU의 문제

**버퍼 상태:**
```
R1 블록 B1 (id=1들):
    읽기 → 사용

R2 블록 B2 (id=1):
    읽기 → R1 B1과 조인
    다음 블록 B3 읽기 → B2 교체됨! (MRU)

백트래킹:
    R2의 id=1로 돌아가기
    B2가 없음 → 버퍼 미스!
    B2 재읽기 필요
```

#### 또 다른 예: 순차 스캔 with 필터

**쿼리:**
```sql
SELECT *
FROM R1
WHERE R1.value > 1000
ORDER BY R1.id;
```

**접근 패턴:**
```
B1 → B2 → B3 → ... → B100

경우에 따라 앞 블록 재방문 필요:
예: B50 처리 중 B10의 데이터 참조
```

**LRU:**
- 최근 접근한 블록들 유지
- 재방문 시 버퍼 히트 가능성 높음

**MRU:**
- 가장 최근 블록 즉시 교체
- 재방문 시 버퍼 미스 발생

### MRU의 근본적 문제

**이슈:**
- 사용하지 않는 블록이 영원히 메모리에 남을 수 있음
- 버퍼 공간 낭비

**예:**
```
100 프레임 버퍼

중첩 루프 조인 시작:
    처음 90개 블록 → 버퍼에 로드
    나머지 10개 프레임으로 나머지 블록 처리

조인 완료 후:
    처음 90개 블록이 여전히 버퍼에!
    다른 쿼리에서 사용 불가
```

**해결책: 하이브리드 접근**
```
쿼리별 버퍼 힌트:
    중첩 루프 → MRU 사용
    정렬 병합 → LRU 사용
    일반 쿼리 → LRU 기본
```

### 실무 권장사항

**현대 DBMS:**
- 기본: LRU 또는 Clock 알고리즘
- 특수 케이스: 쿼리 옵티마이저가 힌트 제공
- MRU는 명시적으로 지정할 때만 사용

**PostgreSQL:**
- Ring buffer: 대량 스캔에 별도 버퍼 사용
- LRU + Clock: 일반 쿼리

**Oracle:**
- LRU + Touch count
- Hot/Cold 리스트 분리

---

## 문제 13.8: PostgreSQL의 버퍼 관리 전략

### 문제
PostgreSQL은 일반적으로 작은 버퍼를 사용하고, 나머지 메인 메모리 관리를 운영체제 버퍼 관리자에 맡깁니다.

**(a)** 이 접근법의 이점은 무엇인가?
**(b)** 이 접근법의 주요 한계는 무엇인가?

### (a) 이점

#### 1. 메모리 경쟁 회피

**문제 시나리오 (전통적 DBMS):**
```
시스템 메모리: 16GB

PostgreSQL: 12GB 할당
웹 서버: 2GB 필요
백업 프로세스: 2GB 필요
OS: 1GB 필요
----------------------------------------
필요: 17GB → 부족!
```

**결과:**
- 스왑 발생
- 전체 시스템 성능 저하
- OOM (Out of Memory) 킬러 발동 가능

**PostgreSQL 접근법:**
```
PostgreSQL 버퍼: 2GB (작게 유지)
OS 파일 캐시: 12GB (동적 할당)
여유 공간: 2GB
```

**이점:**
- OS가 메모리 동적 관리
- 다른 프로세스와 메모리 공유
- 스왑 최소화

#### 2. 이중 버퍼링 (Double Buffering) 효과

**구조:**
```
┌─────────────────────────────────────┐
│ PostgreSQL 버퍼 (Shared Buffers)    │
│ 128MB - 2GB                          │
│ - 자주 접근하는 hot 페이지          │
│ - PostgreSQL이 직접 관리            │
└─────────────────────────────────────┘
          ↓ eviction
┌─────────────────────────────────────┐
│ OS 파일 시스템 캐시 (Page Cache)    │
│ 10-14GB (시스템 메모리의 대부분)    │
│ - 최근 읽은 모든 페이지             │
│ - OS가 LRU로 관리                   │
└─────────────────────────────────────┘
          ↓ eviction
┌─────────────────────────────────────┐
│ 디스크                               │
└─────────────────────────────────────┘
```

**버퍼 미스 시나리오:**
```
1. PostgreSQL 버퍼에서 페이지 검색 → 미스
2. OS 파일 캐시에서 검색 → 히트! (매우 빠름)
3. 디스크 I/O 회피 → 성능 저하 최소
```

**비용 비교:**
- 디스크 I/O: 10ms
- OS 파일 캐시: 0.001ms (10,000배 빠름)
- PostgreSQL 버퍼: 0.0001ms (100,000배 빠름)

#### 3. 워크로드 적응성

**예제: 혼합 워크로드**

**시나리오 1: OLTP (낮 시간)**
```
작은 트랜잭션들:
    PostgreSQL 버퍼: hot 인덱스 페이지 캐싱
    OS 캐시: 데이터 페이지 캐싱

메모리 사용: PostgreSQL 2GB + OS 8GB = 10GB
```

**시나리오 2: 분석 쿼리 (밤 시간)**
```
대량 스캔:
    PostgreSQL 버퍼: 유지 (2GB)
    OS 캐시: 자동 확장 → 12GB

메모리 사용: PostgreSQL 2GB + OS 12GB = 14GB
```

**이점:**
- OS가 자동으로 워크로드에 맞춰 메모리 재분배
- DBA의 수동 조정 불필요

#### 4. 구현 단순성

**PostgreSQL 관리:**
- 작은 버퍼 풀만 관리
- 단순한 교체 정책
- 적은 메타데이터

**OS 역할:**
- 복잡한 메모리 관리
- 프로세스 간 밸런싱
- NUMA 최적화
- 메모리 압축

### (b) 주요 한계

#### 1. 교체 정책 제어 불가

**문제:**
OS는 데이터베이스 의미를 모름

**시나리오: 중첩 루프 조인**
```
쿼리 실행:
    R1 테이블 스캔 (큰 테이블, 1GB)
    → OS는 모든 페이지를 캐시하려 함
    → 중요한 인덱스 페이지가 밀려남
```

**PostgreSQL 관점:**
```
필요:
    인덱스 페이지: 자주 재사용 → 유지 필요
    스캔 페이지: 1회 사용 → 버릴 수 있음

OS 관점:
    모든 페이지를 동등하게 처리 (LRU)
    → 인덱스 페이지도 교체될 수 있음
```

**결과:**
- 인덱스 페이지 재읽기
- 성능 저하

#### 2. 대량 스캔의 캐시 오염

**문제:**
```sql
-- 대용량 로그 테이블 스캔
SELECT COUNT(*)
FROM access_log;  -- 10GB 테이블
```

**OS 동작:**
```
1. access_log의 모든 페이지를 캐시에 로드
2. 기존 캐시 내용(hot 데이터) 밀어냄
3. access_log 스캔 완료 후 캐시에 남음
4. access_log는 다시 사용되지 않음
```

**피해:**
- 자주 사용되는 데이터 (예: 사용자 테이블, 인덱스) 제거
- 다음 쿼리들의 성능 저하

**PostgreSQL의 부분적 해결책:**
- Ring buffer: 대량 스캔에 별도의 작은 버퍼 사용
- 하지만 OS 캐시는 여전히 오염됨

#### 3. 쿼리별 최적화 불가

**예제: 해시 조인**

**PostgreSQL이 알고 있는 것:**
```
쿼리 계획:
    1. 작은 테이블로 해시 테이블 구축
    2. 큰 테이블 스캔하며 조인

메모리 요구:
    해시 테이블: 메모리에 유지 필수
    스캔 페이지: 1회 사용 후 버려도 됨
```

**PostgreSQL이 할 수 있는 것:**
```
Shared buffer 내에서:
    해시 테이블 페이지 우선순위 높임

BUT OS 캐시는 제어 불가:
    OS는 무작위로 페이지 교체 가능
```

**결과:**
- 해시 테이블이 OS 캐시에서 밀려날 수 있음
- 조인 성능 저하

#### 4. 통계 정보 부족

**PostgreSQL이 모르는 것:**
```
OS 캐시 상태:
    - 어떤 페이지가 OS 캐시에 있는가?
    - 얼마나 많은 메모리가 사용 가능한가?
```

**영향:**
```
쿼리 옵티마이저:
    인덱스 스캔 vs 시퀀셜 스캔 결정

잘못된 가정:
    인덱스가 디스크에 있다고 가정
    → 시퀀셜 스캔 선택

실제:
    인덱스는 OS 캐시에 있음
    → 인덱스 스캔이 더 빠를 수 있었음
```

#### 5. 일관성 없는 성능

**시나리오 1: 캐시 워밍 후**
```sql
SELECT * FROM users WHERE id = 1;
-- 실행 시간: 0.1ms (OS 캐시 히트)
```

**시나리오 2: OS가 캐시 정리 후**
```sql
SELECT * FROM users WHERE id = 1;
-- 실행 시간: 10ms (디스크 I/O)
```

**문제:**
- 동일한 쿼리의 성능이 100배 차이
- 예측 불가능
- 벤치마킹 어려움

### PostgreSQL의 하이브리드 접근법

**실제 구현:**

**Shared Buffers (PostgreSQL 관리):**
- 크기: 128MB ~ 8GB (시스템 메모리의 25%)
- 용도: hot 페이지, 인덱스, 자주 수정되는 데이터

**Effective Cache (OS + PostgreSQL):**
- 크기: 시스템 메모리의 75%
- 용도: 모든 데이터베이스 파일

**권장 설정:**
```
시스템 메모리: 64GB

shared_buffers = 16GB      (25%)
effective_cache_size = 48GB (75%)
```

**effective_cache_size:**
- 실제 메모리 할당 아님
- 옵티마이저에게 주는 힌트
- "이 정도 메모리가 캐싱에 사용 가능"

### 대안: 다른 DBMS의 접근법

**Oracle:**
- 대형 SGA (System Global Area)
- 모든 캐싱을 직접 관리
- 세밀한 제어, 복잡한 관리

**MySQL (InnoDB):**
- innodb_buffer_pool_size
- 시스템 메모리의 70-80%
- PostgreSQL보다 적극적

**PostgreSQL의 철학:**
- 단순성과 안정성 우선
- OS의 검증된 메모리 관리 활용
- 복잡한 캐시 관리 회피

### 실무 권장사항

**PostgreSQL 사용 시:**

**작은 시스템 (<16GB):**
- shared_buffers = 2-4GB
- OS 캐시에 대부분 의존

**중형 시스템 (16-64GB):**
- shared_buffers = 4-16GB
- 균형잡힌 접근

**대형 시스템 (>64GB):**
- shared_buffers = 16-32GB
- 더 이상 늘리면 효과 감소

**주의:**
- shared_buffers를 너무 크게 설정하지 말 것
- OS 캐시를 위한 공간 확보
- 일반적으로 시스템 메모리의 25% 이하

---

## 학습 정리

### 핵심 개념

1. **레코드 삭제 기법**
   - 순차 이동: 순서 유지, I/O 많음
   - 마지막 이동: 빠름, 순서 파괴
   - 논리 삭제: 최소 I/O, 단편화

2. **자유 공간 관리**
   - 자유 리스트: 단순, 포인터 오버헤드
   - 비트맵: 효율적, 대략적 정보

3. **다중 테이블 클러스터링**
   - 조인 최적화
   - 공간 절약
   - 비조인 쿼리 저하

4. **버퍼 관리**
   - 해시 테이블로 빠른 검색
   - LRU vs MRU: 워크로드 의존

5. **테이블 파티셔닝**
   - 계층적 저장소 활용
   - 쿼리 성능 향상
   - 관리 복잡도 증가

6. **PostgreSQL 버퍼 전략**
   - 작은 DB 버퍼 + 큰 OS 캐시
   - 메모리 경쟁 회피
   - 제어 불가의 trade-off

### 실무 적용

1. **삭제 전략 선택**
   - 읽기 중심: 논리 삭제 + 주기적 압축
   - 쓰기 중심: 상황에 따라 선택

2. **대용량 테이블**
   - 파티셔닝 필수 (>10GB)
   - 적절한 파티션 키 선택

3. **버퍼 튜닝**
   - 워크로드 분석
   - 적절한 크기 설정
   - 모니터링 필수

---

## 다음 포스트

데이터베이스 시스템의 물리적 저장 구조에 대한 학습을 마쳤습니다. 다음에는 **인덱싱과 해싱**을 다룰 예정입니다.
