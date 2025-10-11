---
title: "[데이터베이스] 6. 관계형 데이터베이스 설계와 정규화 (2) - 함수적 종속성 이론과 분해 알고리즘"
date: 2025-01-14 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [relational-design, normalization, armstrong-axioms, canonical-cover, bcnf-decomposition, 3nf-decomposition, dependency-preservation, theory, advanced]
math: true
mermaid: true
---

## 1. 개요

이전 포스트에서는 정규화의 기본 개념과 정규형에 대해 학습했습니다. 이번 포스트에서는 함수적 종속성의 형식 이론(Formal Theory)과 실제로 BCNF 및 3NF로 분해하는 알고리즘을 다룹니다.

### 학습 목표

- Armstrong의 공리 이해 및 적용
- 함수적 종속성 폐쇄(Closure) 계산
- 정준 커버(Canonical Cover) 생성
- BCNF 및 3NF 분해 알고리즘 적용
- 종속성 보존(Dependency Preservation) 검증

---

## 2. 함수적 종속성 이론 로드맵

정규화를 체계적으로 수행하기 위해서는 다음 세 가지 핵심 이론을 이해해야 합니다:

1. **논리적 함의(Logical Implication)**: 주어진 FD 집합 F로부터 어떤 FD가 논리적으로 유도되는지 판단하는 형식 이론
2. **무손실 분해 알고리즘**: BCNF와 3NF로 무손실 분해를 생성하는 알고리즘
3. **종속성 보존 검사**: 분해가 종속성 보존 특성을 만족하는지 테스트하는 알고리즘

---

## 3. Armstrong의 공리 (Armstrong's Axioms)

### 3.1 기본 공리

Armstrong의 공리는 함수적 종속성의 폐쇄 $F^+$를 계산하기 위한 세 가지 기본 규칙입니다:

**1. 반사 법칙 (Reflexivity Rule)**
- $\beta \subseteq \alpha$ 이면 $\alpha \rightarrow \beta$
- 예: $\{A, B\} \rightarrow A$, $\{A, B\} \rightarrow B$

**2. 증가 법칙 (Augmentation Rule)**
- $\alpha \rightarrow \beta$ 이면 $\gamma\alpha \rightarrow \gamma\beta$
- 예: $A \rightarrow B$ 이면 $AC \rightarrow BC$

**3. 이행 법칙 (Transitivity Rule)**
- $\alpha \rightarrow \beta$이고 $\beta \rightarrow \gamma$ 이면 $\alpha \rightarrow \gamma$
- 예: $A \rightarrow B$이고 $B \rightarrow C$ 이면 $A \rightarrow C$

### 3.2 공리의 특성

Armstrong의 공리는 다음 두 가지 중요한 특성을 가집니다:

- **건전성(Sound)**: 실제로 성립하는 함수적 종속성만을 생성
- **완전성(Complete)**: 성립하는 모든 함수적 종속성을 생성 가능

### 3.3 추가 규칙

기본 공리로부터 유도할 수 있는 유용한 추가 규칙들:

**1. 합집합 법칙 (Union Rule)**
- $\alpha \rightarrow \beta$이고 $\alpha \rightarrow \gamma$ 이면 $\alpha \rightarrow \beta\gamma$

**2. 분해 법칙 (Decomposition Rule)**
- $\alpha \rightarrow \beta\gamma$ 이면 $\alpha \rightarrow \beta$이고 $\alpha \rightarrow \gamma$

**3. 의사이행 법칙 (Pseudotransitivity Rule)**
- $\alpha \rightarrow \beta$이고 $\gamma\beta \rightarrow \delta$ 이면 $\alpha\gamma \rightarrow \delta$

### 3.4 예제: $F^+$ 계산

```
R = (A, B, C, G, H, I)
F = { A → B, A → C, CG → H, CG → I, B → H }
```

$F^+$의 멤버들을 유도해봅시다:

**1. $A \rightarrow H$**
- 이행 법칙: $A \rightarrow B$와 $B \rightarrow H$로부터

**2. $AG \rightarrow I$**
- 증가 법칙: $A \rightarrow C$에 G를 추가하면 $AG \rightarrow CG$
- 이행 법칙: $AG \rightarrow CG$와 $CG \rightarrow I$로부터

**3. $CG \rightarrow HI$**
- 증가 법칙: $CG \rightarrow I$에 CG를 추가하면 $CG \rightarrow CGI$
- 증가 법칙: $CG \rightarrow H$에 I를 추가하면 $CGI \rightarrow HI$
- 이행 법칙: $CG \rightarrow HI$

---

## 4. 속성 집합의 폐쇄 (Closure of Attribute Sets)

### 4.1 정의

속성 B가 $\alpha \rightarrow B$를 만족하면, B는 $\alpha$에 의해 **함수적으로 결정된다**(functionally determined)고 합니다.

속성 집합 $\alpha$의 **폐쇄** $\alpha^+$는 F 하에서 $\alpha$에 의해 함수적으로 결정되는 모든 속성의 집합입니다.

$$\alpha \rightarrow \beta \text{가 } F^+ \text{에 속함} \Leftrightarrow \beta \subseteq \alpha^+$$

### 4.2 속성 폐쇄 계산 알고리즘

```
입력: 속성 집합 α, 함수적 종속성 집합 F
출력: α^+

result := α
while (result에 변화가 있음) do
    for each β → γ in F do
        if β ⊆ result then
            result := result ∪ γ
return result
```

### 4.3 예제: 속성 폐쇄 계산

```
R = (A, B, C, G, H, I)
F = { A → B, A → C, CG → H, CG → I, B → H }
```

**(AG)+ 계산:**

1. `result = AG`
2. $A \rightarrow B$와 $A \rightarrow C$ 적용: `result = ABCG`
3. $CG \rightarrow H$ 적용: `result = ABCGH`
4. $CG \rightarrow I$ 적용: `result = ABCGHI`

**(AG)+ = ABCGHI**이므로 AG는 후보 키입니다!

**AG가 후보 키인지 검증:**

1. **AG가 슈퍼키인가?**
   - $AG \rightarrow R$인가? ⇔ $R = (AG)^+$인가?
   - $(AG)^+ = ABCGHI = R$ ✓

2. **AG의 부분집합이 슈퍼키인가?**
   - $A \rightarrow R$인가? ⇔ $(A)^+ = ABC \neq R$ ✗
   - $G \rightarrow R$인가? ⇔ $(G)^+ = G \neq R$ ✗

따라서 **AG는 후보 키**입니다!

### 4.4 속성 폐쇄의 활용

**1. 슈퍼키 테스트**
- $\alpha$가 슈퍼키인지 확인: $\alpha^+$가 R의 모든 속성을 포함하는지 검사

**2. 함수적 종속성 테스트**
- $\alpha \rightarrow \beta$가 성립하는지 확인: $\beta \subseteq \alpha^+$인지 검사
- 간단하고 효율적인 테스트 방법

**3. F의 폐쇄 계산**
- 각 $\gamma \subseteq R$에 대해 $\gamma^+$를 구함
- 각 $S \subseteq \gamma^+$에 대해 $\gamma \rightarrow S$를 출력

---

## 5. 정준 커버 (Canonical Cover)

### 5.1 필요성

데이터베이스 업데이트 시 시스템은 모든 FD 위반을 검사해야 합니다. 이때 F와 동일한 폐쇄를 가지지만 더 간단한 FD 집합을 사용하면 검사 비용을 줄일 수 있습니다.

이러한 **최소화된 FD 집합**을 **정준 커버(Canonical Cover)**라고 합니다.

### 5.2 외래 속성 (Extraneous Attributes)

**왼쪽의 외래 속성 제거**

$AB \rightarrow C$에서 B를 제거하면 $A \rightarrow C$가 됩니다:
- 더 강한 제약(stronger constraint)
- $A \rightarrow C$는 $AB \rightarrow C$를 논리적으로 함의
- 하지만 $AB \rightarrow C$는 단독으로 $A \rightarrow C$를 함의하지 않음

**예제:**
```
F = { AB → C, A → D, D → C }
```
- F는 $A \rightarrow C$를 논리적으로 함의 (이행 법칙)
- 따라서 $AB \rightarrow C$에서 **B는 외래 속성**

**오른쪽의 외래 속성 제거**

$AB \rightarrow CD$에서 C를 제거하면 $AB \rightarrow D$가 됩니다:
- 더 약한 제약(weaker constraint)
- $AB \rightarrow D$만으로는 $AB \rightarrow C$를 유도할 수 없음

**예제:**
```
F = { AB → CD, A → C }
```
- $AB \rightarrow D$로 바꿔도 $AB \rightarrow C$를 여전히 유도 가능
- 따라서 $AB \rightarrow CD$에서 **C는 외래 속성**

### 5.3 외래 속성의 형식적 정의

FD $\alpha \rightarrow \beta$가 F에 속할 때:

**왼쪽에서 제거 (Remove from left side):**
- $A \in \alpha$이고
- $F$가 $(F - \{\alpha \rightarrow \beta\}) \cup \{(\alpha - A) \rightarrow \beta\}$를 논리적으로 함의하면
- A는 $\alpha$에서 외래

**오른쪽에서 제거 (Remove from right side):**
- $A \in \beta$이고
- $(F - \{\alpha \rightarrow \beta\}) \cup \{\alpha \rightarrow (\beta - A)\}$가 F를 논리적으로 함의하면
- A는 $\beta$에서 외래

### 5.4 외래 속성 테스트 알고리즘

**오른쪽의 속성 A가 $\beta$에서 외래인지 테스트:**

```
F' = (F - {α → β}) ∪ {α → (β - A)}
α^+ under F'를 계산
if A ∈ α^+ then
    A는 β에서 외래
```

**왼쪽의 속성 A가 $\alpha$에서 외래인지 테스트:**

```
γ = α - {A}
F를 사용하여 γ^+를 계산
if γ^+가 β의 모든 속성을 포함 then
    A는 α에서 외래
```

**예제:**
```
F = { AB → CD, A → E, E → C }
```

AB → CD에서 C가 외래인지 확인:
1. `F' = { AB → D, A → E, E → C }`
2. (AB)+ under F' 계산: `(AB)+ = ABCDE`
3. C ∈ (AB)+ ✓
4. **C는 외래 속성**

### 5.5 정준 커버의 정의

$F_C$가 F의 정준 커버이려면:

1. $F$가 $F_C$의 모든 종속성을 논리적으로 함의
2. $F_C$가 $F$의 모든 종속성을 논리적으로 함의
3. $F_C$의 어떤 FD도 외래 속성을 포함하지 않음
4. $F_C$의 각 FD 좌변이 유일 (같은 좌변을 가진 FD가 없음)

$$F_C^+ = F^+$$

**직관적 의미:** F와 동등하면서 중복성이 없는 최소 FD 집합

### 5.6 정준 커버 계산 알고리즘

```
FC = F

repeat
    // 1단계: 합집합 규칙 적용
    같은 좌변을 가진 FD들을 합침
    α1 → β1과 α1 → β2를 α1 → β1β2로 변경

    // 2단계: 외래 속성 제거
    for each FD α → β in FC do
        if α 또는 β에 외래 속성이 있으면
            외래 속성을 제거

until FC에 변화가 없음

return FC
```

### 5.7 정준 커버 계산 예제

```
R = (A, B, C)
F = { A → BC, B → C, A → B, AB → C }
```

**단계 1:** 합집합 규칙 적용
- $A \rightarrow BC$와 $A \rightarrow B$를 $A \rightarrow BC$로 합침
- `FC = { A → BC, B → C, AB → C }`

**단계 2:** AB → C에서 A는 외래
- B → C가 이미 존재하므로 AB → C에서 A 제거 가능
- `FC = { A → BC, B → C }`

**단계 3:** A → BC에서 C는 외래
- A → B와 B → C로부터 A → C 유도 가능 (이행)
- `FC = { A → BC, B → C }`에서 C 제거

**결과:**

$$F_C = \{ A \rightarrow B, B \rightarrow C \}$$

---

## 6. 종속성 보존 (Dependency Preservation)

### 6.1 정의

R을 $\{R_1, ..., R_n\}$으로 분해할 때:

- $F_i$: $F^+$에서 $R_i$의 속성만 포함하는 모든 FD
- $F' = F_1 \cup F_2 \cup ... \cup F_n$

**종속성 보존 조건:**

$$F'^+ = (F_1 \cup F_2 \cup ... \cup F_n)^+ = F^+$$

### 6.2 종속성 보존의 중요성

**종속성이 보존되지 않으면:**
- FD 위반 검사를 위해 조인(join) 연산 필요
- 비용이 많이 들고 비효율적

**종속성이 보존되면:**
- 각 $R_i$에서 $F_i$만 검사하면 충분
- $F_i$의 모든 FD는 $R_i$의 속성만 포함하므로 조인 없이 검사 가능

### 6.3 종속성 보존 테스트 알고리즘

$\alpha \rightarrow \beta$가 분해에서 보존되는지 테스트:

```
result = α
repeat
    for each Ri in the decomposition
        t = (result ∩ Ri)^+ ∩ Ri    // Fi 하에서 폐쇄 계산
        result = result ∪ t
until (result does not change)

if result contains all attributes in β then
    α → β is preserved
```

이 절차는 $F^+$와 $(F_1 \cup F_2 \cup ... \cup F_n)^+$를 계산하는 지수 시간 대신 **다항 시간**에 실행됩니다.

---

## 7. BCNF 분해 알고리즘

### 7.1 BCNF 검사

**단일 FD 검사:**
비자명한 FD $\alpha \rightarrow \beta$가 BCNF를 위반하는지 확인:
1. $\alpha^+$ 계산
2. $\alpha^+$가 R의 모든 속성을 포함하는지 확인 (슈퍼키인지 확인)

**릴레이션 스키마 검사:**
- **간소화된 테스트**: F의 FD만 검사 (F+ 전체를 검사할 필요 없음)
- F의 어떤 FD도 BCNF를 위반하지 않으면, F+의 FD도 위반하지 않음

**주의:** 분해된 릴레이션 검사 시에는 간소화된 테스트가 부정확할 수 있음!

**반례:**
```
R = (A, B, C, D, E)
F = { A → B, BC → D }

R을 R1 = (A, B)와 R2 = (A, C, D, E)로 분해

F의 어떤 FD도 R2의 속성만 포함하지 않음
→ R2가 BCNF를 만족한다고 오판할 수 있음

하지만 F+에서 AC → D가 존재하여 R2는 BCNF 위반!
```

### 7.2 BCNF 분해 알고리즘

```
result := {R}
done := false
compute F+

while (not done) do
    if (result에 BCNF가 아닌 스키마 Ri 존재) then
        // Ri에서 성립하는 비자명한 FD α → β를 찾음
        // 단, α → Ri ∉ F+ (α가 슈퍼키가 아님)
        // 그리고 α ∩ β = ∅

        result := (result - Ri) ∪ (Ri - β) ∪ (α, β)
    else
        done := true

return result
```

**알고리즘 특성:**
- 각 $R_i$는 BCNF
- 분해는 **무손실(lossless-join)**
- 하지만 종속성 보존은 보장하지 않음

### 7.3 BCNF 분해 예제

```
class(course_id, title, dept_name, credits, sec_id, semester,
      year, building, room_number, capacity, time_slot_id)

함수적 종속성:
F1: course_id → title, dept_name, credits
F2: building, room_number → capacity
F3: course_id, sec_id, semester, year → building, room_number, time_slot_id

후보 키: {course_id, sec_id, semester, year}
```

**1단계: F1 처리**

`course_id → title, dept_name, credits`는 BCNF 위반 (course_id는 슈퍼키가 아님)

분해:
- `course(course_id, title, dept_name, credits)`
- `class-1(course_id, sec_id, semester, year, building, room_number, capacity, time_slot_id)`

**2단계: F2 처리**

`building, room_number → capacity`는 class-1에서 BCNF 위반

분해:
- `classroom(building, room_number, capacity)`
- `section(course_id, sec_id, semester, year, building, room_number, time_slot_id)`

**최종 결과:**
```
course(course_id, title, dept_name, credits)
classroom(building, room_number, capacity)
section(course_id, sec_id, semester, year, building,
        room_number, time_slot_id)
```

모든 릴레이션이 BCNF를 만족합니다!

---

## 8. 3NF 분해 알고리즘

### 8.1 3NF 검사

BCNF와 유사하지만 추가 조건 확인:
1. $\alpha$가 슈퍼키인지 확인 (속성 폐쇄 사용)
2. $\alpha$가 슈퍼키가 아니면, $\beta$의 각 속성이 후보 키에 포함되는지 확인

**복잡성:**
- 후보 키 찾기가 필요하므로 비용이 높음
- 3NF 테스트는 NP-hard로 알려짐
- 하지만 3NF 분해는 **다항 시간**에 가능!

### 8.2 3NF 분해 알고리즘

```
// 입력: 릴레이션 R, 함수적 종속성 F
// 출력: 3NF 분해 결과

Let FC be a canonical cover for F
i := 0

// 1단계: 각 FD에 대해 릴레이션 생성
for each FD α → β in FC do
    i := i + 1
    Ri := αβ

// 2단계: 후보 키 보존 확인
if none of Rj (1 ≤ j ≤ i) contains a candidate key for R then
    i := i + 1
    Ri := any candidate key for R

// 3단계: 중복 릴레이션 제거 (선택사항)
repeat
    if any schema Rj is contained in another schema Rk then
        Rj := Ri
        i := i - 1
until no more Rj s can be deleted

return (R1, R2, ..., Ri)
```

**알고리즘 보장:**
- 각 $R_i$는 3NF
- 분해는 **종속성 보존**
- 분해는 **무손실 조인**

### 8.3 3NF 분해 예제

```
cust_banker_branch(customer_id, employee_id, branch_name, type)

함수적 종속성:
F1: customer_id, employee_id → branch_name, type
F2: employee_id → branch_name
F3: customer_id, branch_name → employee_id
```

**1단계: 정준 커버 계산**

F1에서 branch_name이 외래:
- employee_id → branch_name이 존재하므로
- customer_id, employee_id → type으로 단순화

정준 커버:
```
FC = {
    customer_id, employee_id → type,
    employee_id → branch_name,
    customer_id, branch_name → employee_id
}
```

**2단계: for 루프로 3NF 스키마 생성**

```
R1 = (customer_id, employee_id, type)
R2 = (employee_id, branch_name)
R3 = (customer_id, branch_name, employee_id)
```

R1이 후보 키를 포함하므로 추가 릴레이션 불필요

**3단계: 중복 제거**

R2 = (employee_id, branch_name)는 R3의 부분집합
→ R2 제거

**최종 3NF 스키마:**
```
R1 = (customer_id, employee_id, type)
R3 = (customer_id, branch_name, employee_id)
```

---

## 9. 설계 목표와 트레이드오프

### 9.1 이상적인 설계 목표

관계형 데이터베이스 설계의 이상적인 목표:

1. **BCNF** 달성
2. **무손실 조인(Lossless)** 보장
3. **종속성 보존(Dependency Preservation)** 유지

### 9.2 현실적인 트레이드오프

**달성 불가능한 경우:**
- **종속성 보존 포기**: BCNF 사용 (조인으로 FD 검사)
- **중복성 허용**: 3NF 사용 (종속성 보존 유지)

### 9.3 SQL의 한계

SQL은 슈퍼키 외의 FD를 직접 명시하는 방법을 제공하지 않습니다:
- assertion으로 명시 가능하지만 검사 비용이 높음
- 현재 주요 DBMS들은 assertion을 지원하지 않음
- 따라서 종속성 보존 분해를 사용해도 좌변이 키가 아닌 FD는 효율적으로 테스트할 수 없음

---

## 10. 정규화의 실무적 고려사항

### 10.1 정규화와 E-R 모델

**잘 설계된 E-R 다이어그램:**
- 모든 엔티티를 올바르게 식별하면
- 생성된 테이블은 추가 정규화가 불필요

**실제 설계에서의 문제:**
- 엔티티의 비키 속성 간 FD 존재 가능 (BCNF 위반)
- 예: `employee(ID, name, dept_name, building, salary)`
  - FD: `dept_name → building`
  - 올바른 설계: department를 별도 엔티티로

### 10.2 성능을 위한 비정규화

**비정규화 사례:**

모든 course 정보 조회 시 prereq을 항상 조인해야 하는 경우:

**대안 1:** 비정규화된 릴레이션 사용
- course와 prereq의 모든 속성 포함
- 장점: 빠른 조회
- 단점: 저장 공간 증가, 업데이트 시간 증가, 추가 코딩 작업, 오류 가능성

**대안 2:** 구체화된 뷰(Materialized View) 사용
- `course ⋈ prereq`의 구체화된 뷰 생성
- 장점: 대안 1과 동일한 성능, 코딩 작업 불필요
- 단점: 여전히 저장 공간과 업데이트 비용 증가

### 10.3 정규화의 한계

정규화로 포착할 수 없는 나쁜 설계 사례:

**예제 1: 연도별 테이블 분할**

```sql
-- 나쁜 설계 (BCNF이지만 비실용적)
earnings_2004(company_id, earnings)
earnings_2005(company_id, earnings)
earnings_2006(company_id, earnings)
...

-- 좋은 설계
earnings(company_id, year, amount)
```

문제점:
- 매년 새 테이블 생성 필요
- 매년 새 쿼리 작성 필요
- 연도별 통계 쿼리가 복잡해짐

**예제 2: 크로스탭(Crosstab)**

```sql
-- 나쁜 설계 (BCNF이지만 확장 불가)
company_year(company_id, earnings_2004, earnings_2005, earnings_2006)

-- 좋은 설계
earnings(company_id, year, amount)
```

크로스탭은 스프레드시트나 데이터 분석 도구에서 유용하지만, 관계형 데이터베이스 설계로는 부적절합니다.

---

## 11. 요약

### 핵심 개념

1. **Armstrong의 공리**: 함수적 종속성 유도의 형식적 기반
2. **속성 폐쇄**: 슈퍼키 테스트와 FD 검증의 효율적 방법
3. **정준 커버**: 중복 없는 최소 FD 집합
4. **BCNF 분해**: 무손실 분해 보장, 종속성 보존은 미보장
5. **3NF 분해**: 무손실 분해와 종속성 보존 모두 보장

### 알고리즘 비교

| 특성 | BCNF 분해 | 3NF 분해 |
|------|-----------|----------|
| 정규형 | BCNF | 3NF |
| 무손실 조인 | ✓ | ✓ |
| 종속성 보존 | ✗ | ✓ |
| 중복성 | 없음 | 일부 가능 |
| 알고리즘 복잡도 | 지수 시간 가능 | 다항 시간 |

### 설계 지침

1. **이상적 목표**: BCNF + 무손실 + 종속성 보존
2. **달성 불가 시**: 3NF 사용 (종속성 보존 우선) 또는 BCNF 사용 (중복 제거 우선)
3. **성능 고려**: 필요시 선택적 비정규화 또는 구체화된 뷰 사용
4. **E-R 모델 활용**: 초기 설계를 잘하면 정규화 작업 최소화

---

## 참고 자료

- Database System Concepts (7th Edition) - Chapter 7
- Armstrong, W. W. (1974). "Dependency Structures of Data Base Relationships"
- Codd, E. F. (1972). "Further Normalization of the Data Base Relational Model"

---

**다음 포스트 예고**: 데이터 저장 구조와 파일 조직에 대해 학습합니다.
