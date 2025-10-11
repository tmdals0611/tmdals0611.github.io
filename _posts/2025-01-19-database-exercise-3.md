---
title: "[데이터베이스] 문제 해설 #3 - SQL 기초와 활용"
date: 2025-01-19 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [sql, problem-solving, exercise, select, insert, update, delete, subquery, aggregate, case]
pin: false
math: true
mermaid: false
---

이번 포스트에서는 SQL 기초 문법부터 고급 쿼리 작성까지 다양한 SQL 문제들을 다룹니다. University, Insurance, Banking, Employee 등 여러 도메인의 데이터베이스 스키마를 사용하여 실전 SQL 작성 능력을 향상시킬 수 있습니다.

## 📚 사용할 데이터베이스 스키마

### University Schema
```sql
course(course_id, title, dept_name, credits)
instructor(ID, name, dept_name, salary)
section(course_id, sec_id, semester, year, building, room_number, time_slot_id)
teaches(ID, course_id, sec_id, semester, year)
takes(ID, course_id, sec_id, semester, year, grade)
student(ID, name, dept_name, tot_cred)
```

### Insurance Database
```sql
person(driver_id, name, address)
car(license_plate, model, year)
accident(report_number, year, location)
owns(driver_id, license_plate)
participated(report_number, license_plate, driver_id, damage_amount)
```

### Banking Database
```sql
branch(branch_name, branch_city, assets)
customer(ID, customer_name, customer_street, customer_city)
loan(loan_number, branch_name, amount)
borrower(ID, loan_number)
account(account_number, branch_name, balance)
depositor(ID, account_number)
```

### Employee Database
```sql
employee(ID, person_name, street, city)
works(ID, company_name, salary)
company(company_name, city)
manages(ID, manager_id)
```

---

## 문제 3.1 - University 데이터베이스 기본 쿼리

### 문제 3.1.a
**문제**: Comp. Sci. 학과에서 제공하는 3학점 과목의 제목을 찾으시오.

**풀이**:
```sql
SELECT title
FROM course
WHERE dept_name = 'Comp. Sci.' AND credits = 3;
```

**핵심 개념**:
- 기본 SELECT-FROM-WHERE 구조
- 다중 조건을 AND로 결합

---

### 문제 3.1.b
**문제**: Einstein이라는 이름의 교수에게 수업을 들은 모든 학생의 ID를 찾으시오. 중복이 없어야 합니다.

**풀이**:
```sql
SELECT DISTINCT takes.ID
FROM takes, instructor, teaches
WHERE takes.course_id = teaches.course_id
  AND takes.sec_id = teaches.sec_id
  AND takes.semester = teaches.semester
  AND takes.year = teaches.year
  AND teaches.ID = instructor.ID
  AND instructor.name = 'Einstein';
```

**핵심 개념**:
- **DISTINCT**: 중복 제거
- **다중 테이블 조인**: takes, instructor, teaches 테이블 연결
- **조인 조건**: 강좌 정보(course_id, sec_id, semester, year)로 매칭

**주의사항**:
- 여러 테이블을 조인할 때는 정확한 조인 조건이 필수
- DISTINCT를 빼먹으면 같은 학생이 여러 번 나올 수 있음

---

### 문제 3.1.c
**문제**: 가장 높은 급여를 받는 교수의 급여를 찾으시오.

**풀이**:
```sql
SELECT MAX(salary)
FROM instructor;
```

**핵심 개념**:
- **집계 함수 MAX()**: 최댓값 찾기
- 단일 값 반환

---

### 문제 3.1.d
**문제**: 가장 높은 급여를 받는 모든 교수를 찾으시오 (동일한 최고 급여를 받는 교수가 여러 명일 수 있음).

**풀이**:
```sql
SELECT ID, name
FROM instructor
WHERE salary = (SELECT MAX(salary) FROM instructor);
```

**핵심 개념**:
- **중첩 부질의(Nested Subquery)**: WHERE 절에 SELECT 문 사용
- 부질의가 단일 값을 반환하므로 = 연산자 사용 가능

**실행 순서**:
1. 내부 쿼리: MAX(salary) 계산 (예: 95000)
2. 외부 쿼리: salary = 95000인 교수들 조회

---

### 문제 3.1.e
**문제**: 2017년 가을 학기에 개설된 각 섹션의 등록 인원을 찾으시오.

**풀이 1 (상관 부질의 사용)**:
```sql
SELECT course_id, sec_id,
       (SELECT COUNT(ID)
        FROM takes
        WHERE takes.year = section.year
          AND takes.semester = section.semester
          AND takes.course_id = section.course_id
          AND takes.sec_id = section.sec_id) AS enrollment
FROM section
WHERE semester = 'Fall' AND year = 2017;
```

**핵심 개념**:
- **상관 부질의(Correlated Subquery)**: 외부 쿼리의 section 테이블 참조
- **COUNT()**: 등록 학생 수 계산
- **중요**: 등록 학생이 없는 섹션도 0으로 표시됨

**풀이 2 (JOIN 사용 - 주의 필요)**:
```sql
SELECT takes.course_id, takes.sec_id, COUNT(ID)
FROM section, takes
WHERE takes.course_id = section.course_id
  AND takes.sec_id = section.sec_id
  AND takes.semester = section.semester
  AND takes.year = section.year
  AND takes.semester = 'Fall'
  AND takes.year = 2017
GROUP BY takes.course_id, takes.sec_id;
```

**주의사항**:
- 이 방법은 등록 학생이 없는 섹션을 결과에서 제외
- 모든 섹션을 포함하려면 OUTER JOIN 필요 (4장에서 학습)

---

### 문제 3.1.f
**문제**: 2017년 가을 학기 모든 섹션 중 최대 등록 인원을 찾으시오.

**풀이**:
```sql
SELECT MAX(enrollment)
FROM (SELECT COUNT(ID) AS enrollment
      FROM section, takes
      WHERE takes.year = section.year
        AND takes.semester = section.semester
        AND takes.course_id = section.course_id
        AND takes.sec_id = section.sec_id
        AND takes.semester = 'Fall'
        AND takes.year = 2017
      GROUP BY takes.course_id, takes.sec_id) AS sec_enrollment;
```

**핵심 개념**:
- **FROM 절의 부질의**: 부질의 결과를 테이블처럼 사용
- **중첩 집계**: 먼저 COUNT()로 각 섹션의 등록 인원을 구한 후, MAX()로 최댓값 계산

**대안 (WITH 절 사용)**:
```sql
WITH sec_enrollment AS (
    SELECT COUNT(ID) AS enrollment
    FROM section, takes
    WHERE ...
    GROUP BY takes.course_id, takes.sec_id
)
SELECT MAX(enrollment) FROM sec_enrollment;
```

**미묘한 문제점**:
- 등록 학생이 하나도 없으면 결과가 빈 집합
- 이전 문제(3.1.e)의 부질의 방식을 사용하면 0을 반환

---

### 문제 3.1.g
**문제**: 2017년 가을 학기에 최대 등록 인원을 가진 섹션들을 찾으시오.

**풀이 (WITH 절 사용)**:
```sql
WITH sec_enrollment AS (
    SELECT takes.course_id, takes.sec_id, COUNT(ID) AS enrollment
    FROM section, takes
    WHERE takes.year = section.year
      AND takes.semester = section.semester
      AND takes.course_id = section.course_id
      AND takes.sec_id = section.sec_id
      AND takes.semester = 'Fall'
      AND takes.year = 2017
    GROUP BY takes.course_id, takes.sec_id
)
SELECT course_id, sec_id
FROM sec_enrollment
WHERE enrollment = (SELECT MAX(enrollment) FROM sec_enrollment);
```

**핵심 개념**:
- **WITH 절**: 임시 뷰 정의로 쿼리 가독성 향상
- 등록 인원 계산 로직을 재사용

**WITH 절 없이 작성하면**:
- 부질의가 두 번 반복되어 가독성 저하
- WITH 절로 중복 제거 가능

---

## 문제 3.2 - 학점(GPA) 계산

### 배경 지식
`grade_points(grade, points)` 테이블:
- 'A' → 4.0
- 'A-' → 3.7
- 'B+' → 3.3
- 'B' → 3.0 등

**학점 계산 공식**:
- 과목별 점수 = credits × points
- GPA = (총 학점) / (총 학점 수)

---

### 문제 3.2.a
**문제**: ID가 12345인 학생이 모든 과목에서 얻은 총 학점을 계산하시오.

**풀이**:
```sql
SELECT SUM(credits * points)
FROM takes, course, grade_points
WHERE takes.grade = grade_points.grade
  AND takes.course_id = course.course_id
  AND ID = '12345';
```

**문제점**:
- 수강한 과목이 없는 학생은 결과에 나타나지 않음
- 기대하는 결과: 0

**개선된 풀이**:
```sql
(SELECT SUM(credits * points)
 FROM takes, course, grade_points
 WHERE takes.grade = grade_points.grade
   AND takes.course_id = course.course_id
   AND ID = '12345')
UNION
(SELECT 0
 FROM student
 WHERE ID = '12345'
   AND NOT EXISTS (SELECT * FROM takes WHERE ID = '12345'));
```

**핵심 개념**:
- **UNION**: 두 쿼리 결과 합치기
- **NOT EXISTS**: 수강 이력이 없는지 확인
- OUTER JOIN을 사용하면 더 간단 (4장에서 학습)

---

### 문제 3.2.b
**문제**: 위 학생의 GPA를 계산하시오.

**풀이**:
```sql
SELECT SUM(credits * points) / SUM(credits) AS GPA
FROM takes, course, grade_points
WHERE takes.grade = grade_points.grade
  AND takes.course_id = course.course_id
  AND ID = '12345';
```

**추가 고려사항**:
- 수강 과목이 없으면 SUM(credits) = 0 → 0으로 나누기 오류
- 이 경우 GPA는 NULL로 정의하는 것이 적절

**NULL 처리 풀이**:
```sql
SELECT SUM(credits * points) / SUM(credits) AS GPA
FROM takes, course, grade_points
WHERE takes.grade = grade_points.grade
  AND takes.course_id = course.course_id
  AND ID = '12345'
UNION
(SELECT NULL AS GPA
 FROM student
 WHERE ID = '12345'
   AND NOT EXISTS (SELECT * FROM takes WHERE ID = '12345'));
```

---

### 문제 3.2.c
**문제**: 각 학생의 ID와 GPA를 찾으시오.

**풀이**:
```sql
SELECT ID, SUM(credits * points) / SUM(credits) AS GPA
FROM takes, course, grade_points
WHERE takes.grade = grade_points.grade
  AND takes.course_id = course.course_id
GROUP BY ID;
```

**모든 학생 포함 (수강 이력 없는 학생도)**:
```sql
SELECT ID, SUM(credits * points) / SUM(credits) AS GPA
FROM takes, course, grade_points
WHERE takes.grade = grade_points.grade
  AND takes.course_id = course.course_id
GROUP BY ID
UNION
(SELECT ID, NULL AS GPA
 FROM student
 WHERE NOT EXISTS (SELECT * FROM takes WHERE takes.ID = student.ID));
```

---

### 문제 3.2.d
**문제**: grade에 NULL 값이 있을 경우 위 쿼리들이 올바르게 작동하는가?

**답변**:
**작동합니다!** 그 이유는:

1. **조인 조건**: `takes.grade = grade_points.grade`
2. **NULL의 특성**: NULL = NULL은 FALSE
3. **결과**: grade가 NULL인 튜플은 조인에서 제외됨

**장점**:
- NULL grade를 가진 과목의 학점(credits)도 자동으로 제외
- 정확한 GPA 계산 가능

**예시**:
```
Student 12345:
- CS101: 3 credits, A (4.0) → 12 points
- CS102: 3 credits, NULL → 제외됨
- CS103: 4 credits, B (3.0) → 12 points

GPA = (12 + 12) / (3 + 4) = 24 / 7 ≈ 3.43
```

---

## 문제 3.3 - INSERT, DELETE, UPDATE

### 문제 3.3.a
**문제**: Comp. Sci. 학과의 모든 교수 급여를 10% 인상하시오.

**풀이**:
```sql
UPDATE instructor
SET salary = salary * 1.10
WHERE dept_name = 'Comp. Sci.';
```

**핵심 개념**:
- **UPDATE**: 기존 데이터 수정
- **산술 연산**: salary * 1.10 (10% 인상)
- **WHERE 절**: 조건에 맞는 튜플만 수정

---

### 문제 3.3.b
**문제**: 한 번도 개설되지 않은 모든 과목을 삭제하시오 (section 테이블에 없는 과목).

**풀이**:
```sql
DELETE FROM course
WHERE course_id NOT IN (SELECT course_id FROM section);
```

**핵심 개념**:
- **DELETE**: 튜플 삭제
- **NOT IN**: 부질의 결과에 포함되지 않는 것
- 개설 이력이 없는 과목만 삭제

---

### 문제 3.3.c
**문제**: tot_cred > 100인 모든 학생을 같은 학과의 교수로 추가하시오 (급여 $10,000).

**풀이**:
```sql
INSERT INTO instructor
SELECT ID, name, dept_name, 10000
FROM student
WHERE tot_cred > 100;
```

**핵심 개념**:
- **INSERT INTO ... SELECT**: 조회 결과를 직접 삽입
- SELECT 결과의 속성 순서가 INSERT 테이블의 속성 순서와 일치해야 함

**주의사항**:
- ID가 instructor의 기본키면 중복 체크 필요
- dept_name이 외래키면 참조 무결성 확인 필요

---

## 문제 3.4 - Insurance 데이터베이스

### 문제 3.4.a
**문제**: 2017년에 사고에 연루된 차량을 소유한 사람의 총 수를 찾으시오.

**풀이**:
```sql
SELECT COUNT(DISTINCT person.driver_id)
FROM accident, participated, person, owns
WHERE accident.report_number = participated.report_number
  AND owns.driver_id = person.driver_id
  AND owns.license_plate = participated.license_plate
  AND year = 2017;
```

**핵심 개념**:
- **COUNT(DISTINCT)**: 중복 제거 후 카운트
- **중요**: 여러 사고에 연루된 사람도 한 번만 카운트
- **소유자 찾기**: participated의 driver_id가 아닌 owns의 driver_id 사용

**주의사항**:
- 사고 당시 운전자 ≠ 소유자일 수 있음
- 문제에서 "소유자" 수를 요구하므로 owns 테이블 사용 필수

---

### 문제 3.4.b
**문제**: ID가 12345인 사람이 소유한 모든 2010년식 차량을 삭제하시오.

**풀이**:
```sql
DELETE FROM car
WHERE year = 2010
  AND license_plate IN (
      SELECT license_plate
      FROM owns
      WHERE driver_id = '12345'
  );
```

**핵심 개념**:
- **부질의로 대상 식별**: 먼저 소유 차량 찾기
- **IN 연산자**: 부질의 결과 집합에 포함 여부 확인

**주의사항**:
- car 테이블만 삭제됨
- owns, participated, accident의 관련 레코드는 그대로 유지
- 참조 무결성 제약이 있다면 삭제 실패 가능

---

## 문제 3.5 - CASE 문을 이용한 학점 부여

### 배경
`marks(ID, score)` 테이블에서 점수에 따라 학점 부여:
- score < 40: F
- 40 ≤ score < 60: C
- 60 ≤ score < 80: B
- 80 ≤ score: A

---

### 문제 3.5.a
**문제**: 각 학생의 학점을 표시하시오.

**풀이**:
```sql
SELECT ID,
       CASE
           WHEN score < 40 THEN 'F'
           WHEN score < 60 THEN 'C'
           WHEN score < 80 THEN 'B'
           ELSE 'A'
       END AS grade
FROM marks;
```

**핵심 개념**:
- **CASE 문**: 조건에 따라 다른 값 반환
- **WHEN-THEN-ELSE 구조**: if-else와 유사
- **순차 평가**: 위에서 아래로 조건 검사

**주의사항**:
- 조건의 순서가 중요! (score < 60은 이미 score < 40이 아닌 경우만 검사)

---

### 문제 3.5.b
**문제**: 각 학점별 학생 수를 찾으시오.

**풀이**:
```sql
WITH grades AS (
    SELECT ID,
           CASE
               WHEN score < 40 THEN 'F'
               WHEN score < 60 THEN 'C'
               WHEN score < 80 THEN 'B'
               ELSE 'A'
           END AS grade
    FROM marks
)
SELECT grade, COUNT(ID)
FROM grades
GROUP BY grade;
```

**대안 (WITH 절 없이)**:
```sql
SELECT grade, COUNT(ID)
FROM (
    SELECT ID,
           CASE
               WHEN score < 40 THEN 'F'
               WHEN score < 60 THEN 'C'
               WHEN score < 80 THEN 'B'
               ELSE 'A'
           END AS grade
    FROM marks
) AS grades
GROUP BY grade;
```

**핵심 개념**:
- **WITH 절**: 가독성 향상, 부질의 재사용
- **GROUP BY**: 학점별로 그룹화
- **COUNT()**: 각 그룹의 학생 수

---

## 문제 3.6 - 대소문자 구분 없는 검색

**문제**: 학과명에 "sci"가 포함된 학과를 대소문자 구분 없이 찾으시오.

**풀이**:
```sql
SELECT dept_name
FROM department
WHERE LOWER(dept_name) LIKE '%sci%';
```

**핵심 개념**:
- **LOWER()**: 문자열을 소문자로 변환
- **LIKE**: 패턴 매칭
- **%**: 0개 이상의 임의 문자

**매칭 예시**:
- "Computer Science" → "computer science" → 매칭!
- "Physical Sciences" → "physical sciences" → 매칭!
- "SCIENCE" → "science" → 매칭!

**대안**:
```sql
SELECT dept_name
FROM department
WHERE UPPER(dept_name) LIKE '%SCI%';
```

---

## 문제 3.7 - Cartesian Product와 빈 테이블

**쿼리**:
```sql
SELECT p.a1
FROM p, r1, r2
WHERE p.a1 = r1.a1 OR p.a1 = r2.a1;
```

**질문**: 이 쿼리가 r1 또는 r2에 있는 p.a1 값을 선택하는 조건은?

**답변**:

이 쿼리는 **r1과 r2가 모두 비어있지 않을 때만** 올바르게 작동합니다.

**이유**:
1. **Cartesian Product**: p × r1 × r2
2. **r1 또는 r2가 비어있으면**:
   - Cartesian Product 결과 = 빈 집합
   - WHERE 절 평가 불가
   - 최종 결과 = 빈 집합

**케이스별 분석**:
- **r1, r2 모두 비어있지 않음**: 정상 작동 ✅
- **r1만 비어있음**: 빈 결과 ❌
- **r2만 비어있음**: 빈 결과 ❌
- **p가 비어있음**: 빈 결과 (당연함)

**올바른 쿼리 (UNION 사용)**:
```sql
(SELECT p.a1 FROM p, r1 WHERE p.a1 = r1.a1)
UNION
(SELECT p.a1 FROM p, r2 WHERE p.a1 = r2.a1);
```

---

## 문제 3.8 - Banking 데이터베이스

### 문제 3.8.a
**문제**: 계좌는 있지만 대출은 없는 고객의 ID를 찾으시오.

**풀이**:
```sql
(SELECT ID FROM depositor)
EXCEPT
(SELECT ID FROM borrower);
```

**핵심 개념**:
- **EXCEPT (차집합)**: A에 있지만 B에 없는 것
- depositor에 있지만 borrower에 없는 ID

**대안 (NOT IN 사용)**:
```sql
SELECT ID
FROM depositor
WHERE ID NOT IN (SELECT ID FROM borrower);
```

---

### 문제 3.8.b
**문제**: 고객 12345와 같은 거리, 같은 도시에 사는 고객의 ID를 찾으시오.

**풀이**:
```sql
SELECT F.ID
FROM customer AS F, customer AS S
WHERE F.customer_street = S.customer_street
  AND F.customer_city = S.customer_city
  AND S.ID = '12345';
```

**핵심 개념**:
- **Self-Join**: 같은 테이블을 두 번 조인
- **별칭(Alias)**: F(First), S(Second)로 구분
- 자기 자신(12345)도 결과에 포함됨

**자기 자신 제외**:
```sql
SELECT F.ID
FROM customer AS F, customer AS S
WHERE F.customer_street = S.customer_street
  AND F.customer_city = S.customer_city
  AND S.ID = '12345'
  AND F.ID <> '12345';
```

---

### 문제 3.8.c
**문제**: Harrison에 사는 고객이 계좌를 가지고 있는 지점의 이름을 찾으시오.

**풀이**:
```sql
SELECT DISTINCT branch_name
FROM account, depositor, customer
WHERE customer.ID = depositor.ID
  AND depositor.account_number = account.account_number
  AND customer_city = 'Harrison';
```

**핵심 개념**:
- **3-way Join**: customer → depositor → account
- **DISTINCT**: 같은 지점이 여러 번 나오는 것 방지

**조인 순서**:
1. customer와 depositor를 ID로 조인
2. depositor와 account를 account_number로 조인
3. customer_city = 'Harrison' 필터링

---

## 문제 3.9 - Employee 데이터베이스 (고급 쿼리)

이 문제는 내용이 많아 Part 2로 분리하겠습니다.

---

## 💡 핵심 정리

### SQL 기본 개념
1. **SELECT-FROM-WHERE**: 기본 쿼리 구조
2. **DISTINCT**: 중복 제거
3. **집계 함수**: COUNT(), SUM(), AVG(), MAX(), MIN()
4. **GROUP BY**: 그룹화
5. **HAVING**: 그룹 필터링

### 고급 SQL
1. **중첩 부질의**: WHERE 절, FROM 절, SELECT 절에서 사용
2. **상관 부질의**: 외부 쿼리 참조
3. **WITH 절**: 임시 뷰 정의
4. **CASE 문**: 조건부 값 반환
5. **집합 연산**: UNION, EXCEPT, INTERSECT

### 데이터 수정
1. **INSERT INTO ... SELECT**: 조회 결과 삽입
2. **UPDATE**: 기존 데이터 수정
3. **DELETE**: 데이터 삭제

### 자주 하는 실수
1. ❌ DISTINCT 빼먹기 → 중복 결과
2. ❌ NULL 처리 안 함 → 예상과 다른 결과
3. ❌ 조인 조건 누락 → Cartesian Product
4. ❌ 집계 함수에 GROUP BY 안 쓰기 → 오류

### 실무 팁
1. ✅ WHERE 절보다 JOIN ON이 명확
2. ✅ WITH 절로 복잡한 쿼리 단순화
3. ✅ 항상 NULL 케이스 고려
4. ✅ COUNT(DISTINCT)로 중복 방지
5. ✅ 부질의보다 JOIN이 성능 좋음 (대부분)

---

다음 포스트 Part 2에서는 Employee 데이터베이스의 고급 쿼리들(문제 3.9, 3.10)을 다룹니다!
