---
title: "[데이터베이스] 문제 해설 - 정규화 Part 2 (Exercises 7.29-7.36)"
date: 2025-02-04 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [normalization, bcnf, 3nf, lossless-decomposition, dependency-preserving, canonical-cover, decomposition-algorithm, problem-solving]
pin: false
math: true
mermaid: false
---

이번 포스트에서는 정규화(Normalization) 알고리즘의 실전 적용을 다룹니다. Part 2에서는 무손실 분해 검증, BCNF/3NF 분해 알고리즘의 단계별 수행, 종속성 보존 판단을 중심으로 학습합니다.

## 📚 핵심 개념 복습

### 무손실 분해 조건

**조건**: R₁ ∩ R₂ → R₁ 또는 R₁ ∩ R₂ → R₂

**의미**: 교집합이 한쪽의 슈퍼키

### BCNF vs 3NF

| 특성 | BCNF | 3NF |
|------|------|-----|
| **조건** | 모든 결정자가 슈퍼키 | 결정자가 슈퍼키 또는 피결정자가 후보키 일부 |
| **무손실** | ✅ | ✅ |
| **종속성 보존** | ❌ 보장 안 됨 | ✅ |

---

## 문제 7.29 - 무손실 분해 반례

**문제**: Exercise 7.1의 스키마 R을 다음과 같이 분해하면 무손실 분해가 아님을 보이시오:
- (A, B, C)와 (C, D, E)

힌트: ΠA,B,C(r) ⋈ ΠC,D,E(r) ≠ r인 관계 r(R)을 제시하시오.

```
R = (A, B, C, D, E)
F = {
    A → BC
    CD → E
    B → D
    E → A
}
```

**답변**:

### 무손실 분해 조건 검사

**분해**:
- R₁ = (A, B, C)
- R₂ = (C, D, E)
- R₁ ∩ R₂ = {C}

**무손실 조건**: C → R₁ 또는 C → R₂

**C⁺ 계산**:
```
초기: C⁺ = {C}

F의 각 FD 확인:
- A → BC: A ⊆ {C}? ❌
- CD → E: CD ⊆ {C}? ❌
- B → D: B ⊆ {C}? ❌
- E → A: E ⊆ {C}? ❌

결과: C⁺ = {C}
```

**결론**:
- C⁺ = {C} ≠ R₁ = {A,B,C}
- C⁺ = {C} ≠ R₂ = {C,D,E}
- → ❌ **무손실 분해 아님!**

### 반례 구성

**관계 r(R)**:

| A  | B  | C  | D  | E  |
|----|----|----|----|----|
| a₁ | b₁ | c₁ | d₁ | e₁ |
| a₂ | b₂ | c₁ | d₂ | e₂ |

**함수적 종속성 검증**:

1. **A → BC**:
   - A = a₁: B = b₁, C = c₁ ✅
   - A = a₂: B = b₂, C = c₁ ✅

2. **CD → E**:
   - (c₁, d₁): E = e₁ ✅
   - (c₁, d₂): E = e₂ ✅

3. **B → D**:
   - B = b₁: D = d₁ ✅
   - B = b₂: D = d₂ ✅

4. **E → A**:
   - E = e₁: A = a₁ ✅
   - E = e₂: A = a₂ ✅

**모든 FD 만족** → 합법적 인스턴스 ✅

### 분해와 조인

**Step 1**: R₁로 투영

ΠA,B,C(r):

| A  | B  | C  |
|----|----|-----|
| a₁ | b₁ | c₁ |
| a₂ | b₂ | c₁ |

**Step 2**: R₂로 투영

ΠC,D,E(r):

| C  | D  | E  |
|----|----|----|
| c₁ | d₁ | e₁ |
| c₁ | d₂ | e₂ |

**Step 3**: 자연 조인 (C로)

ΠA,B,C(r) ⋈ ΠC,D,E(r):

```
c₁에서 조인:
(a₁, b₁, c₁) ⋈ (c₁, d₁, e₁) = (a₁, b₁, c₁, d₁, e₁) ✅ 원본
(a₁, b₁, c₁) ⋈ (c₁, d₂, e₂) = (a₁, b₁, c₁, d₂, e₂) ⚠️ 가짜!
(a₂, b₂, c₁) ⋈ (c₁, d₁, e₁) = (a₂, b₂, c₁, d₁, e₁) ⚠️ 가짜!
(a₂, b₂, c₁) ⋈ (c₁, d₂, e₂) = (a₂, b₂, c₁, d₂, e₂) ✅ 원본
```

**결과**:

| A  | B  | C  | D  | E  |
|----|----|----|----|----|
| a₁ | b₁ | c₁ | d₁ | e₁ | ← 원본
| a₁ | b₁ | c₁ | d₂ | e₂ | ← **가짜 튜플!**
| a₂ | b₂ | c₁ | d₁ | e₁ | ← **가짜 튜플!**
| a₂ | b₂ | c₁ | d₂ | e₂ | ← 원본

**검증**:
```
|r| = 2
|ΠA,B,C(r) ⋈ ΠC,D,E(r)| = 4

r ≠ ΠA,B,C(r) ⋈ ΠC,D,E(r)
```

**∴ 무손실 분해 아님!** ❌

### 왜 가짜 튜플이 생기는가?

**원인**: C가 약한 키
- C 값이 c₁로 같음
- 하지만 C만으로는 D, E를 결정 못 함
- 조인 시 모든 조합이 생성됨 (카테시안 곱)

**비유**:
```
학생(학번, 이름, 학과)
과목(학과, 과목명, 학점)

학과로 조인하면?
→ 같은 학과 학생들이 모든 과목과 매칭됨!
```

### 올바른 분해

**무손실 분해 예시**:
- (A, B, C)와 (A, D, E) ✅
  - 교집합: A
  - A는 후보키 → 무손실!

---

## 문제 7.30 - 포괄적 정규화 연습

**문제**: 다음 함수적 종속성 집합 F에 대해:

```
R = (A, B, C, D, E, G)

F = {
    A → B
    BD → E
    AC → D
    E → G
}
```

a. B⁺를 계산하시오.
b. AG가 슈퍼키임을 Armstrong's Axioms로 증명하시오.
c. 정준 커버 Fc를 계산하고 각 단계를 설명하시오.
d. 정준 커버 기반 3NF 분해를 제시하시오.
e. 원래 F를 사용한 BCNF 분해를 제시하시오.

**답변**:

### (a) B⁺ 계산

**알고리즘 적용**:

```
초기: result = {B}

반복 1:
  - A → B: A ⊆ {B}? ❌
  - BD → E: BD ⊆ {B}? ❌ (D 없음)
  - AC → D: AC ⊆ {B}? ❌
  - E → G: E ⊆ {B}? ❌

  result = {B} (변화 없음)

종료
```

**결과**: **B⁺ = {B}**

**의미**:
- B 단독으로는 다른 속성 결정 불가
- B → D가 있으려면 D가 필요 (BD → E)
- B는 후보키 아님

### (b) AG가 슈퍼키임을 증명

**방법 1 - 폐포 계산으로 증명**:

```
AG⁺ 계산:

초기: result = {A, G}

반복 1:
  - A → B: A ⊆ {A,G}? ✅
    result = {A, G, B}
  - BD → E: BD ⊆ {A,G,B}? ❌ (D 없음)
  - AC → D: AC ⊆ {A,G,B}? ❌ (C 없음)
  - E → G: E ⊆ {A,G,B}? ❌

반복 2:
  - A → B: 이미 적용
  - BD → E: BD ⊆ {A,G,B}? ❌
  - AC → D: AC ⊆ {A,G,B}? ❌
  - E → G: 이미 적용 (G는 초기에 있음)
```

**문제 발견**: 이 방법으로는 C, D, E를 얻을 수 없음!

**방법 2 - Armstrong's Axioms 증명**:

실제로는 AG가 슈퍼키가 **아닐 수** 있습니다. 다시 확인해보겠습니다.

**AG⁺ 정확한 계산**:
```
AG⁺ = {A, G}
A → B 적용: {A, G, B}

BD → E를 적용하려면 D 필요
AC → D를 적용하려면 C 필요

→ C와 D를 얻을 방법이 없음!

AG⁺ = {A, G, B} ≠ R
```

**결론**: AG는 슈퍼키가 **아닙니다**!

**올바른 후보키 찾기**:

**ACG⁺ 계산**:
```
초기: {A, C, G}
A → B: {A, C, G, B}
AC → D: {A, C, G, B, D}
BD → E: {A, C, G, B, D, E}

ACG⁺ = R → ACG는 슈퍼키! ✅
```

**수정된 답변**: 문제에서 AG를 증명하라고 했지만, 실제로는 AG가 슈퍼키가 아닙니다. ACG가 슈퍼키입니다.

### (c) 정준 커버 계산

**Step 1**: 오른쪽을 단일 속성으로

```
F = {
    A → B
    BD → E
    AC → D
    E → G
}

→ 이미 모두 단일 속성 ✅
```

**Step 2**: 왼쪽의 불필요한 속성 제거

**BD → E 검사**:
- B 제거 가능? D⁺ (F - {BD → E}) = {D} ≠ E → B 필요
- D 제거 가능? B⁺ = {B} ≠ E → D 필요
- 둘 다 필요 ✅

**AC → D 검사**:
- A 제거 가능? C⁺ = {C} ≠ D → A 필요
- C 제거 가능? A⁺ = {A, B} ≠ D → C 필요
- 둘 다 필요 ✅

**나머지**: 왼쪽이 단일 속성 → 검사 불필요

**Step 3**: 불필요한 FD 제거

**각 FD 검사**:

1. **A → B 필요?**
   ```
   F' = F - {A → B}에서 A⁺ 계산

   초기: {A}
   AC → D? C 없음 → ❌

   A⁺ = {A}
   B ∉ A⁺ → 필요 ✅
   ```

2. **BD → E 필요?**
   ```
   F' = F - {BD → E}에서 BD⁺ 계산

   초기: {B, D}
   A → B? A 없음 → ❌

   BD⁺ = {B, D}
   E ∉ BD⁺ → 필요 ✅
   ```

3. **AC → D 필요?**
   ```
   F' = F - {AC → D}에서 AC⁺ 계산

   초기: {A, C}
   A → B: {A, C, B}
   BD → E? D 없음 → ❌

   AC⁺ = {A, C, B}
   D ∉ AC⁺ → 필요 ✅
   ```

4. **E → G 필요?**
   ```
   명백히 필요 ✅
   ```

### 정준 커버 결과

**Fc = F** (이미 정준 커버):
```
Fc = {
    A → B
    BD → E
    AC → D
    E → G
}
```

### (d) 3NF 분해 (정준 커버 기반)

**Step 1**: 각 FD에 대해 릴레이션 생성

```
R₁ = (A, B)     for A → B
R₂ = (B, D, E)  for BD → E
R₃ = (A, C, D)  for AC → D
R₄ = (E, G)     for E → G
```

**Step 2**: 후보키 포함 릴레이션 확인

**후보키 찾기**:
```
모든 속성: {A, B, C, D, E, G}

시도:
- ACG⁺ = {A, C, G, B, D, E} = R ✅

후보키: ACG
```

**ACG가 완전히 포함된 릴레이션?**
- R₁: A ∈, C ✗, G ✗
- R₂: A ✗
- R₃: A ∈, C ∈, G ✗
- R₄: G ∈

→ 없음! 추가 릴레이션 필요:

```
R₅ = (A, C, G)  for candidate key
```

**Step 3**: 중복 제거

모든 릴레이션이 필요함

### 최종 3NF 분해

**Result**:
```
R₁ = (A, B)
R₂ = (B, D, E)
R₃ = (A, C, D)
R₄ = (E, G)
R₅ = (A, C, G)  -- candidate key
```

**SQL 구현**:
```sql
CREATE TABLE ab_relation (
    A VARCHAR(50),
    B VARCHAR(50),
    PRIMARY KEY (A)
);

CREATE TABLE bde_relation (
    B VARCHAR(50),
    D VARCHAR(50),
    E VARCHAR(50),
    PRIMARY KEY (B, D)
);

CREATE TABLE acd_relation (
    A VARCHAR(50),
    C VARCHAR(50),
    D VARCHAR(50),
    PRIMARY KEY (A, C)
);

CREATE TABLE eg_relation (
    E VARCHAR(50),
    G VARCHAR(50),
    PRIMARY KEY (E)
);

CREATE TABLE acg_relation (
    A VARCHAR(50),
    C VARCHAR(50),
    G VARCHAR(50),
    PRIMARY KEY (A, C, G),
    FOREIGN KEY (A) REFERENCES ab_relation(A)
);
```

### (e) BCNF 분해 (원래 F 사용)

**BCNF 위반 찾기**:

```
F = {
    A → B       (A⁺ = {A,B} ≠ R → A는 슈퍼키 아님) ❌ 위반!
    BD → E      (BD⁺ = ? 계산 필요)
    AC → D      (AC⁺ = ? 계산 필요)
    E → G       (E⁺ = {E,G} ≠ R → E는 슈퍼키 아님) ❌ 위반!
}
```

**A → B로 분해**:

```
Step 1: A → B로 분해
  R₁ = A⁺ = {A, B}
  R₂ = R - (A⁺ - A) = {A, C, D, E, G}

Step 2: R₁ 검사
  R₁ = (A, B)
  F₁ = {A → B}
  A는 R₁의 슈퍼키 → ✅ BCNF

Step 3: R₂ 검사
  R₂ = (A, C, D, E, G)
  F₂에서 관련 FD:
    - BD → E (B ∉ R₂) → 제거
    - AC → D ✅
    - E → G ✅

  F₂ = {AC → D, E → G}

  E → G 검사: E⁺ = {E, G} ≠ R₂ → ❌ BCNF 위반

Step 4: E → G로 R₂ 분해
  R₃ = E⁺ = {E, G}
  R₄ = R₂ - (E⁺ - E) = {A, C, D, E}

Step 5: R₃ 검사
  R₃ = (E, G)
  F₃ = {E → G}
  E는 R₃의 슈퍼키 → ✅ BCNF

Step 6: R₄ 검사
  R₄ = (A, C, D, E)
  F₄ = {AC → D}

  AC⁺ (R₄에서) = {A, C, D} ≠ R₄
  → AC는 슈퍼키 아님 → ❌ 위반

  하지만 E는 단독이고 결정 못 받음
  → ACE가 슈퍼키

  AC → D는 AC가 슈퍼키가 아니므로 위반 아님?

  정확히: ACE⁺ = {A, C, E, D} = R₄
  AC⁺ = {A, C, D} ≠ R₄ → 위반

Step 7: AC → D로 R₄ 분해
  R₅ = AC⁺ = {A, C, D}
  R₆ = R₄ - (AC⁺ - AC) = {A, C, E}

Step 8: R₅ 검사
  R₅ = (A, C, D)
  F₅ = {AC → D}
  AC는 R₅의 슈퍼키 → ✅ BCNF

Step 9: R₆ 검사
  R₆ = (A, C, E)
  F₆ = {} (더 이상 FD 없음)
  → ✅ BCNF (자명한 FD만)
```

### 최종 BCNF 분해

**Result**:
```
R₁ = (A, B)
R₃ = (E, G)
R₅ = (A, C, D)
R₆ = (A, C, E)
```

**검증**:
- ✅ 모든 릴레이션이 BCNF
- ✅ 무손실 분해 (알고리즘 보장)
- ❌ 종속성 보존? BD → E 손실!

**종속성 보존 확인**:
```
원래 F = {A → B, BD → E, AC → D, E → G}

분해 후:
- A → B: R₁에서 검증 ✅
- BD → E: B ∈ R₁, D ∈ R₅, E ∈ R₆ (모두 분산) ❌
- AC → D: R₅에서 검증 ✅
- E → G: R₃에서 검증 ✅
```

**결론**: BCNF 분해는 BD → E를 보존하지 못함!

---

## 문제 7.31 - BCNF 분해와 종속성 보존

**문제**: R = (A, B, C, D, E, G)와 다음 F에 대해:

```
F = {
    AB → CD
    AC → BE
    B → E
    E → C
}
```

a. 왜 AB → CD가 R을 BCNF가 아니게 만드는지 설명
b. AB → CD로 시작하는 BCNF 분해 수행
c. 결과가 종속성을 보존하는지 판단 및 설명

**답변**:

### (a) AB → CD가 BCNF 위반인 이유

**BCNF 조건**: α → β가 비자명하면 α는 슈퍼키여야 함

**AB⁺ 계산**:
```
초기: {A, B}

반복 1:
  - AB → CD: {A, B, C, D}
  - AC → BE: AC ⊆ {A,B,C,D}? ✅ → {A, B, C, D, E}
  - B → E: 이미 E 있음
  - E → C: 이미 C 있음

AB⁺ = {A, B, C, D, E}
```

**G 포함 여부**:
```
AB⁺ = {A, B, C, D, E} ≠ R = {A, B, C, D, E, G}
```

**결론**:
- AB는 슈퍼키가 **아님** (G를 결정 못 함)
- AB → CD는 비자명한 FD
- ∴ **BCNF 위반!** ❌

### (b) BCNF 분해 수행

**Step 1**: AB → CD로 분해

```
R₁ = AB⁺ ∩ R = {A, B, C, D, E}
R₂ = R - (AB⁺ - AB) = R - {C, D, E} = {A, B, G}
```

**Step 2**: R₁ 검사

```
R₁ = (A, B, C, D, E)
F₁ = {AB → CD, AC → BE, B → E, E → C}
     (모두 R₁ 속성만 포함)

BCNF 검사:
1. AB → CD:
   AB⁺ (R₁에서) = {A, B, C, D, E} = R₁ ✅ 슈퍼키

2. AC → BE:
   AC⁺ (R₁에서)?
   AC → BE: {A, C, B, E}
   B → E: 이미
   E → C: 이미
   AB → CD: AB ⊆ {A,C,B,E}? ✅ → {A, C, B, E, D}

   AC⁺ = R₁ ✅ 슈퍼키

3. B → E:
   B⁺ = {B, E, C} (E → C 적용)
   B⁺ ≠ R₁ → B는 슈퍼키 아님 ❌ **위반!**

→ R₁을 더 분해해야 함
```

**Step 3**: B → E로 R₁ 분해

```
R₃ = B⁺ = {B, E, C}  (E → C 적용)
R₄ = R₁ - (B⁺ - B) = {A, B, D}
```

**Step 4**: R₃ 검사

```
R₃ = (B, E, C)
F₃ = {B → E, E → C}

B → E: B⁺ = {B, E, C} = R₃ ✅
E → C: E⁺ = {E, C} ≠ R₃

E⁺ 정확히:
E → C: {E, C}
E⁺ = {E, C} ≠ R₃ → ❌ 위반

→ 더 분해
```

**Step 5**: E → C로 R₃ 분해

```
R₅ = E⁺ = {E, C}
R₆ = R₃ - (E⁺ - E) = {B, E}
```

**Step 6**: R₅, R₆ 검사

```
R₅ = (E, C)
F₅ = {E → C}
E는 슈퍼키 ✅ BCNF

R₆ = (B, E)
F₆ = {B → E}
B는 슈퍼키 ✅ BCNF
```

**Step 7**: R₄ 검사

```
R₄ = (A, B, D)
F₄에서 관련 FD?

AB → CD: C ∉ R₄ → AB → D
AB⁺ (R₄에서) = {A, B, D} = R₄ ✅

AC → BE: C ∉ R₄ → 제거
B → E: E ∉ R₄ → 제거
E → C: E ∉ R₄ → 제거

F₄ = {AB → D}
AB는 R₄의 슈퍼키 ✅ BCNF
```

**Step 8**: R₂ 검사

```
R₂ = (A, B, G)

원래 F에서 G 관련 FD 없음
→ F₂ = {} (자명한 FD만)
→ ✅ BCNF
```

### 최종 BCNF 분해

**Result**:
```
R₂ = (A, B, G)
R₄ = (A, B, D)
R₅ = (E, C)
R₆ = (B, E)
```

### (c) 종속성 보존 판단

**원래 F**:
```
F = {
    AB → CD
    AC → BE
    B → E
    E → C
}
```

**각 FD 검증**:

1. **AB → CD**:
   ```
   C: R₅에 있지만 AB와 분리
   D: R₄에 A, B, D 함께 있음

   R₄에서 AB → D 검증 가능 ✅

   AB → C는?
   A, B ∈ R₄
   B ∈ R₆, E ∈ R₆
   E, C ∈ R₅

   AB → B (자명)
   B → E (R₆)
   E → C (R₅)
   → AB → C (이행성) ✅

   ∴ AB → CD 보존됨 ✅
   ```

2. **AC → BE**:
   ```
   A: R₂, R₄에 있음
   C: R₅에 있음
   B, E: R₆에 있음

   A와 C가 함께 있는 릴레이션 없음!

   분산 확인:
   - A ∈ R₂, R₄
   - C ∈ R₅
   - B, E ∈ R₆

   AC를 함께 투영할 릴레이션 없음 ❌

   ∴ AC → BE 보존 안 됨 ❌
   ```

3. **B → E**:
   ```
   R₆ = (B, E)에서 검증 가능 ✅
   ```

4. **E → C**:
   ```
   R₅ = (E, C)에서 검증 가능 ✅
   ```

### 종속성 보존 결과

**보존됨**: AB → CD, B → E, E → C
**보존 안 됨**: AC → BE

**∴ 종속성을 보존하지 않음** ❌

**이유**:
- BCNF 분해는 종속성 보존을 보장하지 않음
- AC → BE는 A와 C가 분리되어 검증 불가능
- 애플리케이션 레벨에서 추가 검증 필요

### 실무적 함의

**종속성 보존 실패 시**:

1. **제약 조건 검증 복잡**:
   ```sql
   -- AC → BE 검증을 위해 조인 필요
   SELECT DISTINCT a.A, e.C, a.B, e.E
   FROM abd_relation a
   JOIN be_relation b ON a.B = b.B
   JOIN ec_relation e ON b.E = e.E;
   ```

2. **트랜잭션 비용 증가**:
   - 여러 테이블 잠금 필요
   - 일관성 검사 오버헤드

3. **설계 선택**:
   - BCNF (성능) vs 3NF (무결성)
   - 상황에 따라 판단

---

## 문제 7.32 - BCNF 분해 상세 분석

**문제**: R = (A, B, C, D, E, G)와 다음 F에 대해:

```
F = {
    A → BC
    BD → E
    CD → AB
}
```

a. 주어진 3개 FD로부터 논리적으로 유도되는 비자명한 FD를 하나 찾고 설명
b. A → BC로 시작하는 BCNF 분해 수행 및 단계 설명
c. 분해가 무손실인지 판단 및 설명
d. 분해가 종속성을 보존하는지 판단 및 설명

**답변**:

### (a) 유도되는 FD 찾기

**방법**: Armstrong's Axioms와 이행성 활용

**FD 분석**:
```
1. A → BC
2. BD → E
3. CD → AB
```

**시도 1 - 이행성**:
```
A → BC에서:
  A → B
  A → C

A → B이고 B를 포함하는 FD?
  BD → E (B 포함)

하지만 A → BD가 필요 (D가 없음)
```

**시도 2 - CD → AB 활용**:
```
CD → AB에서:
  CD → A
  CD → B

CD → A이고 A → BC이므로:
  CD → BC (이행성)

이미 CD → B이므로:
  CD → BC는 새로운 정보 C만 추가

∴ CD → C (유도됨)
```

**검증**:
```
1. CD → A        (CD → AB 분해)
2. A → C         (A → BC 분해)
3. CD → C        (이행성: 1 + 2)
```

**비자명한가?**
- CD → C에서 C ⊆ CD
- C는 CD의 부분집합 → 자명함 ❌

**다른 시도 - BD → ?**:

```
BD → E (주어짐)

BD에서 더 유도 가능?
BD의 폐포 계산 필요
```

**BD⁺ 계산**:
```
초기: {B, D}
BD → E: {B, D, E}

더 이상 적용 가능한 FD 없음
BD⁺ = {B, D, E}

→ BD → BE (비자명) ✅
→ BD → DE (비자명) ✅
```

**더 흥미로운 FD**:

**A⁺ 계산**:
```
초기: {A}
A → BC: {A, B, C}

BD → E 적용 가능? BD ⊆ {A,B,C}? D 없음 ❌
CD → AB 적용 가능? CD ⊆ {A,B,C}? D 없음 ❌

A⁺ = {A, B, C}
```

**CD⁺ 계산**:
```
초기: {C, D}
CD → AB: {C, D, A, B}
A → BC: 이미 B, C 있음
BD → E: {C, D, A, B, E}

CD⁺ = {A, B, C, D, E}
```

**유도된 비자명한 FD**:

**CD → E** ✅

**증명**:
```
1. CD → AB       (주어짐)
2. CD → B        (1번 분해)
3. CD → A        (1번 분해)
4. A → BC        (주어짐)
5. CD → BC       (3번 + 4번 이행성)
6. BD → E        (주어짐)
7. CD → D        (자명)
8. CD → BD       (2번 + 7번 합집합)
9. CD → E        (8번 + 6번 이행성) ✅
```

**최종 답변**: **CD → E**

### (b) BCNF 분해 (A → BC로 시작)

**Step 1**: A → BC로 분해

```
A⁺ 계산:
A → BC: {A, B, C}
(G가 추가될 방법 없음)

A⁺ = {A, B, C}

R₁ = A⁺ = {A, B, C}
R₂ = R - (A⁺ - A) = {A, D, E, G}
```

**Step 2**: R₁ 검사

```
R₁ = (A, B, C)
F₁에서 R₁ 관련 FD:

- A → BC: ✅ 포함
- BD → E: D, E ∉ R₁ → 제거
- CD → AB: D ∉ R₁ → 제거

F₁ = {A → B, A → C}

BCNF 검사:
A⁺ = {A, B, C} = R₁ → A는 슈퍼키 ✅ BCNF
```

**Step 3**: R₂ 검사

```
R₂ = (A, D, E, G)
F₂에서 R₂ 관련 FD:

- A → BC: B, C ∉ R₂ → 제거
- BD → E: B ∉ R₂ → 제거
- CD → AB: C ∉ R₂ → 제거

F₂ = {} (명시적 FD 없음)

→ ✅ BCNF (자명한 FD만)
```

### 최종 BCNF 분해

**Result**:
```
R₁ = (A, B, C)
R₂ = (A, D, E, G)
```

### (c) 무손실 분해 판단

**분해**: {R₁, R₂}

**교집합**: R₁ ∩ R₂ = {A}

**무손실 조건**: A → R₁ 또는 A → R₂

**A⁺ 계산**:
```
A⁺ = {A, B, C} = R₁ ✅
```

**결론**: A → R₁ 성립 → **✅ 무손실 분해**

**일반적 원리**:
- BCNF 분해 알고리즘은 항상 무손실 보장
- 교집합 = 분해에 사용된 FD의 왼쪽
- 그 속성의 폐포 = 한쪽 릴레이션

### (d) 종속성 보존 판단

**원래 F**:
```
F = {
    A → BC
    BD → E
    CD → AB
}
```

**각 FD 검증**:

1. **A → BC**:
   ```
   R₁ = (A, B, C)에서 검증 가능 ✅
   ```

2. **BD → E**:
   ```
   B: R₁에 있음
   D, E: R₂에 있음

   B와 D가 함께 있는 릴레이션 없음!

   투영 시도:
   π_BD(R₁) = B만 (D 없음)
   π_BD(R₂) = D만 (B 없음)

   ∴ BD → E 보존 안 됨 ❌
   ```

3. **CD → AB**:
   ```
   C: R₁에 있음
   D: R₂에 있음

   C와 D가 함께 있는 릴레이션 없음!

   ∴ CD → AB 보존 안 됨 ❌
   ```

### 종속성 보존 결과

**보존됨**: A → BC
**보존 안 됨**: BD → E, CD → AB

**∴ 종속성을 보존하지 않음** ❌

**설명**:
- A → BC만 보존
- BD → E와 CD → AB는 B, C, D가 분산되어 검증 불가
- 이것이 BCNF의 한계: 종속성 보존 보장 안 함
- 무결성 검증을 위해 조인 필요

**대안 - 3NF**:
```
3NF 분해하면 종속성 보존 가능
하지만 중복 가능성 존재
```

---

## 💡 핵심 정리

### 무손실 분해 vs 종속성 보존

| 특성 | 무손실 분해 | 종속성 보존 |
|------|-------------|-------------|
| **의미** | 조인해도 원본 복구 | FD를 개별 릴레이션에서 검증 가능 |
| **BCNF** | ✅ 보장 | ❌ 보장 안 됨 |
| **3NF** | ✅ 보장 | ✅ 보장 |
| **중요도** | 필수 (데이터 손실 방지) | 선택적 (무결성 검증 편의) |

### BCNF 분해 알고리즘

```
while (R이 BCNF 아님):
    BCNF 위반하는 FD α → β 찾기
    R을 (α⁺)와 (R - (α⁺ - α))로 분해
    각 분해된 릴레이션에 대해 반복

결과:
- ✅ 무손실 분해 보장
- ❌ 종속성 보존 보장 안 됨
```

### 3NF 분해 알고리즘

```
1. 정준 커버 Fc 계산
2. 각 FD α → β에 대해 Rᵢ = αβ 생성
3. 후보키 포함 릴레이션 확인, 없으면 추가
4. 중복 릴레이션 제거

결과:
- ✅ 무손실 분해 보장
- ✅ 종속성 보존 보장
```

### 실무 선택 가이드

**BCNF 선택**:
- 성능 최우선
- 중복 최소화
- 조인 비용 감수 가능
- 애플리케이션 레벨 검증 가능

**3NF 선택**:
- 무결성 최우선
- 제약 조건 검증 중요
- 조인 비용 최소화
- 약간의 중복 허용 가능

---

Part 3에서는 정규화 설계 목표, 4NF, 시간적 조인, SQL 제약 조건 구현을 다룹니다!
