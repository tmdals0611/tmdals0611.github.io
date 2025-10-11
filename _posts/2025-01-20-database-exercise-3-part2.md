---
title: "[데이터베이스] 문제 해설 #3 Part 2 - SQL 고급 쿼리와 UPDATE 응용"
date: 2025-01-20 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [sql, advanced-queries, problem-solving, all-operator, division, update, case, complex-queries]
pin: false
math: true
mermaid: false
---

Part 1에 이어서 Employee 데이터베이스를 사용한 고급 SQL 쿼리 문제들을 다룹니다. 전체 조건(universal quantification), 최댓값 찾기, 조건부 UPDATE 등 실무에서 자주 사용되는 복잡한 쿼리 작성 방법을 배웁니다.

## 📚 Employee Database Schema

```sql
employee(ID, person_name, street, city)
works(ID, company_name, salary)
company(company_name, city)
manages(ID, manager_id)
```

**주요 관계**:
- employee: 직원 정보 (거주지)
- works: 근무 정보 (회사, 급여)
- company: 회사 정보 (위치)
- manages: 관리 관계 (직원 - 관리자)

---

## 문제 3.9 - Employee 데이터베이스 고급 쿼리

### 문제 3.9.a
**문제**: First Bank Corporation에서 일하는 모든 직원의 ID, 이름, 거주 도시를 찾으시오.

**풀이**:
```sql
SELECT e.ID, e.person_name, e.city
FROM employee AS e, works AS w
WHERE w.company_name = 'First Bank Corporation'
  AND w.ID = e.ID;
```

**핵심 개념**:
- **2-way Join**: employee와 works를 ID로 연결
- **별칭 사용**: e, w로 테이블 구분
- works에서 회사 정보, employee에서 개인 정보 가져오기

---

### 문제 3.9.b
**문제**: First Bank Corporation에서 일하며 $10,000 이상 버는 직원의 ID, 이름, 거주 도시를 찾으시오.

**풀이 1 (JOIN 사용)**:
```sql
SELECT e.ID, e.person_name, e.city
FROM employee AS e, works AS w
WHERE w.company_name = 'First Bank Corporation'
  AND w.ID = e.ID
  AND w.salary > 10000;
```

**풀이 2 (부질의 사용)**:
```sql
SELECT *
FROM employee
WHERE ID IN (
    SELECT ID
    FROM works
    WHERE company_name = 'First Bank Corporation'
      AND salary > 10000
);
```

**비교**:
- **JOIN**: 한 번에 처리, 성능 우수
- **부질의**: 논리가 명확, 가독성 좋음

---

### 문제 3.9.c
**문제**: First Bank Corporation에서 일하지 않는 직원의 ID를 찾으시오.

**풀이 1 (단순)**:
```sql
SELECT ID
FROM works
WHERE company_name <> 'First Bank Corporation';
```

**주의사항**:
- works 테이블에만 있는 직원들
- employee에 있지만 works에 없는 직원은 제외됨

**풀이 2 (모든 직원 포함)**:
```sql
SELECT ID
FROM employee
WHERE ID NOT IN (
    SELECT ID
    FROM works
    WHERE company_name = 'First Bank Corporation'
);
```

**차이점**:
- 풀이 1: works에 있지만 First Bank가 아닌 회사 직원
- 풀이 2: employee에 있지만 First Bank 직원이 아닌 모든 사람

**OUTER JOIN 사용 (4장)**:
```sql
SELECT e.ID
FROM employee AS e LEFT OUTER JOIN works AS w ON e.ID = w.ID
WHERE w.company_name IS NULL
   OR w.company_name <> 'First Bank Corporation';
```

---

### 문제 3.9.d
**문제**: Small Bank Corporation의 모든 직원보다 많이 버는 직원의 ID를 찾으시오.

**풀이**:
```sql
SELECT ID
FROM works
WHERE salary > ALL (
    SELECT salary
    FROM works
    WHERE company_name = 'Small Bank Corporation'
);
```

**핵심 개념**:
- **ALL 연산자**: 부질의의 모든 값보다 큰
- salary > ALL (40000, 50000, 60000) ≡ salary > 60000

**실행 과정**:
1. 부질의: Small Bank Corporation 직원들의 급여 조회
   - 예: {40000, 50000, 60000}
2. 최댓값: 60000
3. 외부 쿼리: salary > 60000인 직원 조회

**주의사항**:
- 한 사람이 여러 회사에서 일할 수 있다면 더 복잡
- 하지만 ID가 works의 기본키이므로 불가능

**대안 (MAX 사용)**:
```sql
SELECT ID
FROM works
WHERE salary > (
    SELECT MAX(salary)
    FROM works
    WHERE company_name = 'Small Bank Corporation'
);
```

---

### 문제 3.9.e
**문제**: Small Bank Corporation이 위치한 모든 도시에 위치한 회사들을 찾으시오.

**배경**:
- 회사는 여러 도시에 위치 가능
- Small Bank Corp이 도시 A, B, C에 있다면
- 결과: A, B, C 모두에 있는 회사들

**풀이 (나눗셈 연산 - Division)**:
```sql
SELECT S.company_name
FROM company AS S
WHERE NOT EXISTS (
    (SELECT city
     FROM company
     WHERE company_name = 'Small Bank Corporation')
    EXCEPT
    (SELECT city
     FROM company AS T
     WHERE S.company_name = T.company_name)
);
```

**해석**:
1. **내부 부질의 1**: Small Bank가 있는 모든 도시
   - {New York, London, Tokyo}
2. **내부 부질의 2**: S 회사가 있는 모든 도시
   - 예: {New York, London}
3. **EXCEPT**: Small Bank 도시 - S 회사 도시
   - {Tokyo} (비어있지 않음 → S 제외)
   - {} (비어있음 → S 포함!)
4. **NOT EXISTS**: 차집합이 비어있는 회사 선택

**예시**:
```
Small Bank: {New York, London, Tokyo}
Global Corp: {New York, London, Tokyo, Paris} → 포함 ✅
Local Bank: {New York, London} → 제외 ❌
Mega Corp: {New York, London, Tokyo} → 포함 ✅
```

**핵심 개념**:
- **전체 조건(Universal Quantification)**: "모든 X에 대해"
- **이중 부정**: NOT EXISTS (A EXCEPT B) ≡ B ⊇ A

---

### 문제 3.9.f
**문제**: 가장 많은 직원을 고용한 회사의 이름을 찾으시오 (동점 가능).

**풀이**:
```sql
SELECT company_name
FROM works
GROUP BY company_name
HAVING COUNT(DISTINCT ID) >= ALL (
    SELECT COUNT(DISTINCT ID)
    FROM works
    GROUP BY company_name
);
```

**핵심 개념**:
- **GROUP BY**: 회사별로 그룹화
- **HAVING**: 그룹 필터 조건
- **ALL**: 모든 회사의 직원 수보다 크거나 같음

**실행 과정**:
1. 각 회사별 직원 수 계산
   - First Bank: 100명
   - Small Bank: 80명
   - Mega Corp: 100명
2. 부질의: {100, 80, 100}
3. COUNT >= ALL {100, 80, 100} ≡ COUNT >= 100
4. 결과: First Bank, Mega Corp

**주의사항**:
- DISTINCT는 ID가 기본키라 불필요하지만 명확성을 위해 사용
- 동점인 회사들 모두 반환

---

### 문제 3.9.g
**문제**: 직원들의 평균 급여가 First Bank Corporation보다 높은 회사들을 찾으시오.

**풀이**:
```sql
SELECT company_name
FROM works
GROUP BY company_name
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM works
    WHERE company_name = 'First Bank Corporation'
);
```

**핵심 개념**:
- **AVG()**: 평균 계산
- **GROUP BY**: 회사별 그룹화
- **HAVING**: 그룹의 평균 비교

**실행 과정**:
1. First Bank의 평균 급여 계산
   - 예: 75,000
2. 각 회사별 평균 급여 계산
   - Small Bank: 60,000 → 제외
   - Mega Corp: 80,000 → 포함
   - Tech Inc: 90,000 → 포함
3. 75,000보다 큰 회사들 선택

**WHERE vs HAVING 비교**:
```sql
-- WHERE: 개별 튜플 필터링 (집계 전)
WHERE salary > 50000

-- HAVING: 그룹 필터링 (집계 후)
HAVING AVG(salary) > 75000
```

---

## 문제 3.10 - UPDATE 고급 응용

### 문제 3.10.a
**문제**: ID가 12345인 직원이 Newtown에 살도록 데이터베이스를 수정하시오.

**풀이**:
```sql
UPDATE employee
SET city = 'Newtown'
WHERE ID = '12345';
```

**핵심 개념**:
- **UPDATE ... SET ... WHERE**: 조건부 수정
- 특정 직원의 거주 도시만 변경

---

### 문제 3.10.b
**문제**: First Bank Corporation의 모든 관리자에게 10% 인상을 주되, $100,000을 초과하면 3% 인상만 주시오.

**잘못된 풀이 (순서 의존적)**:
```sql
-- 첫 번째 UPDATE
UPDATE works
SET salary = salary * 1.03
WHERE ID IN (SELECT manager_id FROM manages)
  AND salary * 1.1 > 100000
  AND company_name = 'First Bank Corporation';

-- 두 번째 UPDATE
UPDATE works
SET salary = salary * 1.1
WHERE ID IN (SELECT manager_id FROM manages)
  AND salary * 1.1 <= 100000
  AND company_name = 'First Bank Corporation';
```

**문제점**:
- **실행 순서에 따라 결과가 다름**
- 첫 번째 UPDATE가 salary를 변경하면 두 번째 조건이 달라짐

**예시**:
```
원래 급여: $95,000
1. 3% 인상 먼저 → $97,850 (조건: $95,000 * 1.1 > $100,000 거짓) → 실행 안 됨
2. 10% 인상 → $104,500 (조건: $95,000 * 1.1 = $104,500 > $100,000)

반대 순서:
1. 10% 인상 → $104,500
2. 3% 인상 → $104,500 * 1.03 = $107,635 (잘못된 결과!)
```

**올바른 풀이 (CASE 사용)**:
```sql
UPDATE works
SET salary = salary * (
    CASE
        WHEN salary * 1.1 > 100000 THEN 1.03
        ELSE 1.1
    END
)
WHERE ID IN (SELECT manager_id FROM manages)
  AND company_name = 'First Bank Corporation';
```

**핵심 개념**:
- **CASE in UPDATE**: 조건부 수정
- **원자적 연산**: 한 번에 실행, 순서 무관
- 원래 salary 값으로 조건 판단

**실행 과정**:
1. 각 관리자의 salary 읽기
2. salary * 1.1이 100000 초과인지 확인
   - Yes → salary * 1.03
   - No → salary * 1.1
3. 한 번에 모두 업데이트

**예시**:
```
Manager A: $95,000
  → $95,000 * 1.1 = $104,500 > $100,000
  → salary = $95,000 * 1.03 = $97,850

Manager B: $85,000
  → $85,000 * 1.1 = $93,500 ≤ $100,000
  → salary = $85,000 * 1.1 = $93,500

Manager C: $92,000
  → $92,000 * 1.1 = $101,200 > $100,000
  → salary = $92,000 * 1.03 = $94,760
```

**주의사항**:
- 두 개의 UPDATE 문은 실행 순서 의존성 있음
- CASE 문은 순서 무관, 안전
- 실무에서는 항상 CASE 문 사용 권장

---

## 💡 핵심 정리

### 고급 쿼리 패턴

#### 1. 전체 조건(Universal Quantification)
**"모든 X에 대해" 표현**:
```sql
-- 패턴: A가 모든 B를 포함
WHERE NOT EXISTS (
    (SELECT ... FROM B)
    EXCEPT
    (SELECT ... FROM A WHERE ...)
)
```

**예시**:
- 모든 도시에 지점이 있는 은행
- 모든 과목을 수강한 학생
- 모든 제품을 판매하는 매장

#### 2. 최댓값/최솟값 찾기

**방법 1: ALL 연산자**:
```sql
SELECT ...
WHERE value >= ALL (SELECT value FROM ...)
```

**방법 2: MAX/MIN 함수**:
```sql
SELECT ...
WHERE value = (SELECT MAX(value) FROM ...)
```

**비교**:
- ALL: 여러 값 비교
- MAX/MIN: 단일 값 반환, 더 명확

#### 3. 평균 비교

**개별 vs 그룹 평균**:
```sql
-- 개별: 평균보다 높은 급여
WHERE salary > (SELECT AVG(salary) FROM ...)

-- 그룹: 평균이 더 높은 부서
GROUP BY dept
HAVING AVG(salary) > (SELECT AVG(salary) FROM ...)
```

### UPDATE 고급 기법

#### 1. 조건부 UPDATE
```sql
UPDATE table
SET column = CASE
    WHEN condition1 THEN value1
    WHEN condition2 THEN value2
    ELSE default_value
END
WHERE ...
```

#### 2. 계산된 값으로 UPDATE
```sql
UPDATE table
SET salary = salary * rate
WHERE ...
```

#### 3. 순서 의존성 주의
- ❌ 여러 UPDATE 문: 순서에 따라 결과 다름
- ✅ CASE 문: 순서 무관, 안전

### 실무 팁

#### 성능 최적화
1. **부질의 vs JOIN**: 가능하면 JOIN 사용
2. **EXISTS vs IN**: 큰 데이터는 EXISTS가 빠름
3. **ALL vs MAX**: MAX가 더 효율적

#### 안전한 UPDATE
1. ✅ **트랜잭션 사용**: BEGIN - COMMIT/ROLLBACK
2. ✅ **WHERE 절 필수**: 전체 업데이트 방지
3. ✅ **SELECT로 먼저 확인**: UPDATE 전 대상 확인
4. ✅ **CASE 문 사용**: 조건부 수정은 CASE로

#### 쿼리 작성 순서
1. **데이터 이해**: 스키마, 관계, 제약조건
2. **단계별 분해**: 복잡한 쿼리를 작은 단위로
3. **부질의 먼저**: 내부에서 외부로
4. **테스트**: 각 단계별 결과 확인
5. **최적화**: 성능 개선

### 자주 하는 실수

#### 논리 오류
```sql
-- ❌ 잘못: Small Bank보다 많이 버는 사람 (일부만 비교)
WHERE salary > ANY (SELECT salary FROM works WHERE ...)

-- ✅ 정확: Small Bank의 모든 사람보다 많이 버는 사람
WHERE salary > ALL (SELECT salary FROM works WHERE ...)
```

#### UPDATE 순서 문제
```sql
-- ❌ 순서 의존적
UPDATE ... SET salary = salary * 1.1 WHERE ...
UPDATE ... SET salary = salary * 1.03 WHERE ...

-- ✅ 순서 무관
UPDATE ... SET salary = salary * CASE ... END WHERE ...
```

#### NULL 처리
```sql
-- ❌ NULL 무시
SELECT AVG(salary) FROM ...

-- ✅ NULL 처리
SELECT AVG(COALESCE(salary, 0)) FROM ...
```

---

## 🎯 연습 문제

다음 쿼리를 작성해보세요:

1. **전체 조건**: 모든 과목을 가르친 적 있는 교수 찾기
2. **ALL 연산자**: Physics 학과 교수들보다 적게 버는 모든 교수
3. **나눗셈**: 학생 12345가 수강한 모든 과목을 수강한 학생들
4. **조건부 UPDATE**: 급여가 $50,000 미만이면 20% 인상, 이상이면 10% 인상

---

## 다음 포스트 예고

다음 포스트에서는:
- **JOIN의 다양한 형태** (INNER, OUTER, CROSS)
- **뷰(Views)** 생성과 활용
- **인덱스(Indexes)** 설계
- **저장 프로시저(Stored Procedures)**

를 다룰 예정입니다!
