---
title: "[데이터베이스] 문제 해설 - SQL 기초 Part 2 (Exercises 3.19-3.27)"
date: 2025-01-29 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [sql, null-values, library-database, with-clause, unique-constraint, subqueries, division-operation, problem-solving]
pin: false
math: true
mermaid: false
---

## 개요

이 포스트는 SQL 기초에 관한 연습 문제들(Exercises 3.19-3.27)의 상세한 해설을 제공합니다. NULL 값 처리, `<> ALL`과 `NOT IN` 비교, Library 데이터베이스 쿼리, `UNIQUE` 및 `WITH` 절 변환, 그리고 복잡한 집계 쿼리를 다룹니다.

---

## Exercise 3.19: Reasons for NULL Values

### 문제
> List two reasons why null values might be introduced into the database.

### 해설

NULL 값은 데이터베이스에서 **"값이 없음"** 또는 **"알 수 없음"**을 나타내는 특별한 마커입니다.

#### 이유 1: 값이 존재하지만 알려지지 않음 (Unknown)

데이터가 **존재**하지만 **현재 알 수 없는** 경우입니다.

**예시:**

```sql
-- 신규 채용 직원의 전화번호가 아직 수집되지 않음
INSERT INTO employee (emp_id, name, phone_number, hire_date)
VALUES (1001, 'John Doe', NULL, '2024-01-15');

-- 환자의 혈액형이 아직 검사되지 않음
INSERT INTO patient (patient_id, name, blood_type)
VALUES (5001, 'Jane Smith', NULL);

-- 제품의 재고 수량이 아직 입력되지 않음
INSERT INTO product (product_id, name, stock_quantity)
VALUES ('P123', 'Widget', NULL);
```

**특징:**
- **일시적 상태**: 나중에 실제 값으로 업데이트 가능
- **데이터 수집 중**: 정보가 아직 완전하지 않음
- **업데이트 가능**:
  ```sql
  UPDATE employee
  SET phone_number = '555-1234'
  WHERE emp_id = 1001;
  ```

**실무 시나리오:**
- 고객 등록 시 선택 항목(이메일, 보조 연락처 등)
- 주문 처리 중 배송 추적 번호 (아직 발송 전)
- 설문 조사에서 응답하지 않은 항목

#### 이유 2: 값이 적용되지 않음 (Not Applicable / Inapplicable)

해당 속성이 특정 튜플에 **의미가 없거나 적용되지 않는** 경우입니다.

**예시:**

```sql
-- 미혼자의 배우자 이름 (적용되지 않음)
INSERT INTO person (person_id, name, marital_status, spouse_name)
VALUES (2001, 'Alice Brown', 'Single', NULL);

-- 남성의 maiden_name (결혼 전 성)
INSERT INTO person (person_id, name, gender, maiden_name)
VALUES (2002, 'Bob Wilson', 'Male', NULL);

-- 현금 결제 시 신용카드 번호
INSERT INTO payment (payment_id, amount, payment_method, credit_card_number)
VALUES (3001, 100.00, 'Cash', NULL);

-- 자동차가 없는 사람의 차량 번호
INSERT INTO employee (emp_id, name, has_car, license_plate)
VALUES (4001, 'Carol Lee', FALSE, NULL);
```

**특징:**
- **구조적 NULL**: 논리적으로 항상 NULL일 가능성
- **영구적 상태**: 업데이트해도 의미 없음
- **설계 고려**: 때로는 테이블 분리가 더 나은 설계

**더 나은 설계 예시:**

```sql
-- 방법 1: 별도 테이블 (추천)
CREATE TABLE person (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    marital_status VARCHAR(20)
);

CREATE TABLE married_person_info (
    person_id INT PRIMARY KEY,
    spouse_name VARCHAR(100),
    marriage_date DATE,
    FOREIGN KEY (person_id) REFERENCES person(person_id)
);
-- 미혼자는 married_person_info에 레코드가 없음

-- 방법 2: CHECK 제약조건
CREATE TABLE person (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    marital_status VARCHAR(20),
    spouse_name VARCHAR(100),
    CHECK (
        (marital_status = 'Married' AND spouse_name IS NOT NULL) OR
        (marital_status != 'Married' AND spouse_name IS NULL)
    )
);
```

#### 이유 3 (추가): 값이 누락됨 (Missing)

데이터 입력 오류나 **데이터 수집 실패**로 인한 누락입니다.

**예시:**
- 마이그레이션 중 데이터 손실
- 센서 오류로 수집되지 않은 측정값
- 사용자가 실수로 건너뛴 필수 항목

### NULL 처리의 문제점

#### 1. 비교 연산의 모호성

```sql
-- NULL과의 비교는 UNKNOWN을 반환
SELECT * FROM employee WHERE salary > 50000;
-- salary가 NULL인 직원은 결과에 포함되지 않음

SELECT * FROM employee WHERE salary = NULL;  -- 잘못된 방법!
-- 항상 빈 결과 (NULL = NULL도 UNKNOWN)

SELECT * FROM employee WHERE salary IS NULL;  -- 올바른 방법
```

#### 2. 3-값 논리 (Three-Valued Logic)

SQL은 TRUE, FALSE, **UNKNOWN** 세 가지 논리 값을 사용합니다.

```sql
-- age가 NULL인 경우
WHERE age > 30              -- UNKNOWN
WHERE age > 30 OR age <= 30 -- UNKNOWN (TRUE OR FALSE가 아님!)
WHERE age IS NULL           -- TRUE
```

**논리 연산 진리표:**

| A | B | A AND B | A OR B | NOT A |
|---|---|---------|--------|-------|
| TRUE | UNKNOWN | UNKNOWN | TRUE | FALSE |
| FALSE | UNKNOWN | FALSE | UNKNOWN | TRUE |
| UNKNOWN | UNKNOWN | UNKNOWN | UNKNOWN | UNKNOWN |

#### 3. 집계 함수

```sql
-- salary: [50000, NULL, 60000, NULL, 70000]

SELECT COUNT(*) FROM employee;          -- 5 (모든 행)
SELECT COUNT(salary) FROM employee;     -- 3 (NULL 제외)
SELECT AVG(salary) FROM employee;       -- 60000 (180000/3, NULL 무시)
SELECT SUM(salary) FROM employee;       -- 180000 (NULL 무시)
```

**주의:** NULL은 0이 아닙니다!

```sql
-- 잘못된 가정
SELECT AVG(salary) FROM employee;  -- NULL을 0으로 취급하지 않음

-- NULL을 0으로 명시적 변환
SELECT AVG(COALESCE(salary, 0)) FROM employee;
```

#### 4. DISTINCT와 NULL

```sql
SELECT DISTINCT city FROM employee;
-- NULL 값들은 하나의 그룹으로 취급됨
-- 결과: ['New York', 'Boston', NULL]
```

#### 5. GROUP BY와 NULL

```sql
SELECT dept_name, COUNT(*)
FROM employee
GROUP BY dept_name;
-- NULL dept_name들도 하나의 그룹으로 묶임
```

### NULL 처리 함수

#### COALESCE: 첫 번째 비-NULL 값 반환

```sql
SELECT name, COALESCE(phone_number, email, 'No Contact') AS contact
FROM customer;
-- phone_number가 NULL이면 email, 그것도 NULL이면 'No Contact'
```

#### NULLIF: 두 값이 같으면 NULL 반환

```sql
SELECT NULLIF(department, 'Unknown') AS dept
FROM employee;
-- department가 'Unknown'이면 NULL로 변환
```

#### IS NULL / IS NOT NULL

```sql
SELECT * FROM employee WHERE manager_id IS NULL;      -- 매니저가 없는 직원
SELECT * FROM employee WHERE manager_id IS NOT NULL;  -- 매니저가 있는 직원
```

### NULL 최소화 설계 전략

#### 1. NOT NULL 제약조건 사용

```sql
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,           -- 필수
    email VARCHAR(100) NOT NULL,          -- 필수
    phone_number VARCHAR(20),             -- 선택
    hire_date DATE NOT NULL DEFAULT CURRENT_DATE
);
```

#### 2. 기본값(DEFAULT) 설정

```sql
CREATE TABLE product (
    product_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    stock_quantity INT DEFAULT 0,         -- NULL 대신 0
    status VARCHAR(20) DEFAULT 'Active'   -- NULL 대신 기본 상태
);
```

#### 3. 테이블 분리

```sql
-- 선택적 정보를 별도 테이블로
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employee_contact (
    emp_id INT PRIMARY KEY,
    phone VARCHAR(20),
    email VARCHAR(100),
    FOREIGN KEY (emp_id) REFERENCES employee(emp_id)
);
-- 연락처 정보가 없으면 employee_contact에 레코드 없음
```

#### 4. 특별한 값 사용 (주의해서 사용)

```sql
-- 문자열의 경우 'N/A', 'Unknown' 등
-- 숫자의 경우 -1, 0 등 (의미 있는 경우만)
-- 하지만 NULL의 의미를 명확히 하는 것이 더 나음
```

### 권장 사항

1. **NULL의 의미를 문서화**: Unknown인지 Not Applicable인지 명확히
2. **NOT NULL을 기본으로**: 필수 속성은 NOT NULL 지정
3. **애플리케이션 레벨 검증**: 데이터 입력 시 NULL 방지
4. **적절한 기본값 사용**: 의미 있는 경우 DEFAULT 값 설정
5. **NULL 처리 로직 명시**: 쿼리에서 IS NULL 명시적 처리

---

## Exercise 3.20: `<> ALL` vs `NOT IN`

### 문제
> Show that, in SQL, `<> all` is identical to `not in`.

### 해설

SQL에서 `<> ALL`과 `NOT IN`은 **논리적으로 동등**합니다.

#### 정의

**`<> ALL`:**
- 집합의 **모든** 값과 다른 경우 TRUE
- "어떤 값과도 같지 않음"

**`NOT IN`:**
- 집합에 **포함되지 않음**
- "집합의 멤버가 아님"

#### 수학적 동등성

```
x <> ALL (집합) ≡ x NOT IN (집합)

∀y ∈ 집합: x ≠ y ≡ x ∉ 집합
```

#### 예시 1: 기본 비교

```sql
-- 부서가 'Physics'나 'Music'이 아닌 학생 찾기

-- 방법 1: <> ALL
SELECT name
FROM student
WHERE dept_name <> ALL ('Physics', 'Music');

-- 방법 2: NOT IN
SELECT name
FROM student
WHERE dept_name NOT IN ('Physics', 'Music');
```

**논리:**
- `dept_name <> ALL ('Physics', 'Music')`
- = `dept_name <> 'Physics' AND dept_name <> 'Music'`
- = `NOT (dept_name = 'Physics' OR dept_name = 'Music')`
- = `dept_name NOT IN ('Physics', 'Music')`

#### 예시 2: 부질의 사용

```sql
-- CS-101을 수강하지 않은 학생 찾기

-- 방법 1: <> ALL
SELECT name
FROM student
WHERE ID <> ALL (
    SELECT ID
    FROM takes
    WHERE course_id = 'CS-101'
);

-- 방법 2: NOT IN
SELECT name
FROM student
WHERE ID NOT IN (
    SELECT ID
    FROM takes
    WHERE course_id = 'CS-101'
);
```

**결과:** 두 쿼리는 동일한 결과를 반환합니다.

#### 진리표 증명

**집합: {10, 20, 30}**

| x | x <> 10 | x <> 20 | x <> 30 | x <> ALL | x NOT IN |
|---|---------|---------|---------|----------|----------|
| 10 | FALSE | TRUE | TRUE | FALSE | FALSE |
| 20 | TRUE | FALSE | TRUE | FALSE | FALSE |
| 30 | TRUE | TRUE | FALSE | FALSE | FALSE |
| 40 | TRUE | TRUE | TRUE | TRUE | TRUE |
| 50 | TRUE | TRUE | TRUE | TRUE | TRUE |

**결론:** `x <> ALL (10, 20, 30)` = `x NOT IN (10, 20, 30)`

#### NULL 값 처리의 차이점

**중요:** NULL이 포함된 경우 동작이 미묘하게 다를 수 있습니다.

```sql
-- 집합에 NULL이 포함된 경우
SELECT * FROM student
WHERE dept_name NOT IN ('Physics', 'Music', NULL);
-- 결과: 빈 집합!

SELECT * FROM student
WHERE dept_name <> ALL ('Physics', 'Music', NULL);
-- 결과: 빈 집합!
```

**이유:**
- `dept_name NOT IN (..., NULL)` = `NOT (dept_name = 'Physics' OR dept_name = 'Music' OR dept_name = NULL)`
- `dept_name = NULL`은 UNKNOWN을 반환
- `TRUE OR UNKNOWN = TRUE`이지만 `FALSE OR UNKNOWN = UNKNOWN`
- `NOT UNKNOWN = UNKNOWN`
- WHERE 절은 UNKNOWN을 FALSE로 취급

**예시:**

```sql
-- dept_name = 'Comp. Sci.'인 경우
dept_name = 'Physics'    -- FALSE
dept_name = 'Music'      -- FALSE
dept_name = NULL         -- UNKNOWN

-- OR 연산
FALSE OR FALSE OR UNKNOWN = UNKNOWN

-- NOT 연산
NOT UNKNOWN = UNKNOWN

-- WHERE 절에서 UNKNOWN은 FALSE로 취급되어 제외됨
```

#### 성능 비교

대부분의 현대 DBMS에서는 **동일한 실행 계획**을 생성합니다.

```sql
-- 실행 계획 확인 (PostgreSQL)
EXPLAIN ANALYZE
SELECT name FROM student WHERE dept_name <> ALL ('Physics', 'Music');

EXPLAIN ANALYZE
SELECT name FROM student WHERE dept_name NOT IN ('Physics', 'Music');
-- 두 쿼리의 실행 계획이 동일함
```

#### 대안: NOT EXISTS (NULL-safe)

NULL 문제를 피하려면 `NOT EXISTS` 사용을 권장합니다.

```sql
-- NULL-safe 방법
SELECT s.name
FROM student s
WHERE NOT EXISTS (
    SELECT *
    FROM takes t
    WHERE t.ID = s.ID
      AND t.course_id = 'CS-101'
);
```

#### 다른 비교 연산자

**`= ANY`와 `IN`:**
```sql
-- 동등함
WHERE salary = ANY (50000, 60000, 70000)
WHERE salary IN (50000, 60000, 70000)
```

**`> ALL`:**
```sql
-- 모든 값보다 큰
WHERE salary > ALL (SELECT salary FROM instructor WHERE dept_name = 'Biology')
-- = WHERE salary > MAX(SELECT salary FROM instructor WHERE dept_name = 'Biology')
```

**`< ANY`:**
```sql
-- 적어도 하나보다 작은
WHERE salary < ANY (50000, 60000, 70000)
-- = WHERE salary < MAX(50000, 60000, 70000)
-- = WHERE salary < 70000
```

### 정리

| 표현 | 의미 | 동등 표현 |
|------|------|-----------|
| `x <> ALL (집합)` | 모든 값과 다름 | `x NOT IN (집합)` |
| `x = ANY (집합)` | 적어도 하나와 같음 | `x IN (집합)` |
| `x > ALL (집합)` | 모든 값보다 큼 | `x > MAX(집합)` |
| `x < ALL (집합)` | 모든 값보다 작음 | `x < MIN(집합)` |
| `x > ANY (집합)` | 적어도 하나보다 큼 | `x > MIN(집합)` |
| `x < ANY (집합)` | 적어도 하나보다 작음 | `x < MAX(집합)` |

---

## Exercise 3.21: Library Database Queries

### Library 스키마

**Figure 3.20:**
```
member(memb_no, name, age)
book(isbn, title, authors, publisher)
borrowed(memb_no, isbn, date)
```

### Query A: Members who borrowed McGraw-Hill books

**문제:** Find the member number and name of each member who has borrowed at least one book published by "McGraw-Hill".

**SQL:**
```sql
SELECT DISTINCT m.memb_no, m.name
FROM member m
NATURAL JOIN borrowed br
NATURAL JOIN book b
WHERE b.publisher = 'McGraw-Hill';
```

**대안 (명시적 JOIN):**
```sql
SELECT DISTINCT m.memb_no, m.name
FROM member m
JOIN borrowed br ON m.memb_no = br.memb_no
JOIN book b ON br.isbn = b.isbn
WHERE b.publisher = 'McGraw-Hill';
```

**설명:**
1. `member`와 `borrowed` 조인: 대출 기록 연결
2. `borrowed`와 `book` 조인: 책 정보 연결
3. `WHERE publisher = 'McGraw-Hill'`: McGraw-Hill 출판사만
4. `DISTINCT`: 한 회원이 여러 책을 빌렸어도 한 번만 출력

**예시 데이터:**
```
member:
┌─────────┬───────────┬─────┐
│ memb_no │   name    │ age │
├─────────┼───────────┼─────┤
│ M001    │ Alice     │ 25  │
│ M002    │ Bob       │ 30  │
│ M003    │ Carol     │ 28  │
└─────────┴───────────┴─────┘

book:
┌──────────────┬──────────────────────┬─────────────┬─────────────┐
│     isbn     │        title         │   authors   │  publisher  │
├──────────────┼──────────────────────┼─────────────┼─────────────┤
│ 978-0-07-123 │ Database Concepts    │ Silberschatz│ McGraw-Hill │
│ 978-0-13-456 │ Operating Systems    │ Tanenbaum   │ Pearson     │
│ 978-0-07-789 │ Computer Networks    │ Kurose      │ McGraw-Hill │
└──────────────┴──────────────────────┴─────────────┴─────────────┘

borrowed:
┌─────────┬──────────────┬────────────┐
│ memb_no │     isbn     │    date    │
├─────────┼──────────────┼────────────┤
│ M001    │ 978-0-07-123 │ 2024-01-15 │
│ M001    │ 978-0-07-789 │ 2024-01-20 │
│ M002    │ 978-0-13-456 │ 2024-01-18 │
│ M003    │ 978-0-07-123 │ 2024-01-22 │
└─────────┴──────────────┴────────────┘

결과:
┌─────────┬───────┐
│ memb_no │ name  │
├─────────┼───────┤
│ M001    │ Alice │  (2권의 McGraw-Hill 책 대출)
│ M003    │ Carol │  (1권의 McGraw-Hill 책 대출)
└─────────┴───────┘
```

### Query B: Members who borrowed ALL McGraw-Hill books

**문제:** Find the member number and name of each member who has borrowed every book published by "McGraw-Hill".

**분석:**
- "모든(every)" → **관계 대수의 나눗셈** (Division)
- SQL에서는 **이중 부정**으로 표현

**SQL (이중 부정):**
```sql
SELECT m.memb_no, m.name
FROM member m
WHERE NOT EXISTS (
    -- McGraw-Hill 책 중에서
    SELECT b.isbn
    FROM book b
    WHERE b.publisher = 'McGraw-Hill'
      AND NOT EXISTS (
          -- 이 회원이 빌리지 않은 책이 없다
          SELECT *
          FROM borrowed br
          WHERE br.memb_no = m.memb_no
            AND br.isbn = b.isbn
      )
);
```

**해석:**
- "McGraw-Hill 책 중에서, 이 회원이 빌리지 않은 책이 존재하지 않는다"
- = "모든 McGraw-Hill 책을 빌렸다"

**대안 (GROUP BY HAVING):**
```sql
SELECT m.memb_no, m.name
FROM member m
WHERE m.memb_no IN (
    SELECT br.memb_no
    FROM borrowed br
    JOIN book b ON br.isbn = b.isbn
    WHERE b.publisher = 'McGraw-Hill'
    GROUP BY br.memb_no
    HAVING COUNT(DISTINCT br.isbn) = (
        SELECT COUNT(*)
        FROM book
        WHERE publisher = 'McGraw-Hill'
    )
);
```

**해석:**
- McGraw-Hill 책을 빌린 회원을 그룹화
- 빌린 책의 수가 McGraw-Hill 총 책 수와 같은지 확인

**예시 데이터:**
```
book (McGraw-Hill):
┌──────────────┬──────────────────────┐
│     isbn     │        title         │
├──────────────┼──────────────────────┤
│ 978-0-07-123 │ Database Concepts    │
│ 978-0-07-789 │ Computer Networks    │
│ 978-0-07-456 │ Data Mining          │
└──────────────┴──────────────────────┘
총 3권

borrowed:
┌─────────┬──────────────┬────────────┐
│ memb_no │     isbn     │    date    │
├─────────┼──────────────┼────────────┤
│ M001    │ 978-0-07-123 │ 2024-01-15 │
│ M001    │ 978-0-07-789 │ 2024-01-20 │
│ M001    │ 978-0-07-456 │ 2024-01-25 │  ← 3권 모두
│ M002    │ 978-0-07-123 │ 2024-01-18 │
│ M002    │ 978-0-07-789 │ 2024-01-22 │  ← 2권만
└─────────┴──────────────┴────────────┘

결과:
┌─────────┬───────┐
│ memb_no │ name  │
├─────────┼───────┤
│ M001    │ Alice │  (모든 McGraw-Hill 책 대출)
└─────────┴───────┘
```

### Query C: Members who borrowed >5 books per publisher

**문제:** For each publisher, find the member number and name of each member who has borrowed more than five books of that publisher.

**SQL:**
```sql
SELECT b.publisher, m.memb_no, m.name, COUNT(*) AS books_borrowed
FROM member m
NATURAL JOIN borrowed br
NATURAL JOIN book b
GROUP BY b.publisher, m.memb_no, m.name
HAVING COUNT(*) > 5;
```

**설명:**
1. 세 테이블 조인
2. `GROUP BY publisher, memb_no`: 출판사별, 회원별 그룹화
3. `HAVING COUNT(*) > 5`: 5권 초과만 선택

**예시 데이터:**
```
borrowed (집계):
┌─────────────┬─────────┬───────┬────────┐
│  publisher  │ memb_no │ name  │ count  │
├─────────────┼─────────┼───────┼────────┤
│ McGraw-Hill │ M001    │ Alice │ 7      │ ← 선택
│ McGraw-Hill │ M002    │ Bob   │ 3      │
│ Pearson     │ M001    │ Alice │ 4      │
│ Pearson     │ M003    │ Carol │ 6      │ ← 선택
│ O'Reilly    │ M002    │ Bob   │ 8      │ ← 선택
└─────────────┴─────────┴───────┴────────┘

결과:
┌─────────────┬─────────┬───────┬─────────────────┐
│  publisher  │ memb_no │ name  │ books_borrowed  │
├─────────────┼─────────┼───────┼─────────────────┤
│ McGraw-Hill │ M001    │ Alice │ 7               │
│ Pearson     │ M003    │ Carol │ 6               │
│ O'Reilly    │ M002    │ Bob   │ 8               │
└─────────────┴─────────┴───────┴─────────────────┘
```

**대안 (WITH 절 사용):**
```sql
WITH borrowing_counts AS (
    SELECT b.publisher, br.memb_no, COUNT(*) AS book_count
    FROM borrowed br
    JOIN book b ON br.isbn = b.isbn
    GROUP BY b.publisher, br.memb_no
)
SELECT bc.publisher, m.memb_no, m.name, bc.book_count
FROM borrowing_counts bc
JOIN member m ON bc.memb_no = m.memb_no
WHERE bc.book_count > 5;
```

### Query D: Average books borrowed per member

**문제:** Find the average number of books borrowed per member. Take into account that if a member does not borrow any books, then that member does not appear in the borrowed relation at all, but that member still counts in the average.

**핵심:** 책을 빌리지 않은 회원도 **분모에 포함**해야 합니다.

**잘못된 쿼리:**
```sql
-- 이것은 틀렸습니다!
SELECT AVG(book_count) AS avg_books
FROM (
    SELECT memb_no, COUNT(*) AS book_count
    FROM borrowed
    GROUP BY memb_no
) AS member_books;
-- 책을 빌리지 않은 회원은 제외됨!
```

**올바른 쿼리 (방법 1: LEFT JOIN):**
```sql
SELECT AVG(COALESCE(book_count, 0)) AS avg_books_per_member
FROM (
    SELECT m.memb_no, COUNT(br.isbn) AS book_count
    FROM member m
    LEFT JOIN borrowed br ON m.memb_no = br.memb_no
    GROUP BY m.memb_no
) AS member_books;
```

**올바른 쿼리 (방법 2: 전체 계산):**
```sql
SELECT
    CAST(COUNT(br.isbn) AS FLOAT) / COUNT(DISTINCT m.memb_no) AS avg_books_per_member
FROM member m
LEFT JOIN borrowed br ON m.memb_no = br.memb_no;
```

**설명:**
- `COUNT(br.isbn)`: 총 대출 건수 (NULL 제외)
- `COUNT(DISTINCT m.memb_no)`: 총 회원 수 (책을 안 빌린 회원 포함)
- `CAST(... AS FLOAT)`: 정수 나눗셈 방지

**예시 데이터:**
```
member:
┌─────────┬───────┐
│ memb_no │ name  │
├─────────┼───────┤
│ M001    │ Alice │
│ M002    │ Bob   │
│ M003    │ Carol │
│ M004    │ Dave  │  ← 책을 빌리지 않음
│ M005    │ Eve   │  ← 책을 빌리지 않음
└─────────┴───────┘
총 5명

borrowed:
┌─────────┬──────────────┐
│ memb_no │     isbn     │
├─────────┼──────────────┤
│ M001    │ ...          │  (3권)
│ M001    │ ...          │
│ M001    │ ...          │
│ M002    │ ...          │  (2권)
│ M002    │ ...          │
│ M003    │ ...          │  (5권)
│ M003    │ ...          │
│ M003    │ ...          │
│ M003    │ ...          │
│ M003    │ ...          │
└─────────┴──────────────┘
총 10권

계산:
총 대출: 10권
총 회원: 5명
평균: 10 / 5 = 2.0 책/회원
```

**WITH 절 사용 (가독성 향상):**
```sql
WITH member_book_counts AS (
    SELECT m.memb_no, COUNT(br.isbn) AS book_count
    FROM member m
    LEFT JOIN borrowed br ON m.memb_no = br.memb_no
    GROUP BY m.memb_no
)
SELECT AVG(book_count) AS avg_books_per_member
FROM member_book_counts;
```

**검증 쿼리:**
```sql
-- 각 회원별 대출 수 확인
SELECT m.memb_no, m.name, COUNT(br.isbn) AS book_count
FROM member m
LEFT JOIN borrowed br ON m.memb_no = br.memb_no
GROUP BY m.memb_no, m.name
ORDER BY book_count DESC;
```

---

## Exercise 3.22: Rewriting UNIQUE Clause

### 문제
> Rewrite the where clause `where unique (select title from course)` without using the unique construct.

### UNIQUE의 의미

`UNIQUE` 구문은 부질의 결과에 **중복이 없는지** 확인합니다.

```sql
WHERE UNIQUE (SELECT title FROM course)
-- 모든 과목 제목이 유일한지 확인
```

**TRUE인 경우:**
- 부질의 결과가 빈 집합
- 부질의 결과에 중복된 행이 없음

**FALSE인 경우:**
- 부질의 결과에 중복된 행이 있음

### 재작성 방법

**방법 1: COUNT와 DISTINCT 비교**

```sql
WHERE (SELECT COUNT(*) FROM course) = (SELECT COUNT(DISTINCT title) FROM course)
```

**설명:**
- `COUNT(*)`: 총 과목 수
- `COUNT(DISTINCT title)`: 유일한 제목 수
- 같으면 중복 없음

**방법 2: NOT EXISTS로 중복 찾기**

```sql
WHERE NOT EXISTS (
    SELECT c1.title
    FROM course c1
    JOIN course c2 ON c1.title = c2.title
    WHERE c1.course_id <> c2.course_id
)
```

**설명:**
- 같은 제목을 가진 다른 과목이 존재하지 않음
- `c1.course_id <> c2.course_id`: 자기 자신과의 비교 제외

**방법 3: GROUP BY HAVING**

```sql
WHERE NOT EXISTS (
    SELECT title
    FROM course
    GROUP BY title
    HAVING COUNT(*) > 1
)
```

**설명:**
- 제목별로 그룹화
- 2개 이상인 그룹(중복)이 존재하지 않음

### 예시

```sql
-- 원본 쿼리
SELECT *
FROM some_table
WHERE UNIQUE (SELECT title FROM course);

-- 재작성 1
SELECT *
FROM some_table
WHERE (SELECT COUNT(*) FROM course) = (SELECT COUNT(DISTINCT title) FROM course);

-- 재작성 2
SELECT *
FROM some_table
WHERE NOT EXISTS (
    SELECT c1.title
    FROM course c1, course c2
    WHERE c1.title = c2.title
      AND c1.course_id <> c2.course_id
);
```

**테스트 데이터:**

```
course:
┌───────────┬──────────────────────┐
│ course_id │        title         │
├───────────┼──────────────────────┤
│ CS-101    │ Intro to CS          │
│ CS-201    │ Data Structures      │
│ CS-301    │ Algorithms           │
└───────────┴──────────────────────┘

COUNT(*) = 3
COUNT(DISTINCT title) = 3
→ UNIQUE = TRUE (중복 없음)

course (중복 있는 경우):
┌───────────┬──────────────────────┐
│ course_id │        title         │
├───────────┼──────────────────────┤
│ CS-101    │ Intro to CS          │
│ CS-201    │ Data Structures      │
│ CS-301    │ Intro to CS          │  ← 중복
└───────────┴──────────────────────┘

COUNT(*) = 3
COUNT(DISTINCT title) = 2
→ UNIQUE = FALSE (중복 있음)
```

### NULL 처리

**UNIQUE와 NULL:**
```sql
-- NULL은 서로 다른 것으로 취급
WHERE UNIQUE (SELECT title FROM course)
-- title이 [NULL, NULL]이어도 UNIQUE = TRUE
```

**재작성 시 NULL 처리:**
```sql
-- NULL을 고려한 버전
WHERE (
    SELECT COUNT(*) FROM course WHERE title IS NOT NULL
) = (
    SELECT COUNT(DISTINCT title) FROM course WHERE title IS NOT NULL
)
AND (SELECT COUNT(*) FROM course WHERE title IS NULL) <= 1;
```

---

## Exercise 3.23: Rewriting WITH Clause

### 문제
> Rewrite this query without using the `with` construct.

**원본 쿼리:**
```sql
WITH dept_total (dept_name, value) AS (
    SELECT dept_name, SUM(salary)
    FROM instructor
    GROUP BY dept_name
),
dept_total_avg(value) AS (
    SELECT AVG(value)
    FROM dept_total
)
SELECT dept_name
FROM dept_total, dept_total_avg
WHERE dept_total.value >= dept_total_avg.value;
```

**쿼리 의미:** 학과별 총 급여가 평균 이상인 학과 찾기

### 재작성

**방법 1: 중첩 부질의**

```sql
SELECT dept_name
FROM (
    SELECT dept_name, SUM(salary) AS value
    FROM instructor
    GROUP BY dept_name
) AS dept_total
WHERE value >= (
    SELECT AVG(value)
    FROM (
        SELECT dept_name, SUM(salary) AS value
        FROM instructor
        GROUP BY dept_name
    ) AS dept_total2
);
```

**설명:**
- 내부 부질의 1: 학과별 총 급여 (`dept_total`)
- 내부 부질의 2: 학과별 총 급여의 평균 (중복 계산)
- 외부 쿼리: 평균 이상인 학과 선택

**문제점:** `dept_total` 계산이 **두 번** 수행됨 (비효율적)

**방법 2: HAVING 절 활용**

```sql
SELECT dept_name
FROM instructor
GROUP BY dept_name
HAVING SUM(salary) >= (
    SELECT AVG(dept_salary)
    FROM (
        SELECT SUM(salary) AS dept_salary
        FROM instructor
        GROUP BY dept_name
    ) AS dept_totals
);
```

**설명:**
- 학과별 그룹화하여 총 급여 계산
- HAVING 절에서 평균과 비교

**방법 3: JOIN 사용**

```sql
SELECT dt.dept_name
FROM (
    SELECT dept_name, SUM(salary) AS value
    FROM instructor
    GROUP BY dept_name
) AS dt
CROSS JOIN (
    SELECT AVG(value) AS avg_value
    FROM (
        SELECT dept_name, SUM(salary) AS value
        FROM instructor
        GROUP BY dept_name
    ) AS dept_total
) AS dta
WHERE dt.value >= dta.avg_value;
```

### 예시 데이터

```
instructor:
┌───────┬───────────┬────────┐
│  ID   │ dept_name │ salary │
├───────┼───────────┼────────┤
│ 10101 │ Comp.Sci. │ 65000  │
│ 12121 │ Comp.Sci. │ 70000  │
│ 15151 │ Music     │ 40000  │
│ 22222 │ Physics   │ 95000  │
│ 32343 │ History   │ 60000  │
│ 33456 │ Physics   │ 87000  │
│ 45565 │ History   │ 62000  │
└───────┴───────────┴────────┘

학과별 총 급여:
┌───────────┬────────┐
│ dept_name │ value  │
├───────────┼────────┤
│ Comp.Sci. │ 135000 │
│ Music     │ 40000  │
│ Physics   │ 182000 │
│ History   │ 122000 │
└───────────┴────────┘

평균: (135000 + 40000 + 182000 + 122000) / 4 = 119750

결과 (평균 이상):
┌───────────┐
│ dept_name │
├───────────┤
│ Comp.Sci. │  (135000 >= 119750)
│ Physics   │  (182000 >= 119750)
│ History   │  (122000 >= 119750)
└───────────┘
```

### WITH 절의 장점

1. **가독성**: 복잡한 쿼리를 단계별로 명확히 표현
2. **유지보수성**: 각 단계를 독립적으로 수정 가능
3. **성능**: 많은 DBMS가 CTE를 **한 번만 계산**하여 최적화
4. **재사용**: 같은 부질의를 여러 번 참조 가능

**권장 사항:** 복잡한 쿼리는 WITH 절 사용을 권장합니다.

---

## Exercise 3.24: Accounting Students Advised by Physics Instructors

### 문제
> Using the university schema, write an SQL query to find the name and ID of those Accounting students advised by an instructor in the Physics department.

### University 스키마 (추가)
```
student(ID, name, dept_name, tot_cred)
instructor(ID, name, dept_name, salary)
advisor(s_id, i_id)
```

### SQL 쿼리

```sql
SELECT s.ID, s.name
FROM student s
JOIN advisor a ON s.ID = a.s_id
JOIN instructor i ON a.i_id = i.ID
WHERE s.dept_name = 'Accounting'
  AND i.dept_name = 'Physics';
```

**설명:**
1. `student`와 `advisor` 조인: 학생의 지도교수 연결
2. `advisor`와 `instructor` 조인: 지도교수 정보
3. `WHERE`: Accounting 학생 & Physics 교수

**예시 데이터:**
```
student:
┌───────┬─────────┬────────────┬──────────┐
│  ID   │  name   │ dept_name  │ tot_cred │
├───────┼─────────┼────────────┼──────────┤
│ 12345 │ Shankar │ Comp.Sci.  │ 32       │
│ 19991 │ Brandt  │ History    │ 80       │
│ 23121 │ Chavez  │ Accounting │ 110      │
│ 44553 │ Peltier │ Accounting │ 56       │
└───────┴─────────┴────────────┴──────────┘

instructor:
┌───────┬──────────┬───────────┬────────┐
│  ID   │   name   │ dept_name │ salary │
├───────┼──────────┼───────────┼────────┤
│ 10101 │ Einstein │ Physics   │ 95000  │
│ 12121 │ Wu       │ Finance   │ 90000  │
│ 15151 │ Mozart   │ Music     │ 40000  │
│ 22222 │ Gold     │ Physics   │ 87000  │
└───────┴──────────┴───────────┴────────┘

advisor:
┌──────┬───────┐
│ s_id │ i_id  │
├──────┼───────┤
│ 12345│ 10101 │
│ 19991│ 12121 │
│ 23121│ 10101 │  ← Accounting 학생, Physics 교수
│ 44553│ 22222 │  ← Accounting 학생, Physics 교수
└──────┴───────┘

결과:
┌───────┬─────────┐
│  ID   │  name   │
├───────┼─────────┤
│ 23121 │ Chavez  │
│ 44553 │ Peltier │
└───────┴─────────┘
```

---

## Exercise 3.25: Departments with Budget > Philosophy

### 문제
> Using the university schema, write an SQL query to find the names of those departments whose budget is higher than that of Philosophy. List them in alphabetic order.

### SQL 쿼리

```sql
SELECT dept_name
FROM department
WHERE budget > (
    SELECT budget
    FROM department
    WHERE dept_name = 'Philosophy'
)
ORDER BY dept_name;
```

**설명:**
1. 내부 쿼리: Philosophy 학과의 예산 찾기
2. 외부 쿼리: 그보다 높은 예산을 가진 학과
3. `ORDER BY`: 알파벳 순 정렬

**예시 데이터:**
```
department:
┌────────────┬──────────┬─────────┐
│ dept_name  │ building │ budget  │
├────────────┼──────────┼─────────┤
│ Biology    │ Watson   │ 90000   │
│ Comp.Sci.  │ Taylor   │ 100000  │
│ Elec.Eng.  │ Taylor   │ 85000   │
│ Finance    │ Painter  │ 120000  │
│ History    │ Painter  │ 50000   │
│ Music      │ Packard  │ 80000   │
│ Philosophy │ Packard  │ 60000   │ ← 기준
│ Physics    │ Watson   │ 70000   │
└────────────┴──────────┴─────────┘

Philosophy 예산: 60000

결과 (budget > 60000, 알파벳 순):
┌────────────┐
│ dept_name  │
├────────────┤
│ Biology    │  (90000)
│ Comp.Sci.  │  (100000)
│ Elec.Eng.  │  (85000)
│ Finance    │  (120000)
│ Music      │  (80000)
│ Physics    │  (70000)
└────────────┘
```

---

## Exercise 3.26: Students Who Retook Courses

### 문제
> Using the university schema, use SQL to do the following: For each student who has retaken a course at least twice (i.e., the student has taken the course at least three times), show the course ID and the student's ID. Please display your results in order of course ID and do not display duplicate rows.

### SQL 쿼리

```sql
SELECT DISTINCT course_id, ID
FROM takes
GROUP BY course_id, ID
HAVING COUNT(*) >= 3
ORDER BY course_id, ID;
```

**설명:**
1. `GROUP BY course_id, ID`: 학생별, 과목별 그룹화
2. `HAVING COUNT(*) >= 3`: 3회 이상 수강
3. `DISTINCT`: 중복 제거 (GROUP BY로 이미 유일하지만 명시적)
4. `ORDER BY`: 과목 ID, 학생 ID 순 정렬

**예시 데이터:**
```
takes:
┌───────┬───────────┬────────┬──────────┬──────┬───────┐
│  ID   │ course_id │ sec_id │ semester │ year │ grade │
├───────┼───────────┼────────┼──────────┼──────┼───────┤
│ 12345 │ CS-101    │ 1      │ Fall     │ 2017 │ C     │
│ 12345 │ CS-101    │ 1      │ Spring   │ 2018 │ B     │  ← 재수강
│ 12345 │ CS-101    │ 1      │ Fall     │ 2018 │ A     │  ← 재수강
│ 23121 │ CS-190    │ 1      │ Spring   │ 2017 │ B     │
│ 23121 │ CS-190    │ 2      │ Fall     │ 2017 │ C     │  ← 재수강
│ 23121 │ CS-190    │ 1      │ Spring   │ 2018 │ A     │  ← 재수강
│ 44553 │ PHY-101   │ 1      │ Fall     │ 2017 │ B     │
│ 44553 │ PHY-101   │ 1      │ Spring   │ 2018 │ B     │  ← 재수강
└───────┴───────────┴────────┴──────────┴──────┴───────┘

그룹화 및 카운트:
┌───────┬───────────┬───────┐
│  ID   │ course_id │ count │
├───────┼───────────┼───────┤
│ 12345 │ CS-101    │ 3     │  ← 선택 (≥3)
│ 23121 │ CS-190    │ 3     │  ← 선택 (≥3)
│ 44553 │ PHY-101   │ 2     │
└───────┴───────────┴───────┘

결과:
┌───────────┬───────┐
│ course_id │  ID   │
├───────────┼───────┤
│ CS-101    │ 12345 │
│ CS-190    │ 23121 │
└───────────┴───────┘
```

---

## Exercise 3.27: Students Who Retook 3+ Distinct Courses

### 문제
> Using the university schema, write an SQL query to find the IDs of those students who have retaken at least three distinct courses at least once (i.e., the student has taken the course at least two times).

### SQL 쿼리

```sql
SELECT ID
FROM takes
GROUP BY ID, course_id
HAVING COUNT(*) >= 2  -- 각 과목을 2회 이상 수강
GROUP BY ID
HAVING COUNT(DISTINCT course_id) >= 3;  -- 이러한 과목이 3개 이상
```

**오류:** SQL은 이중 GROUP BY를 지원하지 않습니다!

**올바른 쿼리:**

```sql
SELECT ID
FROM (
    SELECT ID, course_id
    FROM takes
    GROUP BY ID, course_id
    HAVING COUNT(*) >= 2  -- 2회 이상 수강한 과목
) AS retaken_courses
GROUP BY ID
HAVING COUNT(DISTINCT course_id) >= 3;  -- 이러한 과목이 3개 이상
```

**설명:**
1. **내부 쿼리**: 각 학생이 2회 이상 수강한 과목 찾기
2. **외부 쿼리**: 그러한 과목이 3개 이상인 학생

**WITH 절 사용 (더 읽기 쉬움):**

```sql
WITH retaken_courses AS (
    SELECT ID, course_id
    FROM takes
    GROUP BY ID, course_id
    HAVING COUNT(*) >= 2
)
SELECT ID
FROM retaken_courses
GROUP BY ID
HAVING COUNT(DISTINCT course_id) >= 3;
```

**예시 데이터:**
```
takes:
┌───────┬───────────┬──────────┬──────┐
│  ID   │ course_id │ semester │ year │
├───────┼───────────┼──────────┼──────┤
│ 12345 │ CS-101    │ Fall     │ 2017 │
│ 12345 │ CS-101    │ Spring   │ 2018 │  ← CS-101 재수강
│ 12345 │ CS-190    │ Fall     │ 2017 │
│ 12345 │ CS-190    │ Spring   │ 2018 │  ← CS-190 재수강
│ 12345 │ CS-315    │ Fall     │ 2018 │
│ 12345 │ CS-315    │ Spring   │ 2019 │  ← CS-315 재수강
│ 12345 │ CS-347    │ Fall     │ 2019 │
│ 12345 │ CS-347    │ Spring   │ 2020 │  ← CS-347 재수강 (4개)
│ 23121 │ FIN-201   │ Spring   │ 2017 │
│ 23121 │ FIN-201   │ Fall     │ 2017 │  ← FIN-201 재수강
│ 23121 │ FIN-301   │ Spring   │ 2018 │
│ 23121 │ FIN-301   │ Fall     │ 2018 │  ← FIN-301 재수강 (2개만)
└───────┴───────────┴──────────┴──────┘

단계 1: 2회 이상 수강한 과목
┌───────┬───────────┐
│  ID   │ course_id │
├───────┼───────────┤
│ 12345 │ CS-101    │
│ 12345 │ CS-190    │
│ 12345 │ CS-315    │
│ 12345 │ CS-347    │
│ 23121 │ FIN-201   │
│ 23121 │ FIN-301   │
└───────┴───────────┘

단계 2: 3개 이상인 학생
┌───────┬───────┐
│  ID   │ count │
├───────┼───────┤
│ 12345 │ 4     │  ← 선택 (≥3)
│ 23121 │ 2     │
└───────┴───────┘

결과:
┌───────┐
│  ID   │
├───────┤
│ 12345 │
└───────┘
```

---

## 핵심 개념 정리

### 1. NULL 값 처리
- **의미**: Unknown, Not Applicable, Missing
- **비교**: `IS NULL`, `IS NOT NULL` (= NULL은 잘못됨)
- **집계**: NULL 제외됨 (COUNT 제외)
- **3-값 논리**: TRUE, FALSE, UNKNOWN

### 2. 양화사 (Quantifiers)
- **ALL**: 모든 값과 비교
- **ANY/SOME**: 적어도 하나와 비교
- **NOT IN**: `<> ALL`과 동등

### 3. Division 연산 (모든/every)
- **이중 부정**: NOT EXISTS ... NOT EXISTS
- **GROUP BY HAVING**: 개수 비교

### 4. UNIQUE와 DISTINCT
- **UNIQUE**: 부질의 결과에 중복 없음
- **DISTINCT**: 결과에서 중복 제거

### 5. WITH 절 (CTE)
- **가독성**: 복잡한 쿼리를 단계별로
- **재사용**: 같은 부질의 여러 번 참조
- **성능**: 한 번만 계산 (대부분의 DBMS)

### 6. 집계와 그룹화
- **GROUP BY**: 그룹별 집계
- **HAVING**: 그룹 필터링
- **COUNT(*)** vs **COUNT(column)**: NULL 처리 차이

---

## 다음 단계

**Part 3 (Exercises 3.28-3.35)에서 다룰 내용:**
- 복잡한 Division 쿼리 (모든 과목 가르치는 교수)
- 고급 문자열 매칭
- NULL과 집계 함수의 상호작용
- 최대 등록 섹션 찾기
- 복잡한 GROUP BY와 HAVING 조합

---

## 참고 자료

- Database System Concepts (7th Edition) - Silberschatz, Korth, Sudarshan
- SQL Standard (ISO/IEC 9075)
- "SQL and Relational Theory" - C.J. Date
- PostgreSQL Documentation on NULL Handling
