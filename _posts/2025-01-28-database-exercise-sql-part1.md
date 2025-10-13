---
title: "[데이터베이스] 문제 해설 - SQL 기초 Part 1 (Exercises 3.11-3.18)"
date: 2025-01-28 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [sql, query-writing, ddl, dml, aggregation, joins, subqueries, foreign-keys, problem-solving]
pin: false
math: true
mermaid: false
---

## 개요

이 포스트는 SQL 기초에 관한 연습 문제들(Exercises 3.11-3.18)의 상세한 해설을 제공합니다. University, Insurance, Bank, Employee 스키마를 사용하여 SELECT, INSERT, UPDATE, DELETE 등 기본적인 SQL 연산과 DDL(Data Definition Language) 작성을 다룹니다.

---

## Exercise 3.11: University Schema - Basic Queries

### University 스키마
```sql
student(ID, name, dept_name, tot_cred)
instructor(ID, name, dept_name, salary)
department(dept_name, building, budget)
course(course_id, title, dept_name, credits)
section(course_id, sec_id, semester, year, building, room_number, time_slot_id)
teaches(ID, course_id, sec_id, semester, year)
takes(ID, course_id, sec_id, semester, year, grade)
```

### Query A: Students who took at least one Comp. Sci. course

**문제:** Find the ID and name of each student who has taken at least one Comp. Sci. course; make sure there are no duplicate names in the result.

**분석:**
- `takes`, `course`, `student` 테이블을 조인
- `course.dept_name = 'Comp. Sci.'` 조건
- `DISTINCT`로 중복 제거

**SQL 쿼리:**
```sql
SELECT DISTINCT s.ID, s.name
FROM student s
NATURAL JOIN takes t
NATURAL JOIN course c
WHERE c.dept_name = 'Comp. Sci.';
```

**대안 (JOIN 명시):**
```sql
SELECT DISTINCT s.ID, s.name
FROM student s
JOIN takes t ON s.ID = t.ID
JOIN course c ON t.course_id = c.course_id
WHERE c.dept_name = 'Comp. Sci.';
```

**설명:**
1. **NATURAL JOIN**: 같은 이름의 속성을 기준으로 자동 조인
   - `student`와 `takes`: `ID`로 조인
   - `takes`와 `course`: `course_id`로 조인
2. **DISTINCT**: 한 학생이 여러 CS 과목을 수강해도 한 번만 출력
3. **WHERE**: Comp. Sci. 학과 과목만 필터링

**예시 결과:**
```
┌───────┬─────────┐
│  ID   │  name   │
├───────┼─────────┤
│ 12345 │ Shankar │
│ 19991 │ Brandt  │
│ 23121 │ Chavez  │
│ 44553 │ Peltier │
└───────┴─────────┘
```

**중요 포인트:**
- `DISTINCT`를 빼면 한 학생이 여러 번 나타날 수 있음
- 인덱스: `takes(ID)`, `takes(course_id)`, `course(dept_name)`에 있으면 성능 향상

### Query B: Students who haven't taken courses before 2017

**문제:** Find the ID and name of each student who has not taken any course offered before 2017.

**분석:**
- 2017년 이전에 수강한 학생을 찾고
- 전체 학생에서 **제외**

**방법 1: NOT IN 사용**
```sql
SELECT ID, name
FROM student
WHERE ID NOT IN (
    SELECT ID
    FROM takes
    WHERE year < 2017
);
```

**방법 2: NOT EXISTS 사용**
```sql
SELECT s.ID, s.name
FROM student s
WHERE NOT EXISTS (
    SELECT *
    FROM takes t
    WHERE t.ID = s.ID
      AND t.year < 2017
);
```

**방법 3: LEFT JOIN 사용**
```sql
SELECT s.ID, s.name
FROM student s
LEFT JOIN takes t ON s.ID = t.ID AND t.year < 2017
WHERE t.ID IS NULL;
```

**설명:**
- **NOT IN**: 2017년 이전 수강 학생 목록에 없는 학생
- **NOT EXISTS**: 상관 부질의로 각 학생마다 2017년 이전 수강 기록 확인
- **LEFT JOIN**: 2017년 이전 수강 기록과 조인 후 매칭되지 않는 학생 선택

**성능 비교:**
- `NOT IN`: NULL 값 처리 주의 필요, 일반적으로 느림
- `NOT EXISTS`: NULL 안전, 상관 부질의로 효율적
- `LEFT JOIN`: 대량 데이터에서 효율적일 수 있음

**예시 결과:**
```
┌───────┬─────────┐
│  ID   │  name   │
├───────┼─────────┤
│ 00128 │ Zhang   │
│ 45678 │ Levy    │
│ 54321 │ Williams│
└───────┴─────────┘
```

**주의사항:**
```sql
-- 잘못된 쿼리: 2017년 이후 수강한 기록도 있는 학생 제외
SELECT s.ID, s.name
FROM student s
NATURAL JOIN takes t
WHERE t.year >= 2017;
-- 이 쿼리는 2016년과 2018년 모두 수강한 학생을 포함함
```

### Query C: Maximum salary per department

**문제:** For each department, find the maximum salary of instructors in that department. You may assume that every department has at least one instructor.

**SQL 쿼리:**
```sql
SELECT dept_name, MAX(salary) AS max_salary
FROM instructor
GROUP BY dept_name;
```

**설명:**
1. **GROUP BY dept_name**: 학과별로 그룹화
2. **MAX(salary)**: 각 그룹의 최대 급여 계산
3. **AS max_salary**: 결과 컬럼에 별칭 부여

**예시 결과:**
```
┌───────────┬────────────┐
│ dept_name │ max_salary │
├───────────┼────────────┤
│ Biology   │ 72000      │
│ Comp.Sci. │ 92000      │
│ Elec.Eng. │ 80000      │
│ Finance   │ 90000      │
│ History   │ 62000      │
│ Music     │ 40000      │
│ Physics   │ 95000      │
└───────────┴────────────┘
```

**집계 함수:**
- `MAX()`: 최댓값
- `MIN()`: 최솟값
- `AVG()`: 평균
- `SUM()`: 합계
- `COUNT()`: 개수

**주의사항:**
```sql
-- 오류: SELECT 절에 집계되지 않은 컬럼
SELECT dept_name, name, MAX(salary)  -- 오류!
FROM instructor
GROUP BY dept_name;
-- name은 GROUP BY에 없고 집계 함수로 감싸지지 않음
```

### Query D: Minimum of per-department maximum salaries

**문제:** Find the lowest, across all departments, of the per-department maximum salary computed by the preceding query.

**방법 1: 중첩 부질의**
```sql
SELECT MIN(max_salary) AS min_of_max
FROM (
    SELECT dept_name, MAX(salary) AS max_salary
    FROM instructor
    GROUP BY dept_name
) AS dept_max;
```

**방법 2: WITH 절 (CTE)**
```sql
WITH dept_max AS (
    SELECT dept_name, MAX(salary) AS max_salary
    FROM instructor
    GROUP BY dept_name
)
SELECT MIN(max_salary) AS min_of_max
FROM dept_max;
```

**설명:**
1. **내부 쿼리**: 각 학과별 최대 급여 계산
2. **외부 쿼리**: 그 중 최솟값 선택

**예시 결과:**
```
┌────────────┐
│ min_of_max │
├────────────┤
│ 40000      │  ← Music 학과의 최대 급여
└────────────┘
```

**WITH 절의 장점:**
- 쿼리 가독성 향상
- 여러 번 참조 가능
- 복잡한 쿼리 단계별 구성

**추가 분석: 어느 학과인지도 찾기**
```sql
WITH dept_max AS (
    SELECT dept_name, MAX(salary) AS max_salary
    FROM instructor
    GROUP BY dept_name
)
SELECT dept_name, max_salary
FROM dept_max
WHERE max_salary = (SELECT MIN(max_salary) FROM dept_max);
```

결과:
```
┌───────────┬────────────┐
│ dept_name │ max_salary │
├───────────┼────────────┤
│ Music     │ 40000      │
└───────────┴────────────┘
```

---

## Exercise 3.12: University Schema - Data Manipulation

### Query A: Create a new course

**문제:** Create a new course "CS-001", titled "Weekly Seminar", with 0 credits.

**SQL:**
```sql
INSERT INTO course (course_id, title, dept_name, credits)
VALUES ('CS-001', 'Weekly Seminar', 'Comp. Sci.', 0);
```

**설명:**
- `course_id`: 'CS-001' (기본키)
- `title`: 'Weekly Seminar'
- `dept_name`: 'Comp. Sci.' (외래키 - department 테이블 참조)
- `credits`: 0

**주의사항:**
```sql
-- dept_name이 department 테이블에 존재해야 함
SELECT dept_name FROM department WHERE dept_name = 'Comp. Sci.';
-- 존재하지 않으면 외래키 제약조건 위반 오류 발생
```

### Query B: Create a section

**문제:** Create a section of this course in Fall 2017, with sec_id of 1, and with the location of this section not yet specified.

**SQL:**
```sql
INSERT INTO section (course_id, sec_id, semester, year, building, room_number, time_slot_id)
VALUES ('CS-001', '1', 'Fall', 2017, NULL, NULL, NULL);
```

**설명:**
- `course_id`: 'CS-001' (외래키)
- `sec_id`: '1'
- `semester`: 'Fall'
- `year`: 2017
- `building`, `room_number`, `time_slot_id`: NULL (아직 미정)

**NULL 처리:**
- SQL에서 `NULL`은 "값이 없음" 또는 "알 수 없음"을 의미
- 위치가 정해지면 나중에 UPDATE 가능

**대안 (기본값 지정):**
```sql
-- 스키마 정의 시 기본값 설정
CREATE TABLE section (
    course_id VARCHAR(8),
    sec_id VARCHAR(8),
    semester VARCHAR(6),
    year NUMERIC(4,0),
    building VARCHAR(15) DEFAULT 'TBA',  -- To Be Announced
    room_number VARCHAR(7) DEFAULT 'TBA',
    time_slot_id VARCHAR(4) DEFAULT NULL,
    PRIMARY KEY (course_id, sec_id, semester, year),
    FOREIGN KEY (course_id) REFERENCES course(course_id)
);
```

### Query C: Enroll all Comp. Sci. students

**문제:** Enroll every student in the Comp. Sci. department in the above section.

**SQL:**
```sql
INSERT INTO takes (ID, course_id, sec_id, semester, year, grade)
SELECT ID, 'CS-001', '1', 'Fall', 2017, NULL
FROM student
WHERE dept_name = 'Comp. Sci.';
```

**설명:**
- `INSERT INTO ... SELECT`: 쿼리 결과를 직접 삽입
- `student` 테이블에서 Comp. Sci. 학생들의 ID 선택
- 각 학생에 대해 수강 신청 레코드 생성
- `grade`: NULL (아직 성적 미부여)

**예시:**
```sql
-- 삽입 전 확인
SELECT ID, name FROM student WHERE dept_name = 'Comp. Sci.';
```

결과:
```
┌───────┬─────────┐
│  ID   │  name   │
├───────┼─────────┤
│ 12345 │ Shankar │
│ 23121 │ Chavez  │
│ 45678 │ Levy    │
└───────┴─────────┘
```

삽입 후 `takes` 테이블:
```
┌───────┬───────────┬────────┬──────────┬──────┬───────┐
│  ID   │ course_id │ sec_id │ semester │ year │ grade │
├───────┼───────────┼────────┼──────────┼──────┼───────┤
│ 12345 │ CS-001    │ 1      │ Fall     │ 2017 │ NULL  │
│ 23121 │ CS-001    │ 1      │ Fall     │ 2017 │ NULL  │
│ 45678 │ CS-001    │ 1      │ Fall     │ 2017 │ NULL  │
└───────┴───────────┴────────┴──────────┴──────┴───────┘
```

**주의사항:**
```sql
-- 중복 삽입 방지: 이미 수강 신청한 학생은 제외
INSERT INTO takes (ID, course_id, sec_id, semester, year, grade)
SELECT s.ID, 'CS-001', '1', 'Fall', 2017, NULL
FROM student s
WHERE s.dept_name = 'Comp. Sci.'
  AND NOT EXISTS (
      SELECT *
      FROM takes t
      WHERE t.ID = s.ID
        AND t.course_id = 'CS-001'
        AND t.sec_id = '1'
        AND t.semester = 'Fall'
        AND t.year = 2017
  );
```

### Query D: Delete specific enrollment

**문제:** Delete enrollments in the above section where the student's ID is 12345.

**SQL:**
```sql
DELETE FROM takes
WHERE ID = '12345'
  AND course_id = 'CS-001'
  AND sec_id = '1'
  AND semester = 'Fall'
  AND year = 2017;
```

**설명:**
- 특정 학생(ID=12345)의 특정 섹션 수강 신청만 삭제
- 모든 기본키 속성을 WHERE 절에 포함하여 정확한 튜플 식별

**주의사항:**
```sql
-- 잘못된 쿼리: 너무 광범위한 삭제
DELETE FROM takes
WHERE ID = '12345';
-- 학생 12345의 모든 수강 신청이 삭제됨 (의도하지 않을 수 있음)
```

**삭제 전 확인:**
```sql
-- 삭제될 레코드 미리 확인
SELECT *
FROM takes
WHERE ID = '12345'
  AND course_id = 'CS-001'
  AND sec_id = '1'
  AND semester = 'Fall'
  AND year = 2017;
```

### Query E: Delete course CS-001

**문제:** Delete the course CS-001. What will happen if you run this delete statement without first deleting offerings (sections) of this course?

**SQL:**
```sql
DELETE FROM course
WHERE course_id = 'CS-001';
```

**결과 예측:**

**시나리오 1: 외래키 제약조건이 있는 경우**

```sql
-- section 테이블의 외래키 정의
FOREIGN KEY (course_id) REFERENCES course(course_id)
```

**에러 발생:**
```
ERROR: update or delete on table "course" violates foreign key constraint
DETAIL: Key (course_id)=(CS-001) is still referenced from table "section".
```

**이유:**
- `section` 테이블의 레코드가 `course_id = 'CS-001'`을 참조 중
- **참조 무결성** 위반
- 데이터베이스가 삭제를 거부

**시나리오 2: CASCADE 옵션이 설정된 경우**

```sql
-- section 테이블의 외래키 정의 (CASCADE)
FOREIGN KEY (course_id) REFERENCES course(course_id)
    ON DELETE CASCADE
```

**결과:**
- `course`에서 'CS-001' 삭제
- `section`에서 `course_id = 'CS-001'`인 모든 레코드 **자동 삭제**
- `takes`에서도 연쇄 삭제 (section을 참조하는 경우)

**시나리오 3: SET NULL 옵션**

```sql
FOREIGN KEY (course_id) REFERENCES course(course_id)
    ON DELETE SET NULL
```

**결과:**
- `section`의 `course_id`가 NULL로 설정됨 (논리적으로 맞지 않음)

**올바른 삭제 순서:**

```sql
-- 1. takes에서 해당 섹션의 수강 신청 삭제
DELETE FROM takes
WHERE course_id = 'CS-001';

-- 2. section에서 해당 과목의 섹션 삭제
DELETE FROM section
WHERE course_id = 'CS-001';

-- 3. course에서 과목 삭제
DELETE FROM course
WHERE course_id = 'CS-001';
```

**외래키 제약조건 옵션:**

| 옵션 | 동작 |
|------|------|
| `ON DELETE RESTRICT` | 참조되는 경우 삭제 거부 (기본값) |
| `ON DELETE CASCADE` | 참조하는 레코드도 함께 삭제 |
| `ON DELETE SET NULL` | 참조하는 컬럼을 NULL로 설정 |
| `ON DELETE SET DEFAULT` | 참조하는 컬럼을 기본값으로 설정 |

**권장 사항:**
- 중요한 데이터는 `RESTRICT` 사용 (실수 방지)
- 종속 데이터는 `CASCADE` 사용 (자동 정리)

### Query F: Delete enrollments for "advanced" courses

**문제:** Delete all takes tuples corresponding to any section of any course with the word "advanced" as a part of the title; ignore case when matching the word with the title.

**SQL:**
```sql
DELETE FROM takes
WHERE course_id IN (
    SELECT course_id
    FROM course
    WHERE LOWER(title) LIKE '%advanced%'
);
```

**대안 (조인 사용):**
```sql
DELETE FROM takes
WHERE EXISTS (
    SELECT *
    FROM course c
    WHERE c.course_id = takes.course_id
      AND LOWER(c.title) LIKE '%advanced%'
);
```

**설명:**
1. **LOWER(title)**: 제목을 소문자로 변환 (대소문자 무시)
2. **LIKE '%advanced%'**: 'advanced'가 포함된 모든 제목
   - `%`: 0개 이상의 임의 문자
3. **IN**: 부질의 결과에 포함되는 course_id
4. **EXISTS**: 상관 부질의로 효율적인 검사

**예시:**

```sql
-- 삭제될 과목 확인
SELECT course_id, title
FROM course
WHERE LOWER(title) LIKE '%advanced%';
```

결과:
```
┌───────────┬───────────────────────────┐
│ course_id │          title            │
├───────────┼───────────────────────────┤
│ CS-315    │ Robotics                  │
│ CS-319    │ Image Processing          │
│ CS-347    │ Database System Concepts  │
│ FIN-201   │ Investment Banking        │
│ PHY-101   │ Physical Principles       │
└───────────┴───────────────────────────┘

-- 'advanced'가 포함된 과목이 없다면 아무것도 삭제되지 않음
```

**대소문자 구분 함수:**
- **PostgreSQL**: `LOWER()`, `UPPER()`, `ILIKE`
- **MySQL**: `LOWER()`, `UPPER()`, 대소문자 무시가 기본
- **SQL Server**: `LOWER()`, `UPPER()`, `COLLATE`
- **Oracle**: `LOWER()`, `UPPER()`, `REGEXP_LIKE`

**PostgreSQL에서 더 간단한 방법:**
```sql
DELETE FROM takes
WHERE course_id IN (
    SELECT course_id
    FROM course
    WHERE title ILIKE '%advanced%'  -- ILIKE: 대소문자 무시
);
```

**성능 고려사항:**
- `LIKE`는 인덱스를 효과적으로 사용하지 못할 수 있음
- 전문 검색(Full-Text Search)이 필요하면 별도 기능 사용
- 대량 데이터에서는 `EXISTS`가 `IN`보다 효율적일 수 있음

---

## Exercise 3.13: Insurance Database DDL

### 문제
> Write SQL DDL corresponding to the schema in Figure 3.17. Make any reasonable assumptions about data types, and be sure to declare primary and foreign keys.

**Figure 3.17 스키마:**
```
person(driver_id, name, address)
car(license_plate, model, year)
accident(report_number, year, location)
owns(driver_id, license_plate)
participated(report_number, license_plate, driver_id, damage_amount)
```

### SQL DDL

```sql
-- person 테이블
CREATE TABLE person (
    driver_id VARCHAR(20) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address VARCHAR(200)
);

-- car 테이블
CREATE TABLE car (
    license_plate VARCHAR(20) PRIMARY KEY,
    model VARCHAR(50) NOT NULL,
    year NUMERIC(4, 0) CHECK (year >= 1900 AND year <= 2100)
);

-- accident 테이블
CREATE TABLE accident (
    report_number VARCHAR(20) PRIMARY KEY,
    year NUMERIC(4, 0) CHECK (year >= 1900 AND year <= 2100),
    location VARCHAR(200)
);

-- owns 테이블 (person과 car의 M:N 관계)
CREATE TABLE owns (
    driver_id VARCHAR(20),
    license_plate VARCHAR(20),
    PRIMARY KEY (driver_id, license_plate),
    FOREIGN KEY (driver_id) REFERENCES person(driver_id)
        ON DELETE CASCADE,
    FOREIGN KEY (license_plate) REFERENCES car(license_plate)
        ON DELETE CASCADE
);

-- participated 테이블 (accident와 car, person 연결)
CREATE TABLE participated (
    report_number VARCHAR(20),
    license_plate VARCHAR(20),
    driver_id VARCHAR(20),
    damage_amount NUMERIC(10, 2) CHECK (damage_amount >= 0),
    PRIMARY KEY (report_number, license_plate, driver_id),
    FOREIGN KEY (report_number) REFERENCES accident(report_number)
        ON DELETE CASCADE,
    FOREIGN KEY (license_plate) REFERENCES car(license_plate)
        ON DELETE CASCADE,
    FOREIGN KEY (driver_id) REFERENCES person(driver_id)
        ON DELETE CASCADE
);
```

### 설계 결정 사항

**1. 데이터 타입 선택:**

| 속성 | 타입 | 이유 |
|------|------|------|
| driver_id | VARCHAR(20) | 문자+숫자 조합 가능, 국가마다 형식 다름 |
| name | VARCHAR(100) | 이름 길이 충분히 수용 |
| address | VARCHAR(200) | 주소는 가변 길이, 여유 있게 |
| license_plate | VARCHAR(20) | 번호판 형식 다양 |
| model | VARCHAR(50) | 차량 모델명 |
| year | NUMERIC(4,0) | 정수형 연도 (1900-2100) |
| report_number | VARCHAR(20) | 사고 보고서 번호 |
| location | VARCHAR(200) | 사고 장소 |
| damage_amount | NUMERIC(10,2) | 소수점 2자리 (센트까지) |

**2. 제약조건 (Constraints):**

**PRIMARY KEY:**
- `person`: `driver_id`
- `car`: `license_plate`
- `accident`: `report_number`
- `owns`: `(driver_id, license_plate)` - 복합 키
- `participated`: `(report_number, license_plate, driver_id)` - 복합 키

**FOREIGN KEY:**
- `owns`: `driver_id`, `license_plate` 모두 외래키
- `participated`: 3개의 외래키 (accident, car, person 참조)

**CHECK 제약조건:**
- `year`: 1900~2100 범위 (현실적인 범위)
- `damage_amount`: 0 이상 (음수 불가)

**NOT NULL:**
- `person.name`: 이름은 필수
- `car.model`: 모델명 필수

**3. ON DELETE CASCADE:**
- 소유자(person) 삭제 시 소유 관계(owns)도 삭제
- 차량(car) 삭제 시 관련 레코드 모두 삭제
- 사고(accident) 삭제 시 참여 기록(participated) 삭제

**대안 설계:**
```sql
-- ON DELETE RESTRICT: 참조되는 경우 삭제 거부
CREATE TABLE owns (
    driver_id VARCHAR(20),
    license_plate VARCHAR(20),
    PRIMARY KEY (driver_id, license_plate),
    FOREIGN KEY (driver_id) REFERENCES person(driver_id)
        ON DELETE RESTRICT,
    FOREIGN KEY (license_plate) REFERENCES car(license_plate)
        ON DELETE RESTRICT
);
-- 이 경우 owns에 레코드가 있으면 person이나 car 삭제 불가
```

**4. 추가 제약조건 (선택 사항):**

```sql
-- 인덱스 추가 (검색 성능 향상)
CREATE INDEX idx_person_name ON person(name);
CREATE INDEX idx_car_model ON car(model);
CREATE INDEX idx_accident_year ON accident(year);
CREATE INDEX idx_participated_driver ON participated(driver_id);

-- UNIQUE 제약조건
ALTER TABLE person ADD CONSTRAINT unique_driver_name
    UNIQUE (name, address);
-- 같은 이름의 같은 주소 사람은 중복 불가

-- DEFAULT 값
ALTER TABLE participated
    ALTER COLUMN damage_amount SET DEFAULT 0.00;
```

---

## Exercise 3.14: Insurance Database Queries

### Query A: Accidents involving John Smith's car

**문제:** Find the number of accidents involving a car belonging to a person named "John Smith".

**SQL:**
```sql
SELECT COUNT(DISTINCT p.report_number) AS accident_count
FROM person per
JOIN owns o ON per.driver_id = o.driver_id
JOIN participated p ON o.license_plate = p.license_plate
WHERE per.name = 'John Smith';
```

**설명:**
1. `person` → `owns`: John Smith가 소유한 차량 찾기
2. `owns` → `participated`: 그 차량이 연루된 사고 찾기
3. `COUNT(DISTINCT report_number)`: 사고 수 세기 (중복 제거)

**왜 DISTINCT가 필요한가?**
- 한 사고에 John Smith의 여러 차량이 연루될 수 있음
- 한 차량에 여러 driver가 탑승했을 수 있음

**예시 데이터:**
```
person:
┌───────────┬────────────┬──────────────┐
│ driver_id │    name    │   address    │
├───────────┼────────────┼──────────────┤
│ D001      │ John Smith │ 123 Main St  │
└───────────┴────────────┴──────────────┘

owns:
┌───────────┬───────────────┐
│ driver_id │ license_plate │
├───────────┼───────────────┤
│ D001      │ ABC123        │
│ D001      │ XYZ789        │
└───────────┴───────────────┘

participated:
┌───────────────┬───────────────┬───────────┬───────────────┐
│ report_number │ license_plate │ driver_id │ damage_amount │
├───────────────┼───────────────┼───────────┼───────────────┤
│ AR001         │ ABC123        │ D001      │ 1500.00       │
│ AR002         │ XYZ789        │ D001      │ 2000.00       │
│ AR003         │ ABC123        │ D001      │ 500.00        │
└───────────────┴───────────────┴───────────┴───────────────┘

결과: accident_count = 3
```

**대안 (부질의 사용):**
```sql
SELECT COUNT(DISTINCT report_number) AS accident_count
FROM participated
WHERE license_plate IN (
    SELECT o.license_plate
    FROM owns o
    JOIN person per ON o.driver_id = per.driver_id
    WHERE per.name = 'John Smith'
);
```

### Query B: Update damage amount

**문제:** Update the damage amount for the car with license_plate "AABB2000" in the accident with report number "AR2197" to $3000.

**SQL:**
```sql
UPDATE participated
SET damage_amount = 3000.00
WHERE license_plate = 'AABB2000'
  AND report_number = 'AR2197';
```

**설명:**
- `SET damage_amount = 3000.00`: 새 값으로 변경
- `WHERE`: 특정 사고의 특정 차량만 업데이트

**업데이트 전 확인:**
```sql
-- 현재 값 확인
SELECT *
FROM participated
WHERE license_plate = 'AABB2000'
  AND report_number = 'AR2197';
```

**업데이트 후 검증:**
```sql
-- 변경 확인
SELECT *
FROM participated
WHERE license_plate = 'AABB2000'
  AND report_number = 'AR2197';
```

**주의사항:**
```sql
-- 잘못된 쿼리: WHERE 절 누락
UPDATE participated
SET damage_amount = 3000.00;
-- 모든 레코드의 damage_amount가 3000으로 변경됨!
```

**트랜잭션 사용 권장:**
```sql
BEGIN TRANSACTION;

UPDATE participated
SET damage_amount = 3000.00
WHERE license_plate = 'AABB2000'
  AND report_number = 'AR2197';

-- 확인
SELECT * FROM participated
WHERE license_plate = 'AABB2000' AND report_number = 'AR2197';

-- 문제 없으면 커밋, 문제 있으면 롤백
COMMIT;
-- 또는 ROLLBACK;
```

---

## Exercise 3.15: Bank Database Queries

### Bank 스키마
```
branch(branch_name, branch_city, assets)
customer(customer_name, customer_street, customer_city)
account(account_number, branch_name, balance)
loan(loan_number, branch_name, amount)
depositor(customer_name, account_number)
borrower(customer_name, loan_number)
```

### Query A: Customers with accounts at all Brooklyn branches

**문제:** Find each customer who has an account at every branch located in "Brooklyn".

**분석:**
- "모든(every)" → **관계 대수의 나눗셈** (Division)
- SQL에서는 **이중 부정**으로 표현

**논리:**
- Brooklyn에 있는 모든 지점을 찾음
- 각 고객에 대해: Brooklyn 지점 중 계좌가 없는 지점이 없는지 확인

**SQL (이중 부정):**
```sql
SELECT DISTINCT d.customer_name
FROM depositor d
WHERE NOT EXISTS (
    -- Brooklyn 지점 중
    SELECT b.branch_name
    FROM branch b
    WHERE b.branch_city = 'Brooklyn'
      AND NOT EXISTS (
          -- 이 고객이 계좌를 가지지 않은 지점
          SELECT *
          FROM depositor d2
          JOIN account a ON d2.account_number = a.account_number
          WHERE d2.customer_name = d.customer_name
            AND a.branch_name = b.branch_name
      )
);
```

**해석:**
- "Brooklyn에 있는 모든 지점에서, 이 고객이 계좌를 가지지 않은 지점이 존재하지 않는다"
- = "모든 Brooklyn 지점에 계좌가 있다"

**예시 데이터:**
```
branch:
┌─────────────┬─────────────┬────────┐
│ branch_name │ branch_city │ assets │
├─────────────┼─────────────┼────────┤
│ Downtown    │ Brooklyn    │ 900000 │
│ Redwood     │ Brooklyn    │ 210000 │
│ Mianus      │ Horseneck   │ 400000 │
└─────────────┴─────────────┴────────┘

customer:
┌───────────────┬────────────────┬──────────────┐
│ customer_name │ customer_street│ customer_city│
├───────────────┼────────────────┼──────────────┤
│ Johnson       │ Alma           │ Brooklyn     │
│ Smith         │ North          │ Rye          │
└───────────────┴────────────────┴──────────────┘

account:
┌────────────────┬─────────────┬─────────┐
│ account_number │ branch_name │ balance │
├────────────────┼─────────────┼─────────┤
│ A-101          │ Downtown    │ 500     │
│ A-201          │ Redwood     │ 900     │
│ A-215          │ Mianus      │ 700     │
└────────────────┴─────────────┴─────────┘

depositor:
┌───────────────┬────────────────┐
│ customer_name │ account_number │
├───────────────┼────────────────┤
│ Johnson       │ A-101          │
│ Johnson       │ A-201          │  ← Brooklyn 2개 지점 모두
│ Smith         │ A-215          │
└───────────────┴────────────────┘

결과: Johnson (모든 Brooklyn 지점에 계좌 있음)
```

**대안 (GROUP BY HAVING):**
```sql
SELECT d.customer_name
FROM depositor d
JOIN account a ON d.account_number = a.account_number
JOIN branch b ON a.branch_name = b.branch_name
WHERE b.branch_city = 'Brooklyn'
GROUP BY d.customer_name
HAVING COUNT(DISTINCT a.branch_name) = (
    SELECT COUNT(*)
    FROM branch
    WHERE branch_city = 'Brooklyn'
);
```

**해석:**
- Brooklyn 지점에서의 계좌 수가
- Brooklyn에 있는 총 지점 수와 같은 고객

### Query B: Total sum of all loan amounts

**문제:** Find the total sum of all loan amounts in the bank.

**SQL:**
```sql
SELECT SUM(amount) AS total_loan_amount
FROM loan;
```

**간단한 집계 함수:**
- `SUM(amount)`: 모든 대출 금액의 합
- `GROUP BY` 없음: 전체 테이블이 하나의 그룹

**예시:**
```
loan:
┌─────────────┬─────────────┬────────┐
│ loan_number │ branch_name │ amount │
├─────────────┼─────────────┼────────┤
│ L-11        │ Round Hill  │ 900    │
│ L-14        │ Downtown    │ 1500   │
│ L-15        │ Perryridge  │ 1500   │
│ L-16        │ Perryridge  │ 1300   │
│ L-17        │ Downtown    │ 1000   │
│ L-23        │ Redwood     │ 2000   │
│ L-93        │ Mianus      │ 500    │
└─────────────┴─────────────┴────────┘

결과:
┌───────────────────┐
│ total_loan_amount │
├───────────────────┤
│ 8700              │
└───────────────────┘
```

**NULL 처리:**
```sql
-- amount가 NULL인 경우 0으로 처리
SELECT SUM(COALESCE(amount, 0)) AS total_loan_amount
FROM loan;

-- 또는 IFNULL (MySQL)
SELECT SUM(IFNULL(amount, 0)) AS total_loan_amount
FROM loan;
```

### Query C: Branches with assets > Brooklyn branches

**문제:** Find the names of all branches that have assets greater than those of at least one branch located in "Brooklyn".

**해석:**
- Brooklyn 지점 중 **최소 하나**보다 자산이 많은 지점
- = Brooklyn 지점의 **최솟값**보다 큰 지점

**SQL:**
```sql
SELECT branch_name
FROM branch
WHERE assets > (
    SELECT MIN(assets)
    FROM branch
    WHERE branch_city = 'Brooklyn'
);
```

**설명:**
1. **내부 쿼리**: Brooklyn 지점의 최소 자산 계산
2. **외부 쿼리**: 그보다 큰 자산을 가진 지점 선택

**예시:**
```
branch:
┌─────────────┬─────────────┬────────┐
│ branch_name │ branch_city │ assets │
├─────────────┼─────────────┼────────┤
│ Brighton    │ Brooklyn    │ 710000 │
│ Downtown    │ Brooklyn    │ 900000 │
│ Mianus      │ Horseneck   │ 400000 │
│ Redwood     │ Palo Alto   │ 210000 │
│ Perryridge  │ Horseneck   │ 170000 │
└─────────────┴─────────────┴────────┘

Brooklyn 지점 최소 자산: MIN(710000, 900000) = 710000

결과:
┌─────────────┐
│ branch_name │
├─────────────┤
│ Downtown    │  (900000 > 710000)
└─────────────┘
```

**대안 (ANY 사용):**
```sql
SELECT branch_name
FROM branch
WHERE assets > ANY (
    SELECT assets
    FROM branch
    WHERE branch_city = 'Brooklyn'
);
```

**비교: SOME, ANY, ALL**

```sql
-- > ANY: 최소 하나보다 큰
WHERE assets > ANY (SELECT assets FROM branch WHERE branch_city = 'Brooklyn')
-- ≡ WHERE assets > MIN(SELECT assets FROM branch WHERE branch_city = 'Brooklyn')

-- > ALL: 모두보다 큰
WHERE assets > ALL (SELECT assets FROM branch WHERE branch_city = 'Brooklyn')
-- ≡ WHERE assets > MAX(SELECT assets FROM branch WHERE branch_city = 'Brooklyn')

-- = ANY: 하나와 같은 (= IN)
WHERE assets = ANY (SELECT assets FROM branch WHERE branch_city = 'Brooklyn')
-- ≡ WHERE assets IN (SELECT assets FROM branch WHERE branch_city = 'Brooklyn')
```

---

## Exercise 3.16: Employee Database Queries

### Employee 스키마
```
employee(person_name, street, city)
works(person_name, company_name, salary)
company(company_name, city)
```

### Query A: Employees living in company city

**문제:** Find ID and name of each employee who lives in the same city as the location of the company for which the employee works.

**SQL:**
```sql
SELECT w.person_name, e.city
FROM employee e
JOIN works w ON e.person_name = w.person_name
JOIN company c ON w.company_name = c.company_name
WHERE e.city = c.city;
```

**설명:**
- `employee`와 `works` 조인: 직원의 근무지 연결
- `works`와 `company` 조인: 회사의 위치 연결
- `WHERE e.city = c.city`: 거주 도시 = 회사 위치

**예시:**
```
employee:
┌─────────────┬────────────┬──────────┐
│ person_name │   street   │   city   │
├─────────────┼────────────┼──────────┤
│ Smith       │ Main St    │ New York │
│ Jones       │ Oak Ave    │ Boston   │
│ Williams    │ Pine Rd    │ Chicago  │
└─────────────┴────────────┴──────────┘

works:
┌─────────────┬──────────────────┬────────┐
│ person_name │  company_name    │ salary │
├─────────────┼──────────────────┼────────┤
│ Smith       │ First Bank Corp  │ 50000  │
│ Jones       │ First Bank Corp  │ 45000  │
│ Williams    │ Small Bank Corp  │ 35000  │
└─────────────┴──────────────────┴────────┘

company:
┌──────────────────┬──────────┐
│  company_name    │   city   │
├──────────────────┼──────────┤
│ First Bank Corp  │ New York │
│ Small Bank Corp  │ Chicago  │
└──────────────────┴──────────┘

결과:
┌─────────────┬──────────┐
│ person_name │   city   │
├─────────────┼──────────┤
│ Smith       │ New York │  (Smith lives in NY, works at First Bank in NY)
│ Williams    │ Chicago  │  (Williams lives in Chicago, works at Small Bank in Chicago)
└─────────────┴──────────┘
```

### Query B: Employees living on same street as manager

**문제:** Find ID and name of each employee who lives in the same city and on the same street as does her or his manager.

**가정:** `manages(manager_name, employee_name)` 테이블이 존재한다고 가정

**SQL:**
```sql
SELECT e1.person_name
FROM employee e1
JOIN manages m ON e1.person_name = m.employee_name
JOIN employee e2 ON m.manager_name = e2.person_name
WHERE e1.city = e2.city
  AND e1.street = e2.street;
```

**설명:**
- `e1`: 직원 정보
- `m`: 관리 관계 (누가 누구의 매니저인지)
- `e2`: 매니저 정보
- `WHERE`: 같은 도시, 같은 거리

**예시:**
```
employee:
┌─────────────┬────────────┬──────────┐
│ person_name │   street   │   city   │
├─────────────┼────────────┼──────────┤
│ Smith       │ Main St    │ New York │
│ Jones       │ Main St    │ New York │
│ Brown       │ Oak Ave    │ Boston   │
│ Davis       │ Oak Ave    │ Boston   │
└─────────────┴────────────┴──────────┘

manages:
┌──────────────┬───────────────┐
│ manager_name │ employee_name │
├──────────────┼───────────────┤
│ Smith        │ Jones         │  ← 같은 거리
│ Brown        │ Davis         │  ← 같은 거리
└──────────────┴───────────────┘

결과:
┌─────────────┐
│ person_name │
├─────────────┤
│ Jones       │
│ Davis       │
└─────────────┘
```

### Query C: Employees earning more than company average

**문제:** Find ID and name of each employee who earns more than the average salary of all employees of her or his company.

**SQL:**
```sql
SELECT w1.person_name, w1.salary
FROM works w1
WHERE w1.salary > (
    SELECT AVG(w2.salary)
    FROM works w2
    WHERE w2.company_name = w1.company_name
);
```

**설명:**
- **상관 부질의**: 각 직원의 회사별 평균 급여 계산
- 자신의 급여가 그 평균보다 큰지 비교

**예시:**
```
works:
┌─────────────┬──────────────────┬────────┐
│ person_name │  company_name    │ salary │
├─────────────┼──────────────────┼────────┤
│ Smith       │ First Bank Corp  │ 80000  │
│ Jones       │ First Bank Corp  │ 50000  │
│ Brown       │ First Bank Corp  │ 40000  │
│ Williams    │ Small Bank Corp  │ 60000  │
│ Davis       │ Small Bank Corp  │ 30000  │
└─────────────┴──────────────────┴────────┘

First Bank Corp 평균: (80000 + 50000 + 40000) / 3 = 56667
Small Bank Corp 평균: (60000 + 30000) / 2 = 45000

결과:
┌─────────────┬────────┐
│ person_name │ salary │
├─────────────┼────────┤
│ Smith       │ 80000  │  (80000 > 56667)
│ Williams    │ 60000  │  (60000 > 45000)
└─────────────┴────────┘
```

### Query D: Company with smallest payroll

**문제:** Find the company that has the smallest payroll.

**SQL:**
```sql
SELECT company_name
FROM works
GROUP BY company_name
HAVING SUM(salary) = (
    SELECT MIN(total_payroll)
    FROM (
        SELECT company_name, SUM(salary) AS total_payroll
        FROM works
        GROUP BY company_name
    ) AS company_payroll
);
```

**WITH 절 사용 (더 읽기 쉬움):**
```sql
WITH company_payroll AS (
    SELECT company_name, SUM(salary) AS total_payroll
    FROM works
    GROUP BY company_name
)
SELECT company_name
FROM company_payroll
WHERE total_payroll = (SELECT MIN(total_payroll) FROM company_payroll);
```

**설명:**
1. **내부 쿼리**: 각 회사별 총 급여(payroll) 계산
2. **MIN**: 가장 작은 payroll 찾기
3. **외부 쿼리**: 그 값과 같은 회사 선택

**예시:**
```
works:
┌─────────────┬──────────────────┬────────┐
│ person_name │  company_name    │ salary │
├─────────────┼──────────────────┼────────┤
│ Smith       │ First Bank Corp  │ 80000  │
│ Jones       │ First Bank Corp  │ 50000  │
│ Brown       │ First Bank Corp  │ 40000  │
│ Williams    │ Small Bank Corp  │ 30000  │
│ Davis       │ Small Bank Corp  │ 20000  │
└─────────────┴──────────────────┴────────┘

Payroll:
┌──────────────────┬───────────────┐
│  company_name    │ total_payroll │
├──────────────────┼───────────────┤
│ First Bank Corp  │ 170000        │
│ Small Bank Corp  │ 50000         │ ← 최소
└──────────────────┴───────────────┘

결과:
┌──────────────────┐
│  company_name    │
├──────────────────┤
│ Small Bank Corp  │
└──────────────────┘
```

---

## Exercise 3.17: Employee Database - Data Modification

### Query A: 10% raise for all employees

**문제:** Give all employees of "First Bank Corporation" a 10 percent raise.

**SQL:**
```sql
UPDATE works
SET salary = salary * 1.10
WHERE company_name = 'First Bank Corporation';
```

**설명:**
- `salary * 1.10`: 급여의 110% (10% 인상)
- `WHERE`: 특정 회사 직원만

**예시:**
```
업데이트 전:
┌─────────────┬─────────────────────────┬────────┐
│ person_name │     company_name        │ salary │
├─────────────┼─────────────────────────┼────────┤
│ Smith       │ First Bank Corporation  │ 50000  │
│ Jones       │ First Bank Corporation  │ 45000  │
└─────────────┴─────────────────────────┴────────┘

업데이트 후:
┌─────────────┬─────────────────────────┬────────┐
│ person_name │     company_name        │ salary │
├─────────────┼─────────────────────────┼────────┤
│ Smith       │ First Bank Corporation  │ 55000  │
│ Jones       │ First Bank Corporation  │ 49500  │
└─────────────┴─────────────────────────┴────────┘
```

### Query B: 10% raise for managers only

**문제:** Give all managers of "First Bank Corporation" a 10 percent raise.

**가정:** `manages` 테이블 존재

**SQL:**
```sql
UPDATE works
SET salary = salary * 1.10
WHERE company_name = 'First Bank Corporation'
  AND person_name IN (
      SELECT DISTINCT manager_name
      FROM manages
  );
```

**설명:**
- `manages` 테이블에서 매니저 목록 추출
- First Bank Corporation의 매니저만 인상

### Query C: Delete all Small Bank Corporation records

**문제:** Delete all tuples in the works relation for employees of "Small Bank Corporation".

**SQL:**
```sql
DELETE FROM works
WHERE company_name = 'Small Bank Corporation';
```

**주의사항:**
- 이 쿼리는 `works` 테이블에서만 삭제
- `employee` 테이블의 레코드는 유지됨
- 직원은 남아있지만 근무 기록만 삭제됨

**완전 삭제 (직원 정보까지):**
```sql
-- 1. works에서 삭제
DELETE FROM works
WHERE company_name = 'Small Bank Corporation';

-- 2. employee에서도 삭제 (다른 회사에 근무하지 않는 경우만)
DELETE FROM employee
WHERE person_name NOT IN (
    SELECT person_name FROM works
);
```

---

## Exercise 3.18: Employee Database DDL

### 문제
> Give an SQL schema definition for the employee database of Figure 3.19. Choose an appropriate domain for each attribute and an appropriate primary key for each relation schema. Include any foreign-key constraints that might be appropriate.

**Employee Database 스키마:**
```
employee(person_name, street, city)
works(person_name, company_name, salary)
company(company_name, city)
manages(manager_name, employee_name)
```

### SQL DDL

```sql
-- employee 테이블
CREATE TABLE employee (
    person_name VARCHAR(100) PRIMARY KEY,
    street VARCHAR(150),
    city VARCHAR(100)
);

-- company 테이블
CREATE TABLE company (
    company_name VARCHAR(100) PRIMARY KEY,
    city VARCHAR(100) NOT NULL
);

-- works 테이블 (employee와 company의 관계)
CREATE TABLE works (
    person_name VARCHAR(100),
    company_name VARCHAR(100),
    salary NUMERIC(10, 2) CHECK (salary >= 0),
    PRIMARY KEY (person_name, company_name),
    FOREIGN KEY (person_name) REFERENCES employee(person_name)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    FOREIGN KEY (company_name) REFERENCES company(company_name)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- manages 테이블 (employee 간의 관리 관계)
CREATE TABLE manages (
    manager_name VARCHAR(100),
    employee_name VARCHAR(100),
    PRIMARY KEY (manager_name, employee_name),
    FOREIGN KEY (manager_name) REFERENCES employee(person_name)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    FOREIGN KEY (employee_name) REFERENCES employee(person_name)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    CHECK (manager_name <> employee_name)  -- 자기 자신을 관리할 수 없음
);

-- 인덱스 추가 (성능 최적화)
CREATE INDEX idx_employee_city ON employee(city);
CREATE INDEX idx_works_company ON works(company_name);
CREATE INDEX idx_works_salary ON works(salary);
CREATE INDEX idx_manages_employee ON manages(employee_name);
```

### 설계 설명

**1. 데이터 타입:**
- `person_name`, `company_name`: `VARCHAR(100)`
- `street`: `VARCHAR(150)` (주소가 길 수 있음)
- `city`: `VARCHAR(100)`
- `salary`: `NUMERIC(10,2)` (최대 99,999,999.99)

**2. PRIMARY KEY:**
- `employee`: `person_name` (이름이 유일하다고 가정)
- `company`: `company_name`
- `works`: `(person_name, company_name)` - 한 사람이 여러 회사에서 일할 수 있음
- `manages`: `(manager_name, employee_name)` - 한 매니저가 여러 직원 관리

**3. FOREIGN KEY:**
- `works.person_name` → `employee.person_name`
- `works.company_name` → `company.company_name`
- `manages.manager_name` → `employee.person_name`
- `manages.employee_name` → `employee.person_name`

**4. CHECK 제약조건:**
- `salary >= 0`: 음수 급여 불가
- `manager_name <> employee_name`: 자기 자신을 관리할 수 없음

**5. ON DELETE CASCADE:**
- 직원 삭제 시 근무 기록과 관리 관계도 자동 삭제
- 회사 삭제 시 근무 기록도 자동 삭제

**대안 설계:**
```sql
-- person_name이 유일하지 않을 수 있는 경우
CREATE TABLE employee (
    emp_id SERIAL PRIMARY KEY,  -- 자동 증가 ID
    person_name VARCHAR(100) NOT NULL,
    street VARCHAR(150),
    city VARCHAR(100),
    UNIQUE (person_name, street, city)  -- 이름+주소 조합으로 유일성 보장
);

-- works 테이블도 emp_id 사용
CREATE TABLE works (
    emp_id INTEGER,
    company_name VARCHAR(100),
    salary NUMERIC(10, 2),
    PRIMARY KEY (emp_id, company_name),
    FOREIGN KEY (emp_id) REFERENCES employee(emp_id)
);
```

---

## 핵심 개념 정리

### 1. SQL 쿼리 작성 패턴

**기본 패턴:**
```sql
SELECT [DISTINCT] 컬럼들
FROM 테이블들
[JOIN ... ON ...]
WHERE 조건
[GROUP BY 그룹화_컬럼]
[HAVING 그룹_조건]
[ORDER BY 정렬_컬럼];
```

### 2. 집계 함수
- `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()`
- `GROUP BY`와 함께 사용
- `HAVING`으로 그룹 필터링

### 3. 부질의 (Subquery)
- **스칼라 부질의**: 단일 값 반환
- **테이블 부질의**: 여러 행 반환 (IN, EXISTS)
- **상관 부질의**: 외부 쿼리 참조

### 4. JOIN 유형
- **INNER JOIN**: 매칭되는 행만
- **LEFT/RIGHT JOIN**: 한쪽 테이블의 모든 행
- **NATURAL JOIN**: 같은 이름 속성으로 자동 조인

### 5. DDL 제약조건
- **PRIMARY KEY**: 유일성 + NOT NULL
- **FOREIGN KEY**: 참조 무결성
- **CHECK**: 값 범위 제한
- **UNIQUE**: 유일성 (NULL 허용)
- **NOT NULL**: NULL 불허

### 6. 참조 동작
- **ON DELETE CASCADE**: 연쇄 삭제
- **ON DELETE RESTRICT**: 삭제 거부
- **ON DELETE SET NULL**: NULL로 설정
- **ON UPDATE CASCADE**: 연쇄 업데이트

---

## 다음 단계

**Part 2 (Exercises 3.19-3.27)에서 다룰 내용:**
- NULL 값 처리
- `<> ALL`과 `NOT IN` 비교
- Library 데이터베이스 쿼리
- `UNIQUE` 구문 재작성
- `WITH` 절 변환
- 복잡한 집계 및 부질의

**Part 3 (Exercises 3.28-3.35)에서 다룰 내용:**
- Division 연산 (모든/every 쿼리)
- 고급 집계 쿼리
- NULL과 집계 함수의 상호작용
- 복잡한 GROUP BY와 HAVING
- 최대/최소 값 찾기

---

## 참고 자료

- Database System Concepts (7th Edition) - Silberschatz, Korth, Sudarshan
- SQL Standard (ISO/IEC 9075)
- PostgreSQL Documentation
- MySQL Reference Manual
