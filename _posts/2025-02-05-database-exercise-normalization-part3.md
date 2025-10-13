---
title: "[데이터베이스] 문제 해설 - 정규화 Part 3 (Exercises 7.37-7.44)"
date: 2025-02-05 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [normalization, bcnf, 3nf, 4nf, 2nf, multivalued-dependency, temporal-join, design-goals, sql-constraints, problem-solving]
pin: false
math: true
mermaid: false
---

이번 포스트에서는 정규화의 고급 주제들을 다룹니다. Part 3에서는 설계 목표, 정규형 선택, 다치 종속성(MVD), 4NF, 시간적 조인, SQL 제약 조건 구현을 중심으로 학습합니다.

## 📚 핵심 개념 복습

### 정규형 계층

```
1NF ⊆ 2NF ⊆ 3NF ⊆ BCNF ⊆ 4NF

낮음 ← 정규화 수준 → 높음
많음 ← 중복/이상 → 적음
적음 ← 릴레이션 수 → 많음
```

### 다치 종속성 (MVD)

**표기**: α →→ β
**의미**: α 값이 같으면 β 값의 집합이 다른 속성과 무관하게 결정됨

---

## 문제 7.33 - 3NF 분해 상세 수행

**문제**: R = (A, B, C, D, E, G)와 다음 F에 대해 3NF 분해 알고리즘을 수행하시오:

```
F = {
    A → BCD
    BC → DE
    B → D
    D → A
}
```

작업:
a. 모든 후보키 나열
b. 정준 커버 Fc 계산 (단계별 설명)
c. 알고리즘의 나머지 단계
d. 최종 분해

**답변**:

### (a) 후보키 찾기

**전략**: 모든 단일 속성과 조합 시도

**G 분석**:
- G는 어떤 FD의 오른쪽에도 없음
- → G는 모든 후보키에 포함되어야 함

**AG⁺ 계산**:
```
초기: {A, G}

A → BCD: {A, G, B, C, D}
BC → DE: 이미 B, C, D, E 있음 → {A, G, B, C, D, E}
B → D: 이미
D → A: 이미

AG⁺ = {A, B, C, D, E, G} = R ✅

AG는 후보키!
```

**다른 후보키 확인**:

**BG⁺ 계산**:
```
초기: {B, G}

B → D: {B, G, D}
D → A: {B, G, D, A}
A → BCD: {B, G, D, A, C} (이미 있음)
BC → DE: {B, G, D, A, C, E}

BG⁺ = R ✅

BG는 후보키!
```

**DG⁺ 계산**:
```
초기: {D, G}

D → A: {D, G, A}
A → BCD: {D, G, A, B, C}
BC → DE: {D, G, A, B, C, E}

DG⁺ = R ✅

DG는 후보키!
```

**CG⁺ 계산**:
```
초기: {C, G}

어떤 FD도 C 단독으로 시작 안 함
CG⁺ = {C, G} ≠ R ❌
```

**후보키**: **AG, BG, DG**

### (b) 정준 커버 계산

**Step 1**: 오른쪽을 단일 속성으로

```
F = {
    A → BCD
    BC → DE
    B → D
    D → A
}

분해:
A → B
A → C
A → D
BC → D
BC → E
B → D
D → A

F' = {
    A → B,
    A → C,
    A → D,
    BC → D,
    BC → E,
    B → D,
    D → A
}
```

**Step 2**: 왼쪽의 불필요한 속성 제거

**BC → D 검사**:
- B 제거 가능? C⁺ (F' - {BC → D})
  ```
  C⁺ = {C} ≠ D → B 필요
  ```

- C 제거 가능? B⁺ (F' - {BC → D})
  ```
  B⁺ = {B, D, A, C} (B → D, D → A, A → C)
  D ∈ B⁺ ✅ → C 불필요!
  ```

**BC → D 제거, B → D 이미 있음**

**BC → E 검사**:
- B 제거 가능? C⁺ = {C} ≠ E → B 필요
- C 제거 가능? B⁺ = {B, D, A, C} ≠ E → C 필요

**둘 다 필요**

```
F'' = {
    A → B,
    A → C,
    A → D,
    BC → E,
    B → D,
    D → A
}
```

**Step 3**: 불필요한 FD 제거

**1. A → B 필요?**
```
F''' = F'' - {A → B}

A⁺ (F''')에서:
A → C: {A, C}
A → D: {A, C, D}
D → A: 이미
BC → E: C ∈, B 없음
B → D: B 없음

A⁺ = {A, C, D}
B ∉ A⁺ → 필요 ✅
```

**2. A → C 필요?**
```
F''' = F'' - {A → C}

A⁺:
A → B: {A, B}
A → D: {A, B, D}
B → D: 이미
D → A: 이미

A⁺ = {A, B, D}
C ∉ A⁺ → 필요 ✅
```

**3. A → D 필요?**
```
F''' = F'' - {A → D}

A⁺:
A → B: {A, B}
A → C: {A, B, C}
BC → E: {A, B, C, E}
B → D: {A, B, C, E, D} ✅
D → A: 이미

A⁺ = {A, B, C, E, D}
D ∈ A⁺ → **불필요!** ❌
```

**A → D 제거**

```
F''' = {
    A → B,
    A → C,
    BC → E,
    B → D,
    D → A
}
```

**4. BC → E 필요?**
```
F'''' = F''' - {BC → E}

BC⁺:
B → D: {B, C, D}
D → A: {B, C, D, A}
A → B: 이미
A → C: 이미

BC⁺ = {B, C, D, A}
E ∉ BC⁺ → 필요 ✅
```

**5. B → D 필요?**
```
명백히 필요 (다른 경로로 D 얻을 수 없음) ✅
```

**6. D → A 필요?**
```
명백히 필요 ✅
```

### 정준 커버 결과

**Fc**:
```
Fc = {
    A → B,
    A → C,
    BC → E,
    B → D,
    D → A
}
```

### (c) 3NF 분해 알고리즘 계속

**Step 4**: 각 FD에 대해 릴레이션 생성

```
R₁ = (A, B)     for A → B
R₂ = (A, C)     for A → C
R₃ = (B, C, E)  for BC → E
R₄ = (B, D)     for B → D
R₅ = (D, A)     for D → A
```

**Step 5**: 후보키 포함 릴레이션 확인

**후보키**: AG, BG, DG

**확인**:
- AG: A ∈ R₁, R₂, R₅; G ∉ 어디에도
- BG: B ∈ R₁, R₃, R₄; G ∉ 어디에도
- DG: D ∈ R₄, R₅; G ∉ 어디에도

**없음! 후보키 릴레이션 추가**:
```
R₆ = (A, G)  -- 또는 (B, G) 또는 (D, G)
```

**Step 6**: 중복 릴레이션 제거

**중복 확인**:
- R₁ = (A, B): 다른 릴레이션에 포함? ❌
- R₂ = (A, C): 다른 릴레이션에 포함? ❌
- R₃ = (B, C, E): 다른 릴레이션에 포함? ❌
- R₄ = (B, D): 다른 릴레이션에 포함? ❌
- R₅ = (D, A): 다른 릴레이션에 포함? ❌
- R₆ = (A, G): 다른 릴레이션에 포함? ❌

모두 필요함

### (d) 최종 3NF 분해

**Result**:
```
R₁ = (A, B)
R₂ = (A, C)
R₃ = (B, C, E)
R₄ = (B, D)
R₅ = (D, A)
R₆ = (A, G)  -- candidate key
```

**SQL 구현**:
```sql
CREATE TABLE ab_relation (
    A VARCHAR(50),
    B VARCHAR(50),
    PRIMARY KEY (A)
);

CREATE TABLE ac_relation (
    A VARCHAR(50),
    C VARCHAR(50),
    PRIMARY KEY (A),
    FOREIGN KEY (A) REFERENCES ab_relation(A)
);

CREATE TABLE bce_relation (
    B VARCHAR(50),
    C VARCHAR(50),
    E VARCHAR(50),
    PRIMARY KEY (B, C)
);

CREATE TABLE bd_relation (
    B VARCHAR(50),
    D VARCHAR(50),
    PRIMARY KEY (B)
);

CREATE TABLE da_relation (
    D VARCHAR(50),
    A VARCHAR(50),
    PRIMARY KEY (D)
);

CREATE TABLE ag_relation (
    A VARCHAR(50),
    G VARCHAR(50),
    PRIMARY KEY (A, G),
    FOREIGN KEY (A) REFERENCES ab_relation(A)
);
```

---

## 문제 7.34 - 3NF 분해 (더 큰 스키마)

**문제**: R = (A, B, C, D, E, G, H)와 다음 F에 대해 3NF 분해:

```
F = {
    AB → C
    AC → B
    AD → E
    B → D
    BC → A
    E → G
}
```

**답변**:

### (a) 후보키 찾기

**H 분석**:
- H는 어떤 FD에도 없음
- → 모든 후보키에 포함

**ABH⁺ 계산**:
```
초기: {A, B, H}

AB → C: {A, B, H, C}
AC → B: 이미
AD → E: D 없음
B → D: {A, B, H, C, D}
BC → A: 이미
E → G: E 없음

AD → E 적용: {A, B, H, C, D, E}
E → G: {A, B, H, C, D, E, G}

ABH⁺ = R ✅
```

**ACH⁺ 계산**:
```
초기: {A, C, H}

AC → B: {A, C, H, B}
AB → C: 이미
B → D: {A, C, H, B, D}
AD → E: {A, C, H, B, D, E}
E → G: {A, C, H, B, D, E, G}

ACH⁺ = R ✅
```

**BCH⁺ 계산**:
```
초기: {B, C, H}

BC → A: {B, C, H, A}
이후 ACH와 동일

BCH⁺ = R ✅
```

**후보키**: **ABH, ACH, BCH**

### (b) 정준 커버

**Step 1**: 오른쪽 단일 속성 (이미 만족) ✅

**Step 2**: 왼쪽 불필요 속성 제거

**AB → C**:
- A 제거? B⁺ = {B, D} ≠ C → 필요
- B 제거? A⁺ = {A} ≠ C → 필요

**AC → B**:
- A 제거? C⁺ = {C} ≠ B → 필요
- C 제거? A⁺ = {A} ≠ B → 필요

**AD → E**:
- A 제거? D⁺ = {D} ≠ E → 필요
- D 제거? A⁺ = {A} ≠ E → 필요

**BC → A**:
- B 제거? C⁺ = {C} ≠ A → 필요
- C 제거? B⁺ = {B, D} ≠ A → 필요

모두 필요 ✅

**Step 3**: 불필요 FD 제거

각 FD가 다른 경로로 유도 불가능 → 모두 필요

**Fc = F**

### (c) 3NF 분해

**Step 4**: 각 FD에 대해 릴레이션

```
R₁ = (A, B, C)  for AB → C, AC → B, BC → A
R₂ = (A, D, E)  for AD → E
R₃ = (B, D)     for B → D
R₄ = (E, G)     for E → G
```

**최적화**: ABC는 3개 FD 모두 포함

**Step 5**: 후보키 포함 확인

후보키: ABH, ACH, BCH

H가 어디에도 없음 → 추가:
```
R₅ = (A, B, H)  -- 또는 (A, C, H) 또는 (B, C, H)
```

### (d) 최종 분해

**Result**:
```
R₁ = (A, B, C)
R₂ = (A, D, E)
R₃ = (B, D)
R₄ = (E, G)
R₅ = (A, B, H)  -- candidate key
```

---

## 문제 7.35 - BCNF 분해의 역설

**문제**: BCNF 알고리즘은 무손실 분해를 보장하지만, 알고리즘으로 생성되지 않았고 BCNF이지만 무손실이 아닌 분해의 예를 제시하시오.

**답변**:

### 스키마 구성

**원래 스키마**:
```
R = (A, B, C)

F = {
    A → B
    B → C
}
```

**분석**:
- A⁺ = {A, B, C} = R → A는 후보키
- B⁺ = {B, C} ≠ R → B는 후보키 아님
- B → C는 BCNF 위반 (B가 슈퍼키 아님)

### BCNF 알고리즘으로 올바른 분해

```
B → C로 분해:
R₁ = B⁺ = {B, C}
R₂ = R - (B⁺ - B) = {A, B}

검증:
- R₁ ∩ R₂ = {B}
- B⁺ = {B, C} = R₁ ✅
- 무손실 ✅
```

### 잘못된 BCNF 분해

**임의로 분해**:
```
R₁ = (A, B)
R₂ = (B, C)
```

**BCNF 확인**:

**R₁ = (A, B)**:
- F₁ = {A → B}
- A⁺ = {A, B} = R₁ ✅ BCNF

**R₂ = (B, C)**:
- F₂ = {B → C}
- B⁺ = {B, C} = R₂ ✅ BCNF

**둘 다 BCNF!** ✅

**무손실 확인**:
- R₁ ∩ R₂ = {B}
- B → R₁? B⁺ = {B, C} ≠ R₁ = {A, B} ❌
- B → R₂? B⁺ = {B, C} = R₂ ✅

**무손실!** ✅

**문제**: 이 예시는 실제로 무손실입니다!

### 올바른 반례

**다른 스키마**:
```
R = (A, B, C, D)

F = {
    AB → C
    C → D
}
```

**잘못된 분해**:
```
R₁ = (A, C)
R₂ = (B, D)
```

**BCNF 확인**:
- R₁: FD 없음 (AB가 분리됨) → ✅ BCNF
- R₂: FD 없음 → ✅ BCNF

**무손실 확인**:
- R₁ ∩ R₂ = ∅
- 교집합이 공집합 → ❌ **무손실 아님!**

**반례 인스턴스**:
```
r(R):
| A  | B  | C  | D  |
|----|----|----|---|
| a₁ | b₁ | c₁ | d₁ |

ΠA,C(r):
| A  | C  |
|----|-----|
| a₁ | c₁ |

ΠB,D(r):
| B  | D  |
|----|-----|
| b₁ | d₁ |

ΠA,C(r) ⋈ ΠB,D(r):
| A  | B  | C  | D  |
|----|----|----|-----|
| a₁ | b₁ | c₁ | d₁ |  ← 원본

(이 경우 우연히 같지만)

r에 튜플 추가:
| A  | B  | C  | D  |
|----|----|----|-----|
| a₁ | b₁ | c₁ | d₁ |
| a₂ | b₂ | c₂ | d₂ |

ΠA,C(r):
| A  | C  |
|----|-----|
| a₁ | c₁ |
| a₂ | c₂ |

ΠB,D(r):
| B  | D  |
|----|-----|
| b₁ | d₁ |
| b₂ | d₂ |

ΠA,C(r) ⋈ ΠB,D(r) (교집합 없으므로 카테시안 곱):
| A  | B  | C  | D  |
|----|----|----|-----|
| a₁ | b₁ | c₁ | d₁ |
| a₁ | b₂ | c₁ | d₂ |  ← 가짜!
| a₂ | b₁ | c₂ | d₁ |  ← 가짜!
| a₂ | b₂ | c₂ | d₂ |

4 튜플 ≠ 원본 2 튜플
```

**∴ BCNF이지만 무손실 아님!** ❌

### 교훈

**BCNF ≠ 무손실**

무손실이려면:
1. 교집합이 공집합이 아니어야 함
2. 교집합이 한쪽의 슈퍼키여야 함

**BCNF 알고리즘의 가치**:
- 올바른 교집합 보장
- 무손실 자동 보장
- 임의 분해는 위험!

---

## 문제 7.36 - 2-속성 스키마는 항상 BCNF

**문제**: 정확히 2개 속성으로 구성된 모든 스키마는 주어진 함수적 종속성 집합 F와 무관하게 BCNF임을 보이시오.

**답변**:

### 증명

**스키마**: R = (A, B)

**가능한 FD**:

1. **자명한 FD**:
   - A → A, B → B, AB → A, AB → B, AB → AB
   - 자명한 FD는 BCNF 판단에서 무시 ✅

2. **비자명한 FD**:
   - A → B
   - B → A
   - AB → ∅ (불가능, 오른쪽이 공집합)

### Case 1: A → B

**A⁺ 계산**:
```
초기: {A}
A → B: {A, B} = R

A⁺ = R → A는 슈퍼키 ✅
```

**BCNF 조건**: A → B에서 A가 슈퍼키 → ✅ BCNF

### Case 2: B → A

**B⁺ 계산**:
```
초기: {B}
B → A: {B, A} = R

B⁺ = R → B는 슈퍼키 ✅
```

**BCNF 조건**: B → A에서 B가 슈퍼키 → ✅ BCNF

### Case 3: A → B이고 B → A (둘 다)

- A는 슈퍼키 ✅
- B도 슈퍼키 ✅
- → BCNF ✅

### Case 4: FD 없음

FD 없으면 BCNF 위반 없음 → ✅ BCNF

### 일반화

**정리**: 2-속성 스키마 R = (X, Y)에서

**비자명한 FD α → β가 있으면**:
- α ≠ ∅ (왼쪽은 공집합 아님)
- β ≠ ∅ (오른쪽도 공집합 아님)
- β ⊈ α (비자명)

**가능성**:
- α = {X}, β = {Y} → α⁺ = {X, Y} = R ✅
- α = {Y}, β = {X} → α⁺ = {Y, X} = R ✅
- α = {X, Y} → β는 자명 (제외)

**결론**: 모든 비자명한 FD에서 왼쪽은 슈퍼키!

**∴ 2-속성 스키마는 항상 BCNF** ✅

### 3-속성에서는?

**반례**: R = (A, B, C)
```
F = { B → C }

B⁺ = {B, C} ≠ R
→ B는 슈퍼키 아님
→ ❌ BCNF 위반
```

---

## 문제 7.37 - 관계형 데이터베이스 설계의 3대 목표

**문제**: 관계형 데이터베이스의 3가지 설계 목표를 나열하고, 각각이 왜 바람직한지 설명하시오.

**답변**:

### 1. 무손실 분해 (Lossless Decomposition)

**정의**: 분해된 릴레이션들을 자연 조인하면 원래 릴레이션을 정확히 복원

**조건**: R₁ ∩ R₂ → R₁ 또는 R₁ ∩ R₂ → R₂

**왜 바람직한가**:

**데이터 무결성**:
```sql
-- 원본
employee(emp_id, name, dept, salary)

-- 잘못된 분해 (무손실 아님)
emp_info(emp_id, name)
dept_info(dept, salary)  -- 교집합 없음!

-- 조인 시 카테시안 곱 발생
-- 모든 직원과 모든 부서 급여가 매칭됨!
```

**정보 보존**:
- 분해 전후 동일한 정보
- 데이터 손실 없음
- 쿼리 결과 정확성 보장

**역연산 가능**:
- 분해 → 조인 → 원본
- 가역적 변환
- 데이터 일관성 유지

**실무 영향**:
```sql
-- 무손실 분해 예시
employee(emp_id, name, dept_id)
department(dept_id, dept_name, budget)

-- 원본 복원
SELECT e.*, d.dept_name, d.budget
FROM employee e
JOIN department d ON e.dept_id = d.dept_id;

-- 정확한 결과 보장!
```

### 2. 종속성 보존 (Dependency Preservation)

**정의**: 원래 함수적 종속성들을 분해된 각 릴레이션에서 개별적으로 검증 가능

**왜 바람직한가**:

**효율적 무결성 검증**:
```sql
-- 종속성 보존 O
-- FD: emp_id → dept
employee(emp_id, name, dept)
-- 삽입/갱신 시 이 테이블만 검사

-- 종속성 보존 X
-- FD: emp_id → dept가 여러 테이블에 분산
employee_info(emp_id, name)
emp_dept(emp_id, dept_id)
dept_info(dept_id, dept_name)
-- FD 검증을 위해 조인 필요!
```

**성능 향상**:
- 조인 없이 제약 조건 검증
- 트랜잭션 속도 향상
- 잠금 범위 최소화

**단순한 비즈니스 로직**:
```java
// 종속성 보존 O
void insertEmployee(Employee emp) {
    // 단일 테이블 검증
    validateDepartment(emp);
    employeeTable.insert(emp);
}

// 종속성 보존 X
void insertEmployee(Employee emp) {
    // 복잡한 조인 검증
    List<Table> tables = fetchRelatedTables();
    validateAcrossTables(tables, emp);
    employeeTable.insert(emp);
}
```

**실시간 검증**:
- 데이터 입력 즉시 검증
- 불일치 조기 발견
- 롤백 비용 감소

### 3. 중복 최소화 (Redundancy Minimization)

**정의**: 같은 정보가 여러 곳에 반복 저장되지 않도록

**왜 바람직한가**:

**저장 공간 절약**:
```sql
-- 중복 O (비정규화)
employee_full(emp_id, name, dept_id, dept_name, dept_budget)
-- 100명 직원, 5개 부서
-- dept_name, dept_budget이 100번 반복!

-- 중복 X (정규화)
employee(emp_id, name, dept_id)  -- 100 rows
department(dept_id, dept_name, dept_budget)  -- 5 rows
-- 부서 정보 5번만 저장!
```

**갱신 이상 방지**:

**갱신 이상 (Update Anomaly)**:
```sql
-- 부서 예산 변경
UPDATE employee_full
SET dept_budget = 150000
WHERE dept_id = 'CS';

-- 문제: CS 부서 직원 10명의 행 모두 갱신 필요
-- 일부만 갱신하면 불일치!
```

**삽입 이상 (Insertion Anomaly)**:
```sql
-- 신규 부서 생성 (직원 없이)
INSERT INTO employee_full
VALUES (NULL, NULL, 'Biology', 'Biology Dept', 80000);

-- 문제: NULL 값 필요, 의미 모호
```

**삭제 이상 (Deletion Anomaly)**:
```sql
-- 마지막 직원 퇴사
DELETE FROM employee_full
WHERE emp_id = 'E001';

-- 문제: 부서 정보도 함께 삭제됨!
```

**데이터 일관성**:
```sql
-- 정규화된 설계
UPDATE department
SET budget = 150000
WHERE dept_id = 'CS';

-- 1번만 갱신, 모든 직원에 자동 반영
-- 일관성 보장!
```

### 목표 간 트레이드오프

| 목표 | 장점 | 단점 |
|------|------|------|
| **무손실** | 필수, 협상 불가 | - |
| **종속성 보존** | 무결성 검증 쉬움 | 릴레이션 수 증가 |
| **중복 최소화** | 공간 절약, 이상 방지 | 조인 비용 증가 |

**실무 균형**:
```
BCNF: 중복 최소화 우선 (종속성 보존 희생 가능)
3NF: 종속성 보존 우선 (약간의 중복 허용)
```

---

## 문제 7.38 - 비BCNF 설계를 선택하는 이유

**문제**: 관계형 데이터베이스 설계 시 왜 비BCNF 설계를 선택할 수 있는가?

**답변**:

### 이유 1: 종속성 보존

**문제**: BCNF는 종속성 보존을 보장하지 않음

**예시**:
```
R = (A, B, C)
F = { AB → C, C → B }

BCNF 분해:
R₁ = (C, B)
R₂ = (A, C)

AB → C 보존 안 됨! (A와 B가 분리)
```

**3NF 선택**:
```
R₁ = (A, B, C)  -- AB → C 보존
R₂ = (C, B)     -- C → B 보존

종속성 보존! ✅
```

**실무 영향**:
- 제약 조건 검증 간단
- 트랜잭션 성능 향상
- 무결성 유지 용이

### 이유 2: 성능 최적화

**조인 비용**:
```sql
-- BCNF (4개 테이블)
SELECT *
FROM table1
JOIN table2 ON ...
JOIN table3 ON ...
JOIN table4 ON ...;

-- 3개 조인, 느림!

-- 3NF (2개 테이블)
SELECT *
FROM table1
JOIN table2 ON ...;

-- 1개 조인, 빠름!
```

**읽기 중심 워크로드**:
- OLAP (분석) 시스템
- 보고서 생성
- 대시보드
- → 약간의 중복 허용하고 조인 최소화

### 이유 3: 비즈니스 요구사항

**실시간 응답**:
```sql
-- BCNF: 5개 테이블 조인 (200ms)
-- 3NF: 2개 테이블 조인 (50ms)

-- 요구사항: 100ms 이내 응답
-- → 3NF 선택
```

**단순성**:
```
BCNF: 10개 테이블 → 관리 복잡
3NF: 5개 테이블 → 관리 간단

소규모 팀 → 3NF 선호
```

### 이유 4: 히스토리컬 데이터

**시점 데이터**:
```sql
-- 주문 시점의 제품 가격 저장
order_item(order_id, product_id, price, quantity)

-- FD: order_id, product_id → price
-- 하지만 product(product_id) → price도 있음
-- → BCNF 위반

-- 이유: 가격 변동 히스토리 유지
-- 정규화하면 과거 가격 손실!
```

### 이유 5: 트랜잭션 무결성

**원자성**:
```sql
-- 3NF: 단일 테이블 갱신
UPDATE customer_address
SET address = '...'
WHERE customer_id = 123;

-- BCNF: 여러 테이블 갱신
BEGIN TRANSACTION;
UPDATE customer SET ... WHERE ...;
UPDATE address SET ... WHERE ...;
UPDATE city SET ... WHERE ...;
COMMIT;

-- 복잡한 트랜잭션, 실패 위험 증가
```

### 균형잡힌 선택

**의사결정 프레임워크**:

**BCNF 선택 시**:
- ✅ OLTP (트랜잭션) 시스템
- ✅ 쓰기 중심 워크로드
- ✅ 데이터 무결성 최우선
- ✅ 저장 공간 제약

**3NF 선택 시**:
- ✅ OLAP (분석) 시스템
- ✅ 읽기 중심 워크로드
- ✅ 성능 최우선
- ✅ 종속성 보존 필요

**비정규화 선택 시**:
- ✅ 극도의 성능 요구
- ✅ 히스토리컬 데이터
- ✅ 읽기 전용 데이터
- ✅ 계산된 컬럼 (예: 총합)

---

## 문제 7.39 - 2NF 설계를 선택하는 이유

**문제**: 관계형 데이터베이스 설계의 3가지 목표를 고려할 때, 2NF이지만 더 높은 정규형이 아닌 데이터베이스 스키마를 설계할 이유가 있는가?

**답변**:

### 2NF 정의 (복습)

**2NF**: 1NF이고 모든 비기본 속성이 후보키에 완전 함수적 종속

**위반 예**:
```
R = (A, B, C, D)
후보키: AB
FD: AB → C, B → D

B → D가 부분 종속 (B는 AB의 진부분집합)
→ 2NF 위반
```

### 2NF를 선택할 이유? 거의 없음!

**3대 설계 목표 관점**:

**1. 무손실 분해**:
- 2NF도 무손실 분해 가능 ✅
- 하지만 3NF도 동일 ✅
- → 2NF만의 장점 없음

**2. 종속성 보존**:
- 2NF도 종속성 보존 가능 ✅
- 3NF도 보존 가능 ✅
- → 2NF만의 장점 없음

**3. 중복 최소화**:
- 2NF는 중복 여전히 존재 ❌
- 3NF가 더 나음 ✅
- → **2NF는 불리!**

### 2NF의 문제점

**이행적 종속 허용**:
```
R = (Student_ID, Course_ID, Instructor, Instructor_Office)
후보키: (Student_ID, Course_ID)

FD:
(Student_ID, Course_ID) → Instructor  ✅
Instructor → Instructor_Office  ✅

2NF 만족 (부분 종속 없음)
하지만 이행적 종속 존재!

문제:
- Instructor_Office 중복
- 갱신 이상 발생
```

### 2NF를 선택하는 (드문) 경우

**케이스 1: 성능 최적화 (극단적)**

```sql
-- 3NF
enrollment(student_id, course_id, instructor_id)
instructor(instructor_id, name, office)

-- 조인 필요
SELECT e.*, i.name, i.office
FROM enrollment e
JOIN instructor i ON e.instructor_id = i.instructor_id;

-- 2NF (의도적 비정규화)
enrollment(student_id, course_id, instructor_id, instructor_name, instructor_office)

-- 조인 불필요, 빠름!
SELECT * FROM enrollment;
```

**대가**:
- instructor_office 중복
- 갱신 이상 위험
- 저장 공간 증가

**정당화**:
- 읽기 빈도 >>> 쓰기 빈도
- 성능이 절대적 우선순위
- 캐싱보다 효과적

**케이스 2: 레거시 시스템 호환**

```
기존 시스템이 2NF 구조
→ 마이그레이션 비용 >>> 중복 비용
→ 유지 결정
```

**케이스 3: 단순성 (매우 작은 시스템)**

```
소규모 앱, 사용자 10명 미만
→ 정규화 오버엔지니어링
→ 2NF로 충분
```

### 실무 권장

**거의 항상 3NF 이상 선택**:

**이유**:
1. 3NF는 2NF보다 약간만 복잡
2. 중복과 이상 현상 대폭 감소
3. 유지보수 비용 절감
4. 확장성 향상

**예외 (드물게)**:
- 극단적 성능 요구
- 레거시 호환성
- 읽기 전용 데이터

**결론**: **2NF를 목표로 하는 것은 거의 없음**. 3NF가 표준!

---

## 문제 7.40 - 다치 종속성의 논리적 함의

**문제**: r(A, B, C, D)에서 A →→ BC가 논리적으로 A →→ B와 A →→ C를 함의하는가? 증명하거나 반례를 제시하시오.

**답변**:

### 다치 종속성 정의

**MVD**: α →→ β

**의미**: α 값이 같으면 β 값의 집합이 나머지 속성과 독립적

**형식적 정의**:
```
t₁, t₂ ∈ r이고 t₁[α] = t₂[α]이면
t₃, t₄ ∈ r 존재:
  t₃[α] = t₁[α]
  t₃[β] = t₁[β]
  t₃[R - α - β] = t₂[R - α - β]
  t₄[α] = t₁[α]
  t₄[β] = t₂[β]
  t₄[R - α - β] = t₁[R - α - β]
```

### 질문 분석

**주어진**: A →→ BC
**물음**: A →→ B와 A →→ C가 성립?

### 증명: 예, 함의함

**A →→ BC 의미**:
```
A 값이 같으면 BC 값 쌍의 집합이 D와 독립적
```

**A →→ B 증명**:

A →→ BC가 성립하면:
```
t₁[A] = t₂[A]일 때

A →→ BC에서:
  t₃ 존재: t₃[A] = t₁[A], t₃[BC] = t₁[BC], t₃[D] = t₂[D]
  t₄ 존재: t₄[A] = t₁[A], t₄[BC] = t₂[BC], t₄[D] = t₁[D]

A →→ B를 보이려면:
  t₁[A] = t₂[A]일 때
  t₃' 존재: t₃'[A] = t₁[A], t₃'[B] = t₁[B], t₃'[CD] = t₂[CD]

A →→ BC에서:
  t₃[A] = t₁[A]
  t₃[BC] = t₁[BC] → t₃[B] = t₁[B] ✅
  t₃[D] = t₂[D] → t₃[CD]는 t₂[CD]와 D가 같음

하지만 C가 다를 수 있음...
```

**더 간단한 증명 (직관적)**:

**보조정리**: α →→ βγ이면 α →→ β

```
증명:
A →→ BC 가정

t₁, t₂에서 t₁[A] = t₂[A]

A →→ BC에서 BC 값 집합이 D와 독립
→ B 값 집합도 D와 독립 (B ⊆ BC)
→ C 값 집합도 D와 독립 (C ⊆ BC)

∴ A →→ B ✅
∴ A →→ C ✅
```

### 반례는 없음

**정리**: **α →→ βγ → α →→ β ∧ α →→ γ** (항상 참)

### 역은?

**α →→ β ∧ α →→ γ → α →→ βγ?**

**답**: **참!** (MVD의 합집합 규칙)

**결론**: α →→ βγ ⟺ α →→ β ∧ α →→ γ

---

## 문제 7.41 - 4NF가 BCNF보다 바람직한 이유

**문제**: 왜 4NF가 BCNF보다 더 바람직한 정규형인지 설명하시오.

**답변**:

### BCNF의 한계

**BCNF**: 함수적 종속성(FD) 기반
- 모든 결정자가 슈퍼키

**문제**: 다치 종속성(MVD)으로 인한 중복 해결 못 함

### MVD가 일으키는 문제

**예시**: 교수-과목-취미

```sql
professor_info(professor_id, course, hobby)

| professor_id | course      | hobby    |
|--------------|-------------|----------|
| P001         | Databases   | Tennis   |
| P001         | Databases   | Reading  |
| P001         | Algorithms  | Tennis   |
| P001         | Algorithms  | Reading  |
```

**MVD**:
```
professor_id →→ course
professor_id →→ hobby
```

**의미**:
- 교수가 가르치는 과목 집합
- 교수의 취미 집합
- 두 집합은 독립적 (모든 조합 존재)

**BCNF 검사**:
```
후보키: (professor_id, course, hobby)

FD 없음! (MVD만 있음)
→ ✅ BCNF 만족

하지만 중복 존재! ❌
```

**중복 문제**:
```
교수 1명, 과목 N개, 취미 M개
→ N × M 튜플 필요

새 과목 추가:
→ M개 튜플 삽입 필요 (모든 취미와 조합)

새 취미 추가:
→ N개 튜플 삽입 필요 (모든 과목과 조합)
```

### 4NF 정의

**4NF**: BCNF이고 모든 비자명한 MVD α →→ β에 대해 α가 슈퍼키

### 4NF 분해

**위 예시를 4NF로**:
```
R₁ = (professor_id, course)
R₂ = (professor_id, hobby)

| professor_id | course      |
|--------------|-------------|
| P001         | Databases   |
| P001         | Algorithms  |

| professor_id | hobby    |
|--------------|----------|
| P001         | Tennis   |
| P001         | Reading  |
```

**장점**:
- 2 + 2 = 4 튜플 (원래 4 튜플과 같지만)
- 새 과목 추가: 1 튜플만 삽입
- 새 취미 추가: 1 튜플만 삽입
- 중복 제거! ✅

**더 많은 데이터에서**:
```
교수 1명, 과목 10개, 취미 5개

BCNF: 10 × 5 = 50 튜플
4NF: 10 + 5 = 15 튜플

70% 감소!
```

### 왜 4NF가 더 바람직한가

**1. 중복 최소화**:
- MVD로 인한 중복 제거
- 저장 공간 절약
- N × M → N + M

**2. 갱신 이상 방지**:

**삽입 이상**:
```
-- BCNF
INSERT INTO professor_info VALUES
('P001', 'DataMining', 'Tennis'),
('P001', 'DataMining', 'Reading');  -- 모든 취미와 조합 필요!

-- 4NF
INSERT INTO professor_course VALUES ('P001', 'DataMining');
-- 끝!
```

**삭제 이상**:
```
-- BCNF
-- Databases 과목 취소
DELETE FROM professor_info
WHERE course = 'Databases';
-- 여러 행 삭제

-- 4NF
DELETE FROM professor_course
WHERE course = 'Databases';
-- 1행만 삭제
```

**갱신 이상**:
```
-- BCNF
-- 과목명 변경
UPDATE professor_info
SET course = 'Database Systems'
WHERE course = 'Databases';
-- 여러 행 갱신 (취미 수만큼)

-- 4NF
UPDATE professor_course
SET course = 'Database Systems'
WHERE course = 'Databases';
-- 1행만 갱신
```

**3. 데이터 독립성**:
- 과목과 취미 독립 관리
- 서로 영향 없이 추가/삭제
- 비즈니스 로직 단순화

**4. 확장성**:
```
-- 새로운 독립적 속성 추가 (예: 자격증)
-- BCNF: N × M × L 폭발
-- 4NF: N + M + L 선형 증가
```

### 실무적 함의

**4NF 선택 시기**:
- 독립적인 다중값 속성
- M:N:L... 관계
- 조합 폭발 가능성

**예시 도메인**:
- 교육: 강사-과목-자격증
- 쇼핑: 제품-카테고리-태그
- 이벤트: 행사-참가자-장비

---

## 문제 7.42 - 4NF로 정규화

**문제**: 다음 스키마를 4NF로 정규화하시오:

```
books(book_id, author, publisher, keyword)
```

제약 조건:
- 한 책은 여러 저자 가능
- 한 책은 여러 출판사 가능
- 한 책은 여러 키워드 가능
- 저자, 출판사, 키워드는 서로 독립적

**답변**:

### MVD 식별

**다치 종속성**:
```
book_id →→ author
book_id →→ publisher
book_id →→ keyword
```

**의미**:
- book_id가 같으면 author 집합이 결정 (publisher, keyword와 독립)
- book_id가 같으면 publisher 집합이 결정 (author, keyword와 독립)
- book_id가 같으면 keyword 집합이 결정 (author, publisher와 독립)

### 현재 상태 (1NF/BCNF)

**예시 데이터**:
```
| book_id | author | publisher | keyword  |
|---------|--------|-----------|----------|
| B001    | Smith  | Wiley     | Database |
| B001    | Smith  | Wiley     | SQL      |
| B001    | Smith  | Pearson   | Database |
| B001    | Smith  | Pearson   | SQL      |
| B001    | Jones  | Wiley     | Database |
| B001    | Jones  | Wiley     | SQL      |
| B001    | Jones  | Pearson   | Database |
| B001    | Jones  | Pearson   | SQL      |
```

**폭발**:
- 저자 2명 × 출판사 2개 × 키워드 2개 = **8 튜플!**

**BCNF 검사**:
```
후보키: (book_id, author, publisher, keyword)
FD 없음
→ ✅ BCNF

하지만 4NF 위반! ❌
```

### 4NF 분해

**Step 1**: book_id →→ author로 분해

```
R₁ = (book_id, author)
R₂ = (book_id, publisher, keyword)
```

**R₂ 검사**:
- MVD: book_id →→ publisher, book_id →→ keyword
- 4NF 위반! 계속 분해

**Step 2**: R₂를 book_id →→ publisher로 분해

```
R₃ = (book_id, publisher)
R₄ = (book_id, keyword)
```

### 최종 4NF 분해

**Result**:
```
book_author(book_id, author)
book_publisher(book_id, publisher)
book_keyword(book_id, keyword)
```

**데이터**:
```
book_author:
| book_id | author |
|---------|--------|
| B001    | Smith  |
| B001    | Jones  |

book_publisher:
| book_id | publisher |
|---------|-----------|
| B001    | Wiley     |
| B001    | Pearson   |

book_keyword:
| book_id | keyword  |
|---------|----------|
| B001    | Database |
| B001    | SQL      |
```

**튜플 수**: 2 + 2 + 2 = **6 튜플** (원래 8 튜플)

### SQL 구현

```sql
CREATE TABLE book (
    book_id VARCHAR(10) PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    isbn VARCHAR(20)
);

CREATE TABLE book_author (
    book_id VARCHAR(10),
    author VARCHAR(100),
    PRIMARY KEY (book_id, author),
    FOREIGN KEY (book_id) REFERENCES book(book_id)
        ON DELETE CASCADE
);

CREATE TABLE book_publisher (
    book_id VARCHAR(10),
    publisher VARCHAR(100),
    PRIMARY KEY (book_id, publisher),
    FOREIGN KEY (book_id) REFERENCES book(book_id)
        ON DELETE CASCADE
);

CREATE TABLE book_keyword (
    book_id VARCHAR(10),
    keyword VARCHAR(50),
    PRIMARY KEY (book_id, keyword),
    FOREIGN KEY (book_id) REFERENCES book(book_id)
        ON DELETE CASCADE
);
```

### 검증

**4NF 만족**:
- book_author: MVD는 book_id →→ author, book_id는 슈퍼키 ✅
- book_publisher: book_id는 슈퍼키 ✅
- book_keyword: book_id는 슈퍼키 ✅

**무손실 분해**:
```sql
-- 원본 복원
SELECT ba.book_id, ba.author, bp.publisher, bk.keyword
FROM book_author ba
JOIN book_publisher bp ON ba.book_id = bp.book_id
JOIN book_keyword bk ON ba.book_id = bk.book_id;

-- 정확히 8 튜플 (원본과 동일)
```

---

## 문제 7.43 - SQL로 FD 제약 구현

**문제**: SQL이 함수적 종속성 제약을 직접 지원하지 않지만, 데이터베이스 시스템이 구체화 뷰(materialized views)에 대한 제약을 지원하고 구체화 뷰가 즉시 유지된다면 SQL로 FD를 강제할 수 있다. r(A, B, C)에서 B → C를 강제하는 방법을 설명하시오.

**답변**:

### FD B → C의 의미

**B → C**: B 값이 같으면 C 값도 같아야 함

**위반 예**:
```
| A  | B  | C  |
|----|----|----|
| a₁ | b₁ | c₁ |
| a₂ | b₁ | c₂ |  ← 위반! 같은 B, 다른 C
```

### 구체화 뷰를 이용한 검증

**핵심 아이디어**: B → C 위반 ⟺ B 값당 C 값이 2개 이상

**Step 1**: B별 C 개수를 세는 뷰 생성

```sql
CREATE MATERIALIZED VIEW fd_check AS
SELECT B, COUNT(DISTINCT C) AS c_count
FROM r
GROUP BY B;
```

**Step 2**: C 개수가 1 이하여야 함을 제약

```sql
ALTER TABLE fd_check
ADD CONSTRAINT check_fd
CHECK (c_count <= 1);
```

### 전체 구현

```sql
-- 원본 테이블
CREATE TABLE r (
    A VARCHAR(50),
    B VARCHAR(50),
    C VARCHAR(50),
    PRIMARY KEY (A)
);

-- FD 검증 뷰 (즉시 유지)
CREATE MATERIALIZED VIEW fd_b_c AS
SELECT B, COUNT(DISTINCT C) AS distinct_c_count
FROM r
GROUP BY B;

-- 제약 조건
ALTER TABLE fd_b_c
ADD CONSTRAINT enforce_fd_b_to_c
CHECK (distinct_c_count <= 1);
```

### 동작 방식

**정상 삽입**:
```sql
INSERT INTO r VALUES ('a1', 'b1', 'c1');
-- fd_b_c: {b1: 1} ✅

INSERT INTO r VALUES ('a2', 'b1', 'c1');  -- 같은 C
-- fd_b_c: {b1: 1} ✅ 통과!
```

**위반 삽입**:
```sql
INSERT INTO r VALUES ('a3', 'b1', 'c2');  -- 다른 C
-- fd_b_c 갱신 시도: {b1: 2}
-- CHECK 위반! ❌
-- 거부됨
```

### 구체화 뷰 즉시 유지의 중요성

**즉시 유지 (Immediate Maintenance)**:
```
INSERT/UPDATE/DELETE 시
→ 뷰 즉시 갱신
→ 제약 조건 즉시 검사
→ 위반 시 롤백
```

**지연 유지 (Deferred Maintenance)**:
```
INSERT/UPDATE/DELETE 시
→ 뷰 갱신 지연
→ 제약 조건 나중에 검사
→ ❌ FD 위반 허용될 수 있음!
```

### 대안 방법

**Trigger 사용** (더 일반적):

```sql
CREATE TRIGGER enforce_fd_b_c
BEFORE INSERT OR UPDATE ON r
FOR EACH ROW
DECLARE
    v_existing_c VARCHAR(50);
BEGIN
    -- 같은 B에 다른 C가 있는지 검사
    SELECT DISTINCT C INTO v_existing_c
    FROM r
    WHERE B = NEW.B
    LIMIT 1;

    IF FOUND AND v_existing_c != NEW.C THEN
        RAISE EXCEPTION 'FD violation: B -> C';
    END IF;
END;
```

### 성능 고려사항

**구체화 뷰 방식**:
- 장점: 선언적, 명확
- 단점: 뷰 유지 오버헤드, 저장 공간

**트리거 방식**:
- 장점: 효율적
- 단점: 복잡한 로직

---

## 문제 7.44 - 시간적 자연 조인

**문제**: 두 관계 r(A, B, validtime)과 s(B, C, validtime)이 주어지고, validtime은 유효 시간 구간을 나타낼 때, 시간적 자연 조인을 계산하는 SQL 쿼리를 작성하시오. && 연산자로 구간 겹침 확인, * 연산자로 구간 교집합 계산 가능.

**답변**:

### 시간적 자연 조인 정의

**일반 자연 조인**: B 값이 같은 튜플 매칭

**시간적 자연 조인**:
1. B 값이 같음
2. **유효 시간이 겹침** (&&)
3. 결과의 유효 시간 = **교집합** (*)

### SQL 구현

```sql
SELECT
    r.A,
    r.B,
    s.C,
    r.validtime * s.validtime AS validtime
FROM r
JOIN s ON r.B = s.B
WHERE r.validtime && s.validtime;
```

### 예시 데이터

**r(A, B, validtime)**:
```
| A  | B  | validtime      |
|----|----|-----------------
| a1 | b1 | [2020-01, 2020-06] |
| a2 | b1 | [2020-04, 2020-09] |
| a3 | b2 | [2020-01, 2020-12] |
```

**s(B, C, validtime)**:
```
| B  | C  | validtime      |
|----|----|-----------------
| b1 | c1 | [2020-03, 2020-08] |
| b1 | c2 | [2020-07, 2020-10] |
| b2 | c3 | [2020-06, 2020-11] |
```

### 조인 과정

**Step 1**: B = b1 매칭

```
r(a1, b1, [2020-01, 2020-06])
s(b1, c1, [2020-03, 2020-08])

겹침? [2020-01, 2020-06] && [2020-03, 2020-08] ✅
교집합: [2020-01, 2020-06] * [2020-03, 2020-08] = [2020-03, 2020-06]

결과: (a1, b1, c1, [2020-03, 2020-06])
```

```
r(a1, b1, [2020-01, 2020-06])
s(b1, c2, [2020-07, 2020-10])

겹침? [2020-01, 2020-06] && [2020-07, 2020-10] ❌
→ 제외
```

```
r(a2, b1, [2020-04, 2020-09])
s(b1, c1, [2020-03, 2020-08])

겹침? ✅
교집합: [2020-03, 2020-08]

결과: (a2, b1, c1, [2020-04, 2020-08])
```

```
r(a2, b1, [2020-04, 2020-09])
s(b1, c2, [2020-07, 2020-10])

겹침? ✅
교집합: [2020-07, 2020-09]

결과: (a2, b1, c2, [2020-07, 2020-09])
```

**Step 2**: B = b2 매칭

```
r(a3, b2, [2020-01, 2020-12])
s(b2, c3, [2020-06, 2020-11])

겹침? ✅
교집합: [2020-06, 2020-11]

결과: (a3, b2, c3, [2020-06, 2020-11])
```

### 최종 결과

```
| A  | B  | C  | validtime      |
|----|----|----|----------------|
| a1 | b1 | c1 | [2020-03, 2020-06] |
| a2 | b1 | c1 | [2020-04, 2020-08] |
| a2 | b1 | c2 | [2020-07, 2020-09] |
| a3 | b2 | c3 | [2020-06, 2020-11] |
```

### PostgreSQL 구현 (구간 타입 사용)

```sql
-- 구간 타입 사용
CREATE TABLE r (
    A VARCHAR(50),
    B VARCHAR(50),
    validtime daterange
);

CREATE TABLE s (
    B VARCHAR(50),
    C VARCHAR(50),
    validtime daterange
);

-- 시간적 조인
SELECT
    r.A,
    r.B,
    s.C,
    r.validtime * s.validtime AS validtime
FROM r
JOIN s ON r.B = s.B
WHERE r.validtime && s.validtime;
```

### 실무 활용

**사용 케이스**:
- 직원 부서 배정 × 부서 예산 (유효 기간)
- 제품 가격 × 할인 정책 (유효 기간)
- 계약 × 법규 (유효 기간)

**예시 - 직원 급여 계산**:
```sql
-- employee_dept(emp_id, dept_id, validtime)
-- dept_budget(dept_id, budget, validtime)

SELECT
    ed.emp_id,
    ed.dept_id,
    db.budget,
    ed.validtime * db.validtime AS validtime
FROM employee_dept ed
JOIN dept_budget db ON ed.dept_id = db.dept_id
WHERE ed.validtime && db.validtime;

-- 직원이 각 부서에 있던 기간 동안의 부서 예산
```

---

## 💡 핵심 정리

### 정규형 비교

| 정규형 | 조건 | 장점 | 단점 |
|-------|------|------|------|
| **2NF** | 부분 종속 제거 | 기본적 중복 감소 | 이행적 종속 허용 |
| **3NF** | 이행적 종속 제거 | 종속성 보존 | 일부 중복 가능 |
| **BCNF** | 모든 결정자가 슈퍼키 | 중복 최소화 | 종속성 보존 X |
| **4NF** | MVD 제거 | MVD 중복 제거 | 릴레이션 수 증가 |

### 설계 목표 우선순위

1. **무손실 분해** (필수)
2. **종속성 보존** (중요)
3. **중복 최소화** (바람직)

**실무 균형**:
- OLTP: BCNF/4NF 선호
- OLAP: 3NF 또는 비정규화
- 하이브리드: 상황에 따라

### 고급 개념

**다치 종속성**:
- 독립적 다중값 속성
- 조합 폭발 문제
- 4NF로 해결

**시간적 데이터**:
- 유효 시간 구간
- 시간적 조인
- 히스토리 관리

### SQL 제약 구현

**FD 강제**:
- 구체화 뷰 + CHECK
- 트리거
- 애플리케이션 레벨

---

정규화 시리즈 완료! 이제 데이터베이스 설계의 핵심 원리를 모두 학습했습니다. 🎉
