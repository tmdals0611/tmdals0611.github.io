---
title: "[데이터베이스] 문제 해설 - SQL 고급 Part 3 (Exercises 3.28-3.35)"
date: 2025-01-30 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [sql, advanced-queries, division-operation, aggregation, null-handling, window-functions, complex-queries, problem-solving]
pin: false
math: true
mermaid: false
---

## 개요

이 포스트는 SQL 고급 쿼리에 관한 연습 문제들(Exercises 3.28-3.35)의 상세한 해설을 제공합니다. Division 연산(모든 과목 가르치는 교수), NULL과 집계 함수의 상호작용, 복잡한 필터링 조건, 최대값 찾기 등 실무에서 자주 마주치는 복잡한 SQL 패턴을 다룹니다.

---

## Exercise 3.28: Instructors Teaching All Department Courses

### 문제
> Using the university schema, write an SQL query to find the names and IDs of those instructors who teach every course taught in his or her department (i.e., every course that appears in the course relation with the instructor's department name). Order result by name.

### 분석

이것은 전형적인 **Division 연산** 문제입니다.
- "모든(every)" 키워드 → Division
- 조건: 교수가 **자신의 학과에서 개설된 모든 과목**을 가르침

### SQL 쿼리 (이중 부정)

```sql
SELECT i.ID, i.name
FROM instructor i
WHERE NOT EXISTS (
    -- 이 교수의 학과 과목 중에서
    SELECT c.course_id
    FROM course c
    WHERE c.dept_name = i.dept_name
      AND NOT EXISTS (
          -- 이 교수가 가르치지 않는 과목이 없다
          SELECT *
          FROM teaches t
          WHERE t.ID = i.ID
            AND t.course_id = c.course_id
      )
)
ORDER BY i.name;
```

**해석:**
- "이 교수의 학과 과목 중에서, 이 교수가 가르치지 않는 과목이 존재하지 않는다"
- = "자신의 학과 모든 과목을 가르친다"

### 대안: GROUP BY HAVING

```sql
SELECT i.ID, i.name
FROM instructor i
WHERE i.ID IN (
    SELECT t.ID
    FROM teaches t
    JOIN course c ON t.course_id = c.course_id
    WHERE c.dept_name = i.dept_name
    GROUP BY t.ID
    HAVING COUNT(DISTINCT t.course_id) = (
        SELECT COUNT(*)
        FROM course c2
        WHERE c2.dept_name = i.dept_name
    )
)
ORDER BY i.name;
```

**해석:**
- 교수가 가르친 자기 학과 과목 수 = 자기 학과 총 과목 수

### 예시 데이터

```
instructor:
┌───────┬──────────┬───────────┬────────┐
│  ID   │   name   │ dept_name │ salary │
├───────┼──────────┼───────────┼────────┤
│ 10101 │ Sriniva  │ Comp.Sci. │ 65000  │
│ 12121 │ Wu       │ Finance   │ 90000  │
│ 15151 │ Mozart   │ Music     │ 40000  │
│ 22222 │ Einstein │ Physics   │ 95000  │
└───────┴──────────┴───────────┴────────┘

course:
┌───────────┬─────────────────────┬───────────┐
│ course_id │        title        │ dept_name │
├───────────┼─────────────────────┼───────────┤
│ CS-101    │ Intro to CS         │ Comp.Sci. │
│ CS-190    │ Game Design         │ Comp.Sci. │
│ CS-315    │ Robotics            │ Comp.Sci. │
│ FIN-201   │ Investment Banking  │ Finance   │
│ MU-199    │ Music Video Prod.   │ Music     │
│ PHY-101   │ Physical Principles │ Physics   │
└───────────┴─────────────────────┴───────────┘

teaches:
┌───────┬───────────┬────────┬──────────┬──────┐
│  ID   │ course_id │ sec_id │ semester │ year │
├───────┼───────────┼────────┼──────────┼──────┤
│ 10101 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 10101 │ CS-190    │ 1      │ Spring   │ 2017 │
│ 10101 │ CS-315    │ 1      │ Spring   │ 2018 │  ← Comp.Sci. 3개 모두
│ 12121 │ FIN-201   │ 1      │ Spring   │ 2018 │  ← Finance 1개 (전부)
│ 15151 │ MU-199    │ 1      │ Spring   │ 2017 │  ← Music 1개 (전부)
│ 22222 │ PHY-101   │ 1      │ Fall     │ 2017 │  ← Physics 1개 (전부)
└───────┴───────────┴────────┴──────────┴──────┘

분석:
- Comp.Sci.: 3개 과목, Sriniva가 3개 모두 가르침 ✓
- Finance: 1개 과목, Wu가 1개 가르침 ✓
- Music: 1개 과목, Mozart가 1개 가르침 ✓
- Physics: 1개 과목, Einstein이 1개 가르침 ✓

결과 (이름순):
┌───────┬──────────┐
│  ID   │   name   │
├───────┼──────────┤
│ 22222 │ Einstein │
│ 15151 │ Mozart   │
│ 10101 │ Sriniva  │
│ 12121 │ Wu       │
└───────┴──────────┘
```

### 엣지 케이스

**Case 1: 학과에 과목이 없는 경우**
```sql
-- 과목이 없는 학과의 모든 교수가 선택됨 (공허하게 참)
-- WHERE NOT EXISTS (SELECT ... FROM course WHERE ...)
-- → 내부 쿼리가 빈 집합 → NOT EXISTS TRUE
```

**방지 방법:**
```sql
SELECT i.ID, i.name
FROM instructor i
WHERE EXISTS (
    SELECT * FROM course WHERE dept_name = i.dept_name
)
AND NOT EXISTS (
    SELECT c.course_id
    FROM course c
    WHERE c.dept_name = i.dept_name
      AND NOT EXISTS (
          SELECT * FROM teaches t
          WHERE t.ID = i.ID AND t.course_id = c.course_id
      )
)
ORDER BY i.name;
```

**Case 2: 교수가 아무 과목도 안 가르치는 경우**
- 학과에 과목이 있으면 제외됨 (올바름)
- 학과에 과목이 없으면 포함됨 (위 방지 방법으로 해결)

---

## Exercise 3.29: History Students Starting with 'D', Not Taking 5+ Music Courses

### 문제
> Using the university schema, write an SQL query to find the name and ID of each History student whose name begins with the letter 'D' and who has not taken at least five Music courses.

### SQL 쿼리

```sql
SELECT s.ID, s.name
FROM student s
WHERE s.dept_name = 'History'
  AND s.name LIKE 'D%'
  AND (
      SELECT COUNT(*)
      FROM takes t
      JOIN course c ON t.course_id = c.course_id
      WHERE t.ID = s.ID
        AND c.dept_name = 'Music'
  ) < 5;
```

**설명:**
1. `dept_name = 'History'`: History 학과 학생
2. `name LIKE 'D%'`: 이름이 'D'로 시작
3. 부질의: Music 과목 수강 수 세기
4. `< 5`: 5개 미만

### 대안: LEFT JOIN

```sql
SELECT s.ID, s.name
FROM student s
LEFT JOIN (
    SELECT t.ID, COUNT(*) AS music_count
    FROM takes t
    JOIN course c ON t.course_id = c.course_id
    WHERE c.dept_name = 'Music'
    GROUP BY t.ID
) AS music_courses ON s.ID = music_courses.ID
WHERE s.dept_name = 'History'
  AND s.name LIKE 'D%'
  AND COALESCE(music_courses.music_count, 0) < 5;
```

**설명:**
- `COALESCE(music_count, 0)`: Music 과목을 안 들은 경우 0으로 처리

### 예시 데이터

```
student:
┌───────┬─────────┬───────────┬──────────┐
│  ID   │  name   │ dept_name │ tot_cred │
├───────┼─────────┼───────────┼──────────┤
│ 12345 │ Shankar │ Comp.Sci. │ 32       │
│ 19991 │ Davis   │ History   │ 80       │  ← History, D로 시작
│ 23121 │ Chavez  │ Finance   │ 110      │
│ 44553 │ Diana   │ History   │ 56       │  ← History, D로 시작
│ 54321 │ Drake   │ History   │ 45       │  ← History, D로 시작
└───────┴─────────┴───────────┴──────────┘

takes (Music 과목만):
┌───────┬───────────┬───────────┐
│  ID   │ course_id │ dept_name │
├───────┼───────────┼───────────┤
│ 19991 │ MU-101    │ Music     │  (Davis: 2개)
│ 19991 │ MU-199    │ Music     │
│ 44553 │ MU-101    │ Music     │  (Diana: 5개)
│ 44553 │ MU-102    │ Music     │
│ 44553 │ MU-199    │ Music     │
│ 44553 │ MU-201    │ Music     │
│ 44553 │ MU-301    │ Music     │
└───────┴───────────┴───────────┘
-- Drake는 Music 과목 없음

분석:
- Davis: History, D로 시작, Music 2개 (< 5) ✓
- Diana: History, D로 시작, Music 5개 (≥ 5) ✗
- Drake: History, D로 시작, Music 0개 (< 5) ✓

결과:
┌───────┬───────┐
│  ID   │ name  │
├───────┼───────┤
│ 19991 │ Davis │
│ 54321 │ Drake │
└───────┴───────┘
```

### LIKE 패턴 매칭

**패턴:**
- `%`: 0개 이상의 임의 문자
- `_`: 정확히 1개의 임의 문자
- `[...]`: 문자 집합 (SQL Server, 일부 DBMS)
- `ESCAPE`: 이스케이프 문자 지정

**예시:**
```sql
-- D로 시작
WHERE name LIKE 'D%'

-- D로 시작하고 정확히 5글자
WHERE name LIKE 'D____'  -- 4개의 underscore

-- Da 또는 Di로 시작
WHERE name LIKE 'Da%' OR name LIKE 'Di%'

-- 대소문자 무시 (PostgreSQL)
WHERE name ILIKE 'd%'

-- % 문자 자체 찾기
WHERE description LIKE '%\%%' ESCAPE '\'
-- '50% off' 같은 문자열 찾기
```

---

## Exercise 3.30: AVG vs SUM/COUNT Paradox

### 문제
> Consider the following SQL query on the university schema:
> ```sql
> SELECT AVG(salary) - (SUM(salary) / COUNT(*))
> FROM instructor
> ```
> We might expect that the result of this query is zero since the average of a set of numbers is defined to be the sum of the numbers divided by the number of numbers. However, there are other possible instances of that relation for which the result would not be zero. Give one such instance, and explain why the result would not be zero.

### 분석

**수학적으로:**
$$\text{AVG}(X) = \frac{\text{SUM}(X)}{\text{COUNT}(X)}$$

**하지만 SQL에서:**
- `AVG(salary)`: **NULL을 제외**하고 평균 계산
- `SUM(salary)`: **NULL을 제외**하고 합계 계산
- `COUNT(*)`: **모든 행** 개수 (NULL 포함)

**따라서:**
$$\text{AVG(salary)} = \frac{\text{SUM(salary)}}{\text{COUNT(salary)}}$$
$$\frac{\text{SUM(salary)}}{\text{COUNT(*}} \neq \frac{\text{SUM(salary)}}{\text{COUNT(salary)}} \text{ if NULL exists}$$

### 예시 인스턴스

```
instructor:
┌───────┬──────────┬───────────┬────────┐
│  ID   │   name   │ dept_name │ salary │
├───────┼──────────┼───────────┼────────┤
│ 10101 │ Sriniva  │ Comp.Sci. │ 60000  │
│ 12121 │ Wu       │ Finance   │ 90000  │
│ 15151 │ Mozart   │ Music     │ NULL   │
│ 22222 │ Einstein │ Physics   │ NULL   │
└───────┴──────────┴───────────┴────────┘
```

**계산:**

```sql
SELECT AVG(salary) - (SUM(salary) / COUNT(*))
FROM instructor;
```

1. `AVG(salary)`:
   - NULL 제외: [60000, 90000]
   - `(60000 + 90000) / 2 = 75000`

2. `SUM(salary)`:
   - NULL 제외: `60000 + 90000 = 150000`

3. `COUNT(*)`:
   - 모든 행: `4`

4. `SUM(salary) / COUNT(*)`:
   - `150000 / 4 = 37500`

5. **결과:**
   - `75000 - 37500 = 37500` ≠ 0

### 왜 다른가?

**AVG의 분모:** `COUNT(salary)` = 2 (NULL 제외)
**계산식의 분모:** `COUNT(*)` = 4 (NULL 포함)

$$\frac{150000}{2} - \frac{150000}{4} = 75000 - 37500 = 37500$$

### 일반화

**NULL이 n개 있을 때:**
- `AVG(salary)`: NULL을 제외한 평균
- `SUM(salary) / COUNT(*)`: 모든 행으로 나눔

**차이:**
$$\text{AVG(salary)} - \frac{\text{SUM(salary)}}{\text{COUNT(*)}} = \text{AVG(salary)} \times \frac{n}{\text{total\_rows}}$$

**NULL이 없으면:**
- `COUNT(*) = COUNT(salary)`
- 결과 = 0 ✓

### NULL-safe 버전

```sql
-- 올바른 비교
SELECT AVG(salary) - (SUM(salary) / COUNT(salary))
FROM instructor;
-- 항상 0

-- 또는 NULL을 명시적으로 처리
SELECT AVG(COALESCE(salary, 0)) - (SUM(COALESCE(salary, 0)) / COUNT(*))
FROM instructor;
-- NULL을 0으로 취급
```

### 교훈

1. **AVG는 NULL을 무시함**
2. **COUNT(*)는 모든 행을 셈**
3. **COUNT(column)은 NULL을 제외함**
4. **집계 함수의 NULL 처리를 항상 고려해야 함**

---

## Exercise 3.31: Instructors Who Never Gave an A Grade

### 문제
> Using the university schema, write an SQL query to find the ID and name of each instructor who has never given an A grade in any course she or he has taught. (Instructors who have never taught a course trivially satisfy this condition.)

### SQL 쿼리

```sql
SELECT i.ID, i.name
FROM instructor i
WHERE NOT EXISTS (
    SELECT *
    FROM teaches t
    JOIN takes tk ON t.course_id = tk.course_id
                  AND t.sec_id = tk.sec_id
                  AND t.semester = tk.semester
                  AND t.year = tk.year
    WHERE t.ID = i.ID
      AND tk.grade = 'A'
);
```

**설명:**
- "이 교수가 가르친 섹션에서 'A' 학점을 받은 학생이 존재하지 않는다"
- 한 번도 가르치지 않은 교수도 포함됨 (NOT EXISTS → TRUE)

### 대안: NOT IN

```sql
SELECT i.ID, i.name
FROM instructor i
WHERE i.ID NOT IN (
    SELECT DISTINCT t.ID
    FROM teaches t
    JOIN takes tk ON t.course_id = tk.course_id
                  AND t.sec_id = tk.sec_id
                  AND t.semester = tk.semester
                  AND t.year = tk.year
    WHERE tk.grade = 'A'
);
```

### 예시 데이터

```
instructor:
┌───────┬──────────┬───────────┐
│  ID   │   name   │ dept_name │
├───────┼──────────┼───────────┤
│ 10101 │ Sriniva  │ Comp.Sci. │
│ 12121 │ Wu       │ Finance   │
│ 15151 │ Mozart   │ Music     │
│ 22222 │ Einstein │ Physics   │
│ 33456 │ Gold     │ Physics   │  ← 한 번도 안 가르침
└───────┴──────────┴───────────┘

teaches:
┌───────┬───────────┬────────┬──────────┬──────┐
│  ID   │ course_id │ sec_id │ semester │ year │
├───────┼───────────┼────────┼──────────┼──────┤
│ 10101 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 12121 │ FIN-201   │ 1      │ Spring   │ 2018 │
│ 15151 │ MU-199    │ 1      │ Spring   │ 2017 │
│ 22222 │ PHY-101   │ 1      │ Fall     │ 2017 │
└───────┴───────────┴────────┴──────────┴──────┘

takes:
┌───────┬───────────┬────────┬──────────┬──────┬───────┐
│  ID   │ course_id │ sec_id │ semester │ year │ grade │
├───────┼───────────┼────────┼──────────┼──────┼───────┤
│ 12345 │ CS-101    │ 1      │ Fall     │ 2017 │ A     │  ← Sriniva가 A 줌
│ 23121 │ FIN-201   │ 1      │ Spring   │ 2018 │ B     │  ← Wu는 A 안 줌
│ 44553 │ MU-199    │ 1      │ Spring   │ 2017 │ B+    │  ← Mozart는 A 안 줌
│ 45678 │ PHY-101   │ 1      │ Fall     │ 2017 │ C     │  ← Einstein은 A 안 줌
└───────┴───────────┴────────┴──────────┴──────┴───────┘

분석:
- Sriniva (10101): A 학점 줌 ✗
- Wu (12121): A 학점 안 줌 ✓
- Mozart (15151): A 학점 안 줌 ✓
- Einstein (22222): A 학점 안 줌 ✓
- Gold (33456): 가르친 적 없음 ✓ (trivially satisfy)

결과:
┌───────┬──────────┐
│  ID   │   name   │
├───────┼──────────┤
│ 12121 │ Wu       │
│ 15151 │ Mozart   │
│ 22222 │ Einstein │
│ 33456 │ Gold     │
└───────┴──────────┘
```

### NULL Grade 처리

**문제:** `grade`가 NULL인 경우는?

```sql
-- NULL grade는 'A'가 아니므로 조건 만족
-- 하지만 명시적으로 처리하려면:

WHERE NOT EXISTS (
    ...
    WHERE t.ID = i.ID
      AND tk.grade = 'A'
      AND tk.grade IS NOT NULL  -- 명시적 NULL 제외
);
```

---

## Exercise 3.32: Instructors Who Gave At Least One Non-Null Grade (But No A)

### 문제
> Rewrite the preceding query, but also ensure that you include only instructors who have given at least one other non-null grade in some course.

### SQL 쿼리

```sql
SELECT i.ID, i.name
FROM instructor i
WHERE EXISTS (
    -- 적어도 하나의 non-null grade를 줌
    SELECT *
    FROM teaches t
    JOIN takes tk ON t.course_id = tk.course_id
                  AND t.sec_id = tk.sec_id
                  AND t.semester = tk.semester
                  AND t.year = tk.year
    WHERE t.ID = i.ID
      AND tk.grade IS NOT NULL
)
AND NOT EXISTS (
    -- 하지만 'A'는 안 줌
    SELECT *
    FROM teaches t
    JOIN takes tk ON t.course_id = tk.course_id
                  AND t.sec_id = tk.sec_id
                  AND t.semester = tk.semester
                  AND t.year = tk.year
    WHERE t.ID = i.ID
      AND tk.grade = 'A'
);
```

**설명:**
1. **첫 번째 EXISTS**: 적어도 하나의 non-null grade 부여
2. **두 번째 NOT EXISTS**: 'A' 학점은 부여하지 않음

### 대안: WITH 절

```sql
WITH instructor_grades AS (
    SELECT t.ID,
           COUNT(*) AS total_grades,
           SUM(CASE WHEN tk.grade = 'A' THEN 1 ELSE 0 END) AS a_grades
    FROM teaches t
    JOIN takes tk ON t.course_id = tk.course_id
                  AND t.sec_id = tk.sec_id
                  AND t.semester = tk.semester
                  AND t.year = tk.year
    WHERE tk.grade IS NOT NULL
    GROUP BY t.ID
)
SELECT i.ID, i.name
FROM instructor i
JOIN instructor_grades ig ON i.ID = ig.ID
WHERE ig.total_grades > 0
  AND ig.a_grades = 0;
```

### 예시 데이터

```
teaches + takes:
┌───────┬───────────┬───────┐
│  ID   │   name    │ grade │
├───────┼───────────┼───────┤
│ 10101 │ Sriniva   │ A     │  ← A 줌
│ 10101 │ Sriniva   │ B     │
│ 12121 │ Wu        │ B     │  ← A 안 줌, non-null 있음
│ 12121 │ Wu        │ C     │
│ 15151 │ Mozart    │ NULL  │  ← non-null 없음
│ 22222 │ Einstein  │ NULL  │  ← non-null 없음
│ 33456 │ Gold      │ -     │  ← 가르친 적 없음
└───────┴───────────┴───────┘

분석:
- Sriniva: A 줌 ✗
- Wu: A 안 줌, non-null 있음 ✓
- Mozart: non-null 없음 ✗
- Einstein: non-null 없음 ✗
- Gold: 가르친 적 없음 ✗

결과:
┌───────┬──────┐
│  ID   │ name │
├───────┼──────┤
│ 12121 │ Wu   │
└───────┴──────┘
```

### Exercise 3.31과의 차이

| 조건 | 3.31 | 3.32 |
|------|------|------|
| A를 안 줌 | ✓ | ✓ |
| 가르친 적 없음 | ✓ (포함) | ✗ (제외) |
| NULL grade만 있음 | ✓ (포함) | ✗ (제외) |
| Non-null grade 있음 | ✓ | ✓ (필수) |

---

## Exercise 3.33: Comp. Sci. Courses with Afternoon Sections

### 문제
> Using the university schema, write an SQL query to find the ID and title of each course in Comp. Sci. that has had at least one section with afternoon hours (i.e., ends at or after 12:00). (You should eliminate duplicates if any.)

### 가정

`time_slot` 테이블이 있다고 가정:
```
time_slot(time_slot_id, day, start_time, end_time)
```

### SQL 쿼리

```sql
SELECT DISTINCT c.course_id, c.title
FROM course c
JOIN section s ON c.course_id = s.course_id
JOIN time_slot ts ON s.time_slot_id = ts.time_slot_id
WHERE c.dept_name = 'Comp. Sci.'
  AND ts.end_time >= '12:00';
```

**설명:**
1. `course` ← `section`: 과목의 섹션들
2. `section` ← `time_slot`: 섹션의 시간대
3. `WHERE`: Comp. Sci. 과목 & 종료 시간 12:00 이후
4. `DISTINCT`: 중복 제거 (여러 섹션이 조건 만족 가능)

### 예시 데이터

```
course (Comp. Sci.):
┌───────────┬─────────────────────┬───────────┐
│ course_id │        title        │ dept_name │
├───────────┼─────────────────────┼───────────┤
│ CS-101    │ Intro to CS         │ Comp.Sci. │
│ CS-190    │ Game Design         │ Comp.Sci. │
│ CS-315    │ Robotics            │ Comp.Sci. │
└───────────┴─────────────────────┴───────────┘

section:
┌───────────┬────────┬──────────┬──────┬───────────────┐
│ course_id │ sec_id │ semester │ year │ time_slot_id  │
├───────────┼────────┼──────────┼──────┼───────────────┤
│ CS-101    │ 1      │ Fall     │ 2017 │ A             │
│ CS-101    │ 2      │ Spring   │ 2018 │ B             │
│ CS-190    │ 1      │ Spring   │ 2017 │ C             │
│ CS-315    │ 1      │ Spring   │ 2018 │ D             │
└───────────┴────────┴──────────┴──────┴───────────────┘

time_slot:
┌───────────────┬─────┬────────────┬──────────┐
│ time_slot_id  │ day │ start_time │ end_time │
├───────────────┼─────┼────────────┼──────────┤
│ A             │ M   │ 08:00      │ 09:00    │  ← 오전
│ B             │ W   │ 13:00      │ 14:30    │  ← 오후 ✓
│ C             │ F   │ 11:30      │ 12:45    │  ← 오후 ✓
│ D             │ M   │ 10:00      │ 11:00    │  ← 오전
└───────────────┴─────┴────────────┴──────────┘

조인 결과:
┌───────────┬─────────────────────┬──────────┐
│ course_id │        title        │ end_time │
├───────────┼─────────────────────┼──────────┤
│ CS-101    │ Intro to CS         │ 09:00    │
│ CS-101    │ Intro to CS         │ 14:30    │  ← 오후 ✓
│ CS-190    │ Game Design         │ 12:45    │  ← 오후 ✓
│ CS-315    │ Robotics            │ 11:00    │
└───────────┴─────────────────────┴──────────┘

결과 (DISTINCT):
┌───────────┬─────────────────────┐
│ course_id │        title        │
├───────────┼─────────────────────┤
│ CS-101    │ Intro to CS         │
│ CS-190    │ Game Design         │
└───────────┴─────────────────────┘
```

### 시간 비교

**시간 형식:**
- `TIME`: '12:00:00'
- `VARCHAR`: '12:00'

**비교 연산:**
```sql
-- TIME 타입
WHERE ts.end_time >= TIME '12:00:00'

-- VARCHAR (문자열 비교도 가능, 형식이 일관되면)
WHERE ts.end_time >= '12:00'

-- 시간 추출 (TIMESTAMP에서)
WHERE EXTRACT(HOUR FROM ts.end_time) >= 12
```

---

## Exercise 3.34: Number of Students in Each Section

### 문제
> Using the university schema, write an SQL query to find the number of students in each section. The result columns should appear in the order "courseid, secid, year, semester, num". You do not need to output sections with 0 students.

### SQL 쿼리

```sql
SELECT course_id, sec_id, year, semester, COUNT(*) AS num
FROM takes
GROUP BY course_id, sec_id, year, semester
ORDER BY course_id, sec_id, year, semester;
```

**설명:**
1. `GROUP BY`: 섹션별 그룹화 (course_id, sec_id, year, semester)
2. `COUNT(*)`: 각 그룹의 학생 수
3. `ORDER BY`: 지정된 순서로 정렬

### 0명 포함 버전 (문제에서 요구하지 않음)

```sql
SELECT s.course_id, s.sec_id, s.year, s.semester,
       COUNT(t.ID) AS num
FROM section s
LEFT JOIN takes t ON s.course_id = t.course_id
                  AND s.sec_id = t.sec_id
                  AND s.semester = t.semester
                  AND s.year = t.year
GROUP BY s.course_id, s.sec_id, s.year, s.semester
ORDER BY s.course_id, s.sec_id, s.year, s.semester;
```

**차이:**
- `COUNT(*)`: 모든 행 (LEFT JOIN이면 항상 ≥1)
- `COUNT(t.ID)`: NULL 제외 (수강생이 없으면 0)

### 예시 데이터

```
takes:
┌───────┬───────────┬────────┬──────────┬──────┐
│  ID   │ course_id │ sec_id │ semester │ year │
├───────┼───────────┼────────┼──────────┼──────┤
│ 12345 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 23121 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 44553 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 12345 │ CS-190    │ 1      │ Spring   │ 2017 │
│ 23121 │ CS-190    │ 1      │ Spring   │ 2017 │
│ 19991 │ FIN-201   │ 1      │ Spring   │ 2018 │
└───────┴───────────┴────────┴──────────┴──────┘

결과:
┌───────────┬────────┬──────┬──────────┬─────┐
│ course_id │ sec_id │ year │ semester │ num │
├───────────┼────────┼──────┼──────────┼─────┤
│ CS-101    │ 1      │ 2017 │ Fall     │ 3   │
│ CS-190    │ 1      │ 2017 │ Spring   │ 2   │
│ FIN-201   │ 1      │ 2018 │ Spring   │ 1   │
└───────────┴────────┴──────┴──────────┴─────┘
```

### 추가 정보 포함

```sql
-- 과목 제목 포함
SELECT s.course_id, c.title, s.sec_id, s.year, s.semester,
       COUNT(t.ID) AS num
FROM section s
JOIN course c ON s.course_id = c.course_id
LEFT JOIN takes t ON s.course_id = t.course_id
                  AND s.sec_id = t.sec_id
                  AND s.semester = t.semester
                  AND s.year = t.year
GROUP BY s.course_id, c.title, s.sec_id, s.year, s.semester
ORDER BY s.course_id, s.sec_id, s.year, s.semester;
```

---

## Exercise 3.35: Sections with Maximum Enrollment

### 문제
> Using the university schema, write an SQL query to find section(s) with maximum enrollment. The result columns should appear in the order "courseid, secid, year, semester, num". (It may be convenient to use the `with` construct.)

### SQL 쿼리 (WITH 절 사용)

```sql
WITH section_enrollment AS (
    SELECT course_id, sec_id, year, semester, COUNT(*) AS num
    FROM takes
    GROUP BY course_id, sec_id, year, semester
)
SELECT course_id, sec_id, year, semester, num
FROM section_enrollment
WHERE num = (SELECT MAX(num) FROM section_enrollment);
```

**설명:**
1. **CTE**: 각 섹션의 등록 학생 수 계산
2. **WHERE**: 최대 등록 수와 같은 섹션들 선택

### 대안: 중첩 부질의

```sql
SELECT course_id, sec_id, year, semester, num
FROM (
    SELECT course_id, sec_id, year, semester, COUNT(*) AS num
    FROM takes
    GROUP BY course_id, sec_id, year, semester
) AS section_counts
WHERE num = (
    SELECT MAX(num)
    FROM (
        SELECT COUNT(*) AS num
        FROM takes
        GROUP BY course_id, sec_id, year, semester
    ) AS max_count
);
```

### 대안: ALL 사용

```sql
SELECT course_id, sec_id, year, semester, COUNT(*) AS num
FROM takes
GROUP BY course_id, sec_id, year, semester
HAVING COUNT(*) >= ALL (
    SELECT COUNT(*)
    FROM takes
    GROUP BY course_id, sec_id, year, semester
);
```

**설명:**
- `>= ALL`: 모든 값보다 크거나 같음 (= 최댓값)

### 예시 데이터

```
takes:
┌───────┬───────────┬────────┬──────────┬──────┐
│  ID   │ course_id │ sec_id │ semester │ year │
├───────┼───────────┼────────┼──────────┼──────┤
│ 12345 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 23121 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 44553 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 54321 │ CS-101    │ 1      │ Fall     │ 2017 │
│ 76543 │ CS-101    │ 1      │ Fall     │ 2017 │  (5명)
│ 12345 │ CS-190    │ 1      │ Spring   │ 2017 │
│ 23121 │ CS-190    │ 1      │ Spring   │ 2017 │
│ 44553 │ CS-190    │ 1      │ Spring   │ 2017 │
│ 54321 │ CS-190    │ 1      │ Spring   │ 2017 │
│ 76543 │ CS-190    │ 1      │ Spring   │ 2017 │  (5명)
│ 19991 │ FIN-201   │ 1      │ Spring   │ 2018 │
│ 23121 │ FIN-201   │ 1      │ Spring   │ 2018 │  (2명)
└───────┴───────────┴────────┴──────────┴──────┘

섹션별 등록 수:
┌───────────┬────────┬──────┬──────────┬─────┐
│ course_id │ sec_id │ year │ semester │ num │
├───────────┼────────┼──────┼──────────┼─────┤
│ CS-101    │ 1      │ 2017 │ Fall     │ 5   │  ← 최대
│ CS-190    │ 1      │ 2017 │ Spring   │ 5   │  ← 최대
│ FIN-201   │ 1      │ 2018 │ Spring   │ 2   │
└───────────┴────────┴──────┴──────────┴─────┘

최댓값: 5

결과 (최댓값과 같은 모든 섹션):
┌───────────┬────────┬──────┬──────────┬─────┐
│ course_id │ sec_id │ year │ semester │ num │
├───────────┼────────┼──────┼──────────┼─────┤
│ CS-101    │ 1      │ 2017 │ Fall     │ 5   │
│ CS-190    │ 1      │ 2017 │ Spring   │ 5   │
└───────────┴────────┴──────┴──────────┴─────┘
```

### 추가 정보 포함

```sql
WITH section_enrollment AS (
    SELECT s.course_id, c.title, s.sec_id, s.year, s.semester,
           COUNT(t.ID) AS num
    FROM section s
    JOIN course c ON s.course_id = c.course_id
    LEFT JOIN takes t ON s.course_id = t.course_id
                      AND s.sec_id = t.sec_id
                      AND s.semester = t.semester
                      AND s.year = t.year
    GROUP BY s.course_id, c.title, s.sec_id, s.year, s.semester
)
SELECT course_id, title, sec_id, year, semester, num
FROM section_enrollment
WHERE num = (SELECT MAX(num) FROM section_enrollment);
```

### Top N 섹션 찾기

```sql
-- 상위 3개 섹션
WITH section_enrollment AS (
    SELECT course_id, sec_id, year, semester, COUNT(*) AS num
    FROM takes
    GROUP BY course_id, sec_id, year, semester
)
SELECT course_id, sec_id, year, semester, num
FROM section_enrollment
ORDER BY num DESC
LIMIT 3;

-- 또는 RANK 함수 사용 (동점 처리)
WITH section_enrollment AS (
    SELECT course_id, sec_id, year, semester, COUNT(*) AS num,
           RANK() OVER (ORDER BY COUNT(*) DESC) AS rank
    FROM takes
    GROUP BY course_id, sec_id, year, semester
)
SELECT course_id, sec_id, year, semester, num
FROM section_enrollment
WHERE rank <= 3;
```

---

## 핵심 개념 정리

### 1. Division 연산 (모든/every)
- **이중 부정**: `NOT EXISTS ... NOT EXISTS`
- **GROUP BY HAVING**: 개수 비교
- **공허하게 참**: 집합이 비었을 때 주의

### 2. NULL과 집계 함수
- `AVG(col)`: NULL 제외하고 평균
- `SUM(col)`: NULL 제외하고 합계
- `COUNT(*)`: 모든 행 (NULL 포함)
- `COUNT(col)`: NULL 제외

### 3. 복잡한 필터링
- `LIKE` 패턴 매칭
- 부질의로 개수 세기
- `EXISTS`로 존재 여부 확인

### 4. 최댓값/최솟값 찾기
- `WHERE value = (SELECT MAX(value) ...)`
- `HAVING value >= ALL (...)`
- `WITH` 절로 가독성 향상
- `RANK()` 윈도우 함수

### 5. 조인과 집계
- `LEFT JOIN`: 0개 포함
- `COUNT(*)` vs `COUNT(col)`: NULL 처리
- `GROUP BY`: 여러 컬럼 조합

### 6. 쿼리 최적화
- `WITH` 절로 중복 계산 방지
- 인덱스 활용 (WHERE, JOIN 키)
- `EXISTS` vs `IN`: 대량 데이터에서 차이

---

## 실무 적용 팁

### 1. Division 쿼리 작성법
```
"모든 X를 만족하는 Y 찾기"
→ "X 중에서, Y가 만족하지 않는 것이 존재하지 않는다"
→ NOT EXISTS (X WHERE NOT EXISTS (Y satisfies X))
```

### 2. NULL 안전 쿼리
- 항상 NULL 가능성 고려
- `COALESCE` 또는 `IS NULL` 명시
- 집계 함수의 NULL 처리 확인

### 3. 복잡한 쿼리 단계별 작성
1. 내부에서 외부로
2. WITH 절로 명명
3. 각 단계 검증
4. 최종 결합

### 4. 성능 고려사항
- `EXISTS` > `IN` (대량 데이터)
- 인덱스: JOIN 키, WHERE 조건
- `LIMIT` 사용으로 불필요한 데이터 제한

### 5. 가독성
- WITH 절 적극 활용
- 의미 있는 별칭
- 주석 추가
- 일관된 들여쓰기

---

## 전체 시리즈 요약

### Part 1 (3.11-3.18)
- 기본 SELECT, JOIN, 집계
- INSERT, UPDATE, DELETE
- DDL과 제약조건

### Part 2 (3.19-3.27)
- NULL 값 처리
- `<> ALL` vs `NOT IN`
- UNIQUE, WITH 절
- Library 데이터베이스

### Part 3 (3.28-3.35)
- Division 연산
- 복잡한 필터링
- NULL과 집계의 상호작용
- 최댓값 찾기

---

## 참고 자료

- Database System Concepts (7th Edition) - Silberschatz, Korth, Sudarshan
- SQL Performance Explained - Markus Winand
- "SQL Antipatterns" - Bill Karwin
- PostgreSQL Documentation
- MySQL Reference Manual

---

## 마무리

이 세 개의 파트를 통해 SQL의 기초부터 고급 쿼리까지 다루었습니다. 실무에서는 이러한 패턴들을 조합하여 더욱 복잡한 문제를 해결하게 됩니다. 항상 다음을 기억하세요:

1. **가독성이 최우선**: 복잡한 쿼리는 WITH 절로 분해
2. **NULL을 항상 고려**: 집계, 비교, 조인에서
3. **성능을 측정**: 추측 대신 EXPLAIN 사용
4. **테스트 철저히**: 엣지 케이스 확인
5. **문서화**: 복잡한 로직은 주석 추가
