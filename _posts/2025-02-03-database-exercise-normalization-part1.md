---
title: "[데이터베이스] 문제 해설 - 정규화 Part 1 (Exercises 7.21-7.28)"
date: 2025-02-03 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [normalization, bcnf, 3nf, lossless-decomposition, dependency-preserving, armstrong-axioms, closure, problem-solving]
pin: false
math: true
mermaid: false
---

이번 포스트에서는 관계형 데이터베이스 설계의 핵심인 정규화(Normalization) 문제들을 다룹니다. Part 1에서는 BCNF/3NF 분해, Armstrong's Axioms 증명, 정보 중복 문제를 중심으로 학습합니다.

## 📚 참고 자료

### Exercise 7.1의 스키마

이번 문제들에서 자주 참조되는 기본 스키마입니다:

```
R = (A, B, C, D, E)

F = {
    A → BC
    CD → E
    B → D
    E → A
}
```

**후보키**: A, BC, CD, E (Exercise 7.6에서 계산)

---

## 문제 7.21 - BCNF 무손실 분해

**문제**: Exercise 7.1의 스키마 R을 BCNF로 무손실 분해하시오.

**답변**:

### BCNF 정의

**Boyce-Codd Normal Form (BCNF)**:
- 모든 비자명한 함수적 종속성 α → β에 대해
- α가 슈퍼키여야 함

### BCNF 위반 검사

**주어진 FD 검사**:

1. **A → BC**: A는 후보키 → ✅ BCNF 만족
2. **CD → E**: CD는 후보키 → ✅ BCNF 만족
3. **B → D**: B는 후보키? B⁺ = {B, D} ≠ R → ❌ **BCNF 위반!**
4. **E → A**: E는 후보키 → ✅ BCNF 만족

**결론**: B → D 때문에 R이 BCNF가 아님

### BCNF 분해 알고리즘

**B → D로 분해**:

```
1단계: B → D로 분해
   R₁ = B⁺ = {B, D}
   R₂ = R - (B⁺ - B) = {A, B, C, E}

2단계: R₁ 검사
   R₁ = (B, D)
   F₁ = {B → D}
   B는 R₁의 슈퍼키 → ✅ BCNF

3단계: R₂ 검사
   R₂ = (A, B, C, E)
   F₂ = {A → BC, E → A}

   - A → BC: A⁺ = {A, B, C} (R₂에서)
     A는 R₂의 후보키? A⁺ = {A, B, C} ≠ R₂ → ❌

   실제로 A⁺을 R₂의 속성만으로 계산:
   A → B (A → BC에서)
   A → C (A → BC에서)
   A⁺ = {A, B, C} ≠ {A, B, C, E}

   하지만 E → A이므로 순환적:
   - A⁺ = {A, B, C}
   - E⁺ = {E, A, B, C}

   E가 R₂의 슈퍼키이지만 A는 아님 → 분해 계속

4단계: A → BC로 R₂ 분해
   R₃ = A⁺ = {A, B, C}
   R₄ = R₂ - (A⁺ - A) = {A, E}

5단계: R₃ 검사
   R₃ = (A, B, C)
   F₃ = {A → B, A → C}
   A는 R₃의 슈퍼키 → ✅ BCNF

6단계: R₄ 검사
   R₄ = (A, E)
   F₄ = {E → A, A → E} (순환적)
   둘 다 슈퍼키 → ✅ BCNF
```

### 최종 BCNF 분해

**Result**:
```
R₁ = (B, D)    with {B → D}
R₂ = (A, B, C) with {A → B, A → C}
R₃ = (A, E)    with {E → A, A → E}
```

**검증**:
- ✅ 모든 릴레이션이 BCNF
- ✅ 무손실 분해 (분해 과정에서 보장)

**SQL 구현**:
```sql
CREATE TABLE bd_relation (
    B VARCHAR(50),
    D VARCHAR(50),
    PRIMARY KEY (B)
);

CREATE TABLE abc_relation (
    A VARCHAR(50),
    B VARCHAR(50),
    C VARCHAR(50),
    PRIMARY KEY (A),
    FOREIGN KEY (B) REFERENCES bd_relation(B)
);

CREATE TABLE ae_relation (
    A VARCHAR(50),
    E VARCHAR(50),
    PRIMARY KEY (E),
    FOREIGN KEY (A) REFERENCES abc_relation(A)
);
```

---

## 문제 7.22 - 3NF 종속성 보존 분해

**문제**: Exercise 7.1의 스키마 R을 3NF로 무손실이고 종속성을 보존하는 분해를 하시오.

**답변**:

### 3NF 정의

**Third Normal Form (3NF)**:
- 모든 비자명한 함수적 종속성 α → β에 대해
- α가 슈퍼키이거나
- β가 후보키의 일부여야 함

### 3NF 분해 알고리즘

**Step 1**: 정준 커버 계산

Exercise 7.7에서 Fc = F (이미 정준 커버):
```
Fc = {
    A → BC
    CD → E
    B → D
    E → A
}
```

**Step 2**: 각 FD에 대해 릴레이션 생성

```
R₁ = (A, B, C)  for A → BC
R₂ = (C, D, E)  for CD → E
R₃ = (B, D)     for B → D
R₄ = (E, A)     for E → A
```

**Step 3**: 후보키가 포함된 릴레이션이 있는지 확인

**후보키**: A, BC, CD, E

- A는 R₁, R₄에 포함
- BC는 R₁과 R₃에 분산 (B∈R₃, C∈R₁)
- CD는 R₂에 포함 ✅
- E는 R₄에 포함

CD가 완전히 포함되어 있으므로 추가 릴레이션 불필요

**Step 4**: 중복 릴레이션 제거

모든 릴레이션이 필요함

### 최종 3NF 분해

**Result**:
```
R₁ = (A, B, C)  with {A → B, A → C}
R₂ = (C, D, E)  with {CD → E}
R₃ = (B, D)     with {B → D}
R₄ = (E, A)     with {E → A}
```

### 검증

**무손실 분해**:
- 알고리즘이 보장 ✅

**종속성 보존**:
- A → BC: R₁에서 검증 가능 ✅
- CD → E: R₂에서 검증 가능 ✅
- B → D: R₃에서 검증 가능 ✅
- E → A: R₄에서 검증 가능 ✅

**3NF 만족**:
모든 릴레이션에서 왼쪽이 슈퍼키 ✅

**BCNF와 비교**:
- 3NF는 4개 릴레이션
- BCNF는 3개 릴레이션
- 3NF가 종속성 보존하지만 더 많은 릴레이션

---

## 문제 7.23 - 정보 중복과 표현 불가능 문제

**문제**: "정보 중복"과 "정보 표현 불가능"의 의미를 설명하고, 왜 이들이 나쁜 데이터베이스 설계를 나타내는지 설명하시오.

**답변**:

### 정보 중복 (Repetition of Information)

**정의**: 같은 정보가 여러 튜플에 반복적으로 저장되는 현상

**예시**:
```sql
-- 비정규화 테이블
instructor_dept(ID, name, salary, dept_name, building, budget)

-- 샘플 데이터
| ID    | name   | salary | dept_name | building | budget  |
|-------|--------|--------|-----------|----------|---------|
| 10101 | Kim    | 80000  | CS        | Taylor   | 100000  |
| 12121 | Lee    | 90000  | CS        | Taylor   | 100000  |  <- 중복!
| 15151 | Park   | 70000  | CS        | Taylor   | 100000  |  <- 중복!
| 22222 | Choi   | 85000  | Physics   | Watson   | 70000   |
```

**문제점**:

1. **저장 공간 낭비**
   - CS 학과 정보(building, budget)가 3번 반복
   - 학과당 교수 수에 비례하여 중복 증가

2. **갱신 이상 (Update Anomaly)**
   ```sql
   -- CS 학과 예산 변경
   UPDATE instructor_dept
   SET budget = 120000
   WHERE dept_name = 'CS';

   -- 문제: 3개 튜플을 모두 갱신해야 함
   -- 일부만 갱신하면 불일치 발생!
   ```

3. **삽입 이상 (Insertion Anomaly)**
   ```sql
   -- 새 학과 정보만 저장하려면?
   INSERT INTO instructor_dept
   VALUES (NULL, NULL, NULL, 'Biology', 'Watson', 90000);

   -- 문제: 교수 없이 학과만 저장 불가능
   -- NULL 값 필요 → 데이터 무결성 위반
   ```

4. **삭제 이상 (Deletion Anomaly)**
   ```sql
   -- 마지막 Physics 교수 삭제
   DELETE FROM instructor_dept
   WHERE ID = 22222;

   -- 문제: Physics 학과 정보도 함께 삭제됨!
   -- 의도하지 않은 정보 손실
   ```

**해결책 - 정규화**:
```sql
-- 정규화된 설계
instructor(ID, name, salary, dept_name)
department(dept_name, building, budget)

-- 장점:
-- 1. 학과 정보 1번만 저장
-- 2. 갱신 이상 해결
-- 3. 삽입/삭제 이상 해결
```

### 정보 표현 불가능 (Inability to Represent Information)

**정의**: 특정 정보를 단독으로 표현할 수 없는 현상

**예시 1 - 삽입 이상**:
```sql
-- 학생이 없는 새 과목 개설
course(course_id, title, dept_name, credits)
student_course(student_id, course_id, semester, year, grade)

-- 문제: student_course 테이블에 과목만 저장 불가능
-- student_id가 기본키의 일부이므로 NULL 불가
```

**예시 2 - NULL 강제**:
```sql
-- 비정규화 설계
employee_project(emp_id, emp_name, project_id, project_name, role)

-- 프로젝트 없는 신입 사원
INSERT INTO employee_project
VALUES (1001, 'Kim', NULL, NULL, NULL);

-- 문제: 불필요한 NULL 값 생성
-- 데이터 의미 모호
```

**해결책**:
```sql
-- 독립적인 엔티티 분리
course(course_id, title, dept_name, credits)
enrollment(student_id, course_id, semester, year, grade)

-- 장점:
-- 1. 수강 없이도 과목 등록 가능
-- 2. 학생 없이도 과목 정보 저장
-- 3. NULL 값 불필요
```

### 왜 나쁜 설계인가?

**1. 데이터 무결성 위협**
- 중복 데이터의 불일치 가능성
- 참조 무결성 유지 어려움
- 제약 조건 강제 복잡

**2. 유지보수 비용 증가**
- 갱신 시 여러 곳 수정 필요
- 일관성 검사 오버헤드
- 버그 발생 가능성 증가

**3. 성능 문제**
- 불필요한 저장 공간
- 인덱스 크기 증가
- 조인 비용 상승 (역설적이지만 중복이 오히려 비효율적)

**4. 비즈니스 로직 복잡도**
- 애플리케이션 레벨 검증 필요
- 트랜잭션 관리 복잡
- 에러 처리 어려움

**5. 확장성 저하**
- 새로운 요구사항 대응 어려움
- 스키마 변경 시 영향 범위 큼
- 데이터 마이그레이션 복잡

### 실무 교훈

**정규화의 목적**:
```
정보 중복 제거 → 갱신 이상 방지
정보 독립성 확보 → 표현 불가능 문제 해결
```

**균형점 찾기**:
- 과도한 정규화 → 조인 비용 증가
- 과소 정규화 → 이상 현상 발생
- 성능과 무결성의 트레이드오프

---

## 문제 7.24 - 자명한 함수적 종속성

**문제**: 왜 특정 함수적 종속성들을 "자명한(trivial)" 함수적 종속성이라 부르는가?

**답변**:

### 자명한 함수적 종속성 정의

**정의**: α → β가 자명한 FD ⇔ β ⊆ α

**의미**: 오른쪽이 왼쪽의 부분집합

### 왜 "자명한(Trivial)"인가?

**1. 항상 참이기 때문**

모든 관계 인스턴스에서 자동으로 만족:

```sql
-- 예시 테이블
student(student_id, name, dept_name)

| student_id | name | dept_name |
|------------|------|-----------|
| 1001       | Kim  | CS        |
| 1002       | Lee  | Physics   |
| 1003       | Park | CS        |
```

**자명한 FD**:
- `(student_id, name) → student_id` ✅ 항상 참
- `(student_id, name) → name` ✅ 항상 참
- `(student_id, name, dept_name) → name` ✅ 항상 참

**이유**: 같은 (student_id, name) 값을 가진 튜플들은 당연히 같은 student_id와 name을 가짐!

**2. 정보를 제공하지 않기 때문**

```
(student_id, name) → student_id
```

**의미 분석**:
- "student_id와 name이 같으면 student_id가 같다"
- 당연한 동어반복(tautology)
- 데이터베이스 제약으로서 의미 없음

**3. 수학적으로 자명하기 때문**

**집합론적 관점**:
```
α → β가 자명 ⇔ β ⊆ α

증명:
  β ⊆ α이면
  t₁[α] = t₂[α]일 때
  자동으로 t₁[β] = t₂[β] (β가 α의 부분이므로)
```

### 자명한 FD의 예시

**완전 자명 (Completely Trivial)**:
```
A → A
AB → A
AB → B
ABC → A
ABC → AB
```

**부분 자명 (Partially Trivial)**:
```
AB → BC (C는 새로운 정보)
  = AB → B (자명) + AB → C (비자명)
```

### 비자명한 FD와 비교

**비자명한 FD**: β ⊈ α

```sql
student_id → name         -- 비자명!
student_id → dept_name    -- 비자명!
(course_id, semester) → instructor_id  -- 비자명!
```

**특징**:
- 실제 데이터 제약을 표현
- 데이터베이스 설계에 의미 있음
- 후보키 결정에 중요
- 정규화에 사용

### 왜 구별하는가?

**1. 정규화 판단**

BCNF 정의:
```
모든 "비자명한" FD α → β에 대해
α가 슈퍼키여야 함
```

자명한 FD는 정규화 고려 대상이 아님!

**2. 정준 커버 계산**

```
F = {
    A → A,        -- 자명 → 제거
    A → B,        -- 비자명 → 유지
    AB → A,       -- 자명 → 제거
    AB → C        -- 비자명 → 유지
}

Fc = {
    A → B,
    AB → C
}
```

**3. 속성 폐포 계산**

자명한 FD는 명시하지 않아도 암묵적으로 포함:
```
A⁺를 계산할 때
A → A는 명시하지 않아도
A ∈ A⁺는 자명
```

**4. Armstrong's Axioms**

재귀성 공리가 모든 자명한 FD 생성:
```
Reflexivity: β ⊆ α이면 α → β

→ 모든 자명한 FD는 재귀성으로 유도 가능
```

### 실무적 의미

**데이터베이스 설계 시**:

```sql
-- 자명한 제약 조건 불필요
-- ❌ 나쁜 예
CHECK (student_id = student_id)  -- 무의미!

-- ✅ 좋은 예
CHECK (age >= 18)                -- 의미 있는 제약!
```

**SQL에서**:
```sql
-- 자명한 FD는 UNIQUE 제약 불필요
-- student_id가 PK라면
-- (student_id, name) → student_id는 자동 만족
```

### 핵심 정리

**"자명한(Trivial)"의 의미**:
1. **논리적**: 항상 참 (동어반복)
2. **정보적**: 새로운 정보 없음
3. **실용적**: 설계 제약으로 의미 없음
4. **수학적**: 재귀성 공리로 자동 생성

**실무 원칙**:
- 자명한 FD 무시
- 비자명한 FD에 집중
- 정규화/최적화에 활용

---

## 문제 7.25 - Armstrong's Axioms 건전성 증명

**문제**: 함수적 종속성의 정의를 사용하여 Armstrong's Axioms(재귀성, 증대성, 이행성)가 건전함을 증명하시오.

**답변**:

### 함수적 종속성 정의

**정의**: α → β가 성립 ⇔ 모든 합법적 인스턴스 r에 대해
```
∀t₁, t₂ ∈ r: t₁[α] = t₂[α] → t₁[β] = t₂[β]
```

**의미**: α 값이 같으면 β 값도 같음

### 1. 재귀성 (Reflexivity) 증명

**공리**: β ⊆ α이면 α → β

**증명**:

```
가정: β ⊆ α
증명할 것: α → β (즉, t₁[α] = t₂[α] → t₁[β] = t₂[β])

Let t₁, t₂ ∈ r such that t₁[α] = t₂[α]

목표: t₁[β] = t₂[β]를 보이기

증명:
1. β ⊆ α (가정)
2. t₁[α] = t₂[α] (가정)
3. β의 모든 속성은 α에 포함됨 (1번에서)
4. α의 모든 속성 값이 같으므로 (2번에서)
   β의 모든 속성 값도 같음 (3번에서)
5. ∴ t₁[β] = t₂[β] ✅
```

**예시**:
```
α = {A, B, C}
β = {A, B}     (β ⊆ α)

t₁[A,B,C] = (1, 2, 3)
t₂[A,B,C] = (1, 2, 4)

t₁[α] = t₂[α]? → (1,2,3) vs (1,2,4) → NO
하지만 만약 t₁[α] = t₂[α]였다면
자동으로 t₁[A,B] = t₂[A,B] (β ⊆ α이므로)
```

**직관**: 왼쪽이 같으면 그 부분집합도 당연히 같음

### 2. 증대성 (Augmentation) 증명

**공리**: α → β이면 γα → γβ

**증명**:

```
가정: α → β (즉, t₁[α] = t₂[α] → t₁[β] = t₂[β])
증명할 것: γα → γβ (즉, t₁[γα] = t₂[γα] → t₁[γβ] = t₂[γβ])

Let t₁, t₂ ∈ r such that t₁[γα] = t₂[γα]

증명:
1. t₁[γα] = t₂[γα] (가정)
2. γα = γ ∪ α이므로
   t₁[γ] = t₂[γ] AND t₁[α] = t₂[α] (1번에서)
3. t₁[α] = t₂[α]이고 α → β이므로
   t₁[β] = t₂[β] (가정의 FD 적용)
4. t₁[γ] = t₂[γ] (2번에서) AND t₁[β] = t₂[β] (3번에서)
5. γβ = γ ∪ β이므로
   t₁[γβ] = t₂[γβ] (4번에서)
6. ∴ γα → γβ ✅
```

**예시**:
```
α → β: {A} → {B}
γ = {C}

증명: {C,A} → {C,B}

t₁[C,A] = (c, a)
t₂[C,A] = (c, a)  (같음)

→ t₁[A] = t₂[A] = a
→ A → B이므로 t₁[B] = t₂[B]
→ t₁[C] = t₂[C] = c (이미 같음)
→ ∴ t₁[C,B] = t₂[C,B]
```

**직관**: 양쪽에 같은 속성을 추가해도 FD 유지

### 3. 이행성 (Transitivity) 증명

**공리**: α → β이고 β → γ이면 α → γ

**증명**:

```
가정:
  α → β (즉, t₁[α] = t₂[α] → t₁[β] = t₂[β])
  β → γ (즉, t₁[β] = t₂[β] → t₁[γ] = t₂[γ])
증명할 것: α → γ (즉, t₁[α] = t₂[α] → t₁[γ] = t₂[γ])

Let t₁, t₂ ∈ r such that t₁[α] = t₂[α]

증명:
1. t₁[α] = t₂[α] (가정)
2. α → β이므로
   t₁[β] = t₂[β] (1번과 첫 번째 가정 FD 적용)
3. β → γ이므로
   t₁[γ] = t₂[γ] (2번과 두 번째 가정 FD 적용)
4. ∴ t₁[α] = t₂[α] → t₁[γ] = t₂[γ]
5. ∴ α → γ ✅
```

**예시**:
```
α → β: {A} → {B}
β → γ: {B} → {C}

증명: {A} → {C}

| A | B | C |
|---|---|---|
| 1 | 2 | 3 |
| 1 | 2 | 3 |  ← A 같으면 B 같음 (A → B)
| 2 | 4 | 5 |

B 같으면 C 같음 (B → C)
→ A 같으면 최종적으로 C 같음
```

**직관**: 함수적 종속성의 연쇄 (A→B→C이면 A→C)

### 건전성(Soundness)의 의미

**건전하다** = 추론 규칙이 틀린 결론을 내지 않음

```
F ⊨ f  →  F ⊢ f

의미:
  F로부터 의미론적으로 유도되는 모든 FD는
  Armstrong's Axioms로도 유도 가능
```

**완전성(Completeness)**:
- Armstrong's Axioms는 건전하면서 완전함
- F⁺의 모든 FD를 유도 가능

### 실무적 의미

**1. 신뢰성**:
```sql
-- A → B이고 B → C이면
-- A → C를 안전하게 가정 가능
CREATE INDEX idx_a ON table(a);  -- A로 C 검색 가능
```

**2. 최적화**:
```sql
-- 쿼리 최적화에 활용
SELECT c FROM table WHERE a = 1;
-- A → B → C를 알면 조인 없이 직접 접근 가능
```

**3. 설계 검증**:
```
후보키 찾기, 정규화 판단 등에서
Armstrong's Axioms로 유도한 FD가
실제 데이터베이스에서도 성립함을 보장
```

---

## 문제 7.26 - 잘못된 추론 규칙 반례

**문제**: 다음 함수적 종속성 규칙이 건전하지 않음을 증명하시오:
"α → β이고 γ → β이면 α → γ"

즉, α → β와 γ → β를 만족하지만 α → γ를 만족하지 않는 관계 r을 제시하시오.

**답변**:

### 규칙 분석

**제안된 규칙**: α → β ∧ γ → β → α → γ

**직관적 문제**:
- β는 "결과"
- α와 γ는 모두 β를 결정
- 하지만 α와 γ 사이에 관계 없을 수 있음!

**비유**:
```
"키가 180cm이면 성인이다"  (키 → 성인)
"나이가 20세이면 성인이다" (나이 → 성인)
→ "키가 180cm이면 나이가 20세다"? ❌ 말이 안 됨!
```

### 반례 구성

**스키마**: R(A, B, C)

**함수적 종속성**:
- A → B ✅
- C → B ✅

**관계 인스턴스 r**:

| A  | B  | C  |
|----|----|----|
| a₁ | b₁ | c₁ |
| a₂ | b₁ | c₂ |

### 검증

**1. A → B 성립?**

- Row 1: A = a₁ → B = b₁
- Row 2: A = a₂ → B = b₁

A 값이 다르므로 검증할 튜플 쌍 없음
하지만 합법적 인스턴스 (조건 위반 없음) ✅

더 명확한 예:
| A  | B  | C  |
|----|----|----|
| a₁ | b₁ | c₁ |
| a₁ | b₁ | c₂ |

A = a₁인 튜플들: B 모두 b₁ → ✅ A → B 만족

**2. C → B 성립?**

| A  | B  | C  |
|----|----|----|
| a₁ | b₁ | c₁ |
| a₁ | b₁ | c₂ |

- C = c₁인 튜플들: B = b₁
- C = c₂인 튜플들: B = b₁

모두 만족 → ✅ C → B 성립

**3. A → C 성립?**

| A  | B  | C  |
|----|----|----|
| a₁ | b₁ | c₁ |
| a₁ | b₁ | c₂ |  ← 같은 A, 다른 C!

A = a₁인 튜플들의 C 값: c₁, c₂ (다름!)
→ ❌ **A → C 불성립!**

### 결론

**증명 완료**:
```
r ⊨ A → B  ✅
r ⊨ C → B  ✅
r ⊭ A → C  ❌

∴ "α → β ∧ γ → β → α → γ" 규칙은 건전하지 않음
```

### 실세계 예시

**학생 데이터베이스**:
```sql
student(student_id, gpa, dept_name)

-- 함수적 종속성
student_id → gpa      -- 학번으로 학점 결정
dept_name → gpa       -- 학과 평균 학점 (가정)

-- 이것이 의미하는가?
student_id → dept_name?  ❌ 아님!

-- 반례
| student_id | gpa | dept_name |
|------------|-----|-----------|
| 1001       | 3.5 | CS        |
| 1002       | 3.5 | Physics   |

-- 같은 GPA, 다른 학과!
```

### 올바른 추론

**역방향은 성립할 수 있음**:
```
β → α ∧ β → γ → α ≡ γ?

반례: 여전히 성립 안 함
β가 α와 γ를 모두 결정해도
α와 γ가 동일하다는 보장 없음
```

**성립하는 경우**:
```
α → β ∧ γ → β ∧ β → γ → α → γ  ✅

증명:
  α → β (가정)
  β → γ (가정)
  α → γ (이행성)
```

---

## 문제 7.27 - Decomposition Rule 건전성 증명

**문제**: Armstrong's Axioms를 사용하여 분해 규칙(Decomposition Rule)의 건전성을 증명하시오.

**Decomposition Rule**: α → βγ이면 α → β이고 α → γ

**답변**:

### 증명

**주어진 것**: α → βγ

**증명할 것**:
1. α → β
2. α → γ

**방법 1 - 재귀성과 이행성 사용**:

```
Step 1: β ⊆ βγ (명백함)

Step 2: 재귀성 적용
  βγ → β  (β ⊆ βγ이므로)

Step 3: 이행성 적용
  α → βγ  (주어짐)
  βγ → β  (Step 2)
  ∴ α → β  ✅

Step 4: γ ⊆ βγ (명백함)

Step 5: 재귀성 적용
  βγ → γ  (γ ⊆ βγ이므로)

Step 6: 이행성 적용
  α → βγ  (주어짐)
  βγ → γ  (Step 5)
  ∴ α → γ  ✅
```

**방법 2 - 정의를 이용한 직접 증명**:

```
가정: α → βγ
  즉, t₁[α] = t₂[α] → t₁[βγ] = t₂[βγ]

증명 α → β:
  1. t₁[α] = t₂[α] (가정)
  2. t₁[βγ] = t₂[βγ] (α → βγ 적용)
  3. βγ = β ∪ γ이므로
     t₁[β] = t₂[β] AND t₁[γ] = t₂[γ]
  4. 특히 t₁[β] = t₂[β]
  5. ∴ α → β  ✅

증명 α → γ:
  3번 단계에서 t₁[γ] = t₂[γ]도 얻음
  ∴ α → γ  ✅
```

### 실용적 의미

**정준 커버 계산**:
```
F = { A → BCD }

분해:
  A → B
  A → C
  A → D

→ 오른쪽을 단일 속성으로 만들기
```

**예시**:
```sql
-- 원래 FD
employee_id → (name, dept, salary)

-- 분해
employee_id → name
employee_id → dept
employee_id → salary

-- SQL 구현
CREATE TABLE employee (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    dept VARCHAR(50),
    salary DECIMAL(10,2)
);

-- employee_id가 PK이므로
-- 모든 속성을 결정 (분해된 FD 모두 만족)
```

### Union Rule과의 관계

**Union Rule**: α → β ∧ α → γ → α → βγ

**Decomposition Rule**: α → βγ → α → β ∧ α → γ

**관계**: 서로 역(converse)

```
α → βγ  ⟺  α → β ∧ α → γ

의미:
  여러 속성을 한 번에 결정하는 것과
  각각 따로 결정하는 것은 동치
```

---

## 문제 7.28 - 속성 폐포 계산

**문제**: Exercise 7.6의 함수적 종속성을 사용하여 B⁺를 계산하시오.

```
F = {
    A → BC
    CD → E
    B → D
    E → A
}
```

**답변**:

### 속성 폐포 알고리즘

```
result := B
repeat
    for each α → β in F do
        if α ⊆ result then
            result := result ∪ β
until (result 변화 없음)
return result
```

### B⁺ 계산 과정

**초기화**:
```
result = {B}
```

**반복 1**:

1. **A → BC 검사**
   - A ⊆ result? {A} ⊆ {B}? → ❌ 적용 불가

2. **CD → E 검사**
   - CD ⊆ result? {C,D} ⊆ {B}? → ❌ 적용 불가

3. **B → D 검사**
   - B ⊆ result? {B} ⊆ {B}? → ✅ 적용!
   - result = result ∪ {D} = {B, D}

4. **E → A 검사**
   - E ⊆ result? {E} ⊆ {B,D}? → ❌ 적용 불가

**반복 1 결과**: result = {B, D} (변화 있음 → 계속)

**반복 2**:

1. **A → BC 검사**
   - A ⊆ {B,D}? → ❌

2. **CD → E 검사**
   - CD ⊆ {B,D}? → ❌ (C 없음)

3. **B → D 검사**
   - B ⊆ {B,D}? → ✅
   - result = {B,D} ∪ {D} = {B,D} (변화 없음)

4. **E → A 검사**
   - E ⊆ {B,D}? → ❌

**반복 2 결과**: result = {B, D} (변화 없음 → 종료)

### 최종 결과

**B⁺ = {B, D}**

### 검증 및 분석

**B는 후보키인가?**
```
B⁺ = {B, D} ≠ R = {A, B, C, D, E}
→ ❌ B는 후보키가 아님
```

**B로 결정할 수 있는 속성**:
- B → B (자명)
- B → D (주어진 FD)
- B → BD (합집합)

**B로 결정할 수 없는 속성**:
- A, C, E

**의미**:
- B 값을 알아도 A, C, E는 결정 불가
- 이것이 B → D가 BCNF를 위반하는 이유
  (B가 슈퍼키가 아니므로)

### 다른 속성들의 폐포 비교

**Exercise 7.6 결과 요약**:
```
A⁺ = {A, B, C, D, E} = R  → 후보키 ✅
B⁺ = {B, D}              → 후보키 아님
C⁺ = {C}                 → 후보키 아님
D⁺ = {D}                 → 후보키 아님
E⁺ = {A, B, C, D, E} = R  → 후보키 ✅

BC⁺ = {A, B, C, D, E} = R → 후보키 ✅
CD⁺ = {A, B, C, D, E} = R → 후보키 ✅
```

**관찰**:
- B 단독으로는 약함
- C와 결합하면 후보키 (BC)
- D와 결합해도 여전히 부족

### 실무 활용

**1. 인덱스 설계**:
```sql
-- B로 D를 직접 조회 가능
CREATE INDEX idx_b ON table(B);
-- D 검색 최적화

-- 하지만 A, C, E 조회는 조인 필요
```

**2. 정규화 판단**:
```
B → D이지만 B가 슈퍼키 아님
→ BCNF 위반
→ 분해 필요: (B, D) 분리
```

**3. 쿼리 최적화**:
```sql
-- B로 D를 유도 가능
SELECT d FROM table WHERE b = 'value';
-- 효율적

-- B로 A, C, E 유도 불가능
SELECT a, c, e FROM table WHERE b = 'value';
-- 조인 또는 추가 조건 필요
```

---

## 💡 핵심 정리

### BCNF vs 3NF 분해

| 특성 | BCNF | 3NF |
|------|------|-----|
| **정의** | 모든 결정자가 후보키 | 결정자가 후보키 또는 결정되는 속성이 후보키의 일부 |
| **무손실성** | ✅ 보장 | ✅ 보장 |
| **종속성 보존** | ❌ 보장 안 됨 | ✅ 보장 |
| **릴레이션 수** | 적음 | 많음 |
| **선택 기준** | 성능 우선 | 무결성 우선 |

### 정보 중복 문제

**원인**:
- 비정규화된 스키마
- 부적절한 함수적 종속성

**결과**:
- 갱신 이상
- 삽입 이상
- 삭제 이상

**해결책**:
- 정규화 (BCNF/3NF)
- 적절한 분해

### Armstrong's Axioms

**기본 규칙**:
1. **재귀성**: β ⊆ α → α → β
2. **증대성**: α → β → γα → γβ
3. **이행성**: α → β, β → γ → α → γ

**유도 규칙**:
- **합집합**: α → β, α → γ → α → βγ
- **분해**: α → βγ → α → β, α → γ

**특징**:
- 건전하고 완전함
- 모든 FD 유도 가능

### 속성 폐포 활용

**용도**:
1. 후보키 찾기: α⁺ = R?
2. FD 검증: β ⊆ α⁺?
3. 정규화 판단
4. 인덱스 설계

**알고리즘**: 반복적 확장

---

Part 2에서는 3NF 분해 알고리즘, BCNF 분해 알고리즘, 종속성 보존 판단을 중심으로 다룹니다!
