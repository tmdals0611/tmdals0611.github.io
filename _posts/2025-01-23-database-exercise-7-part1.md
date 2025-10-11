---
title: "[데이터베이스] 문제 해설 #7 Part 1 - 함수적 종속성과 정규화 기초"
date: 2025-01-23 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [normalization, functional-dependency, lossless-decomposition, armstrong-axioms, closure, canonical-cover, bcnf, 3nf]
pin: false
math: true
mermaid: false
---

이번 포스트에서는 관계형 데이터베이스 설계의 핵심인 함수적 종속성(Functional Dependency)과 정규화(Normalization) 문제들을 다룹니다. Part 1에서는 기초 개념과 Armstrong's Axioms를 중심으로 학습합니다.

## 📚 핵심 개념 정리

### 함수적 종속성 (Functional Dependency)
- **표기법**: α → β
- **의미**: α 값이 같으면 β 값도 같음
- **비자명한(Nontrivial)**: β ⊈ α

### Armstrong's Axioms (추론 규칙)
1. **재귀성(Reflexivity)**: β ⊆ α이면 α → β
2. **증대성(Augmentation)**: α → β이면 γα → γβ
3. **이행성(Transitivity)**: α → β이고 β → γ이면 α → γ

### 추가 규칙
- **합집합(Union)**: α → β이고 α → γ이면 α → βγ
- **분해(Decomposition)**: α → βγ이면 α → β이고 α → γ
- **의사이행성(Pseudotransitivity)**: α → β이고 γβ → δ이면 αγ → δ

### 속성 폐포 (Attribute Closure)
- **표기법**: α⁺
- **정의**: α로부터 유도 가능한 모든 속성의 집합
- **용도**: 후보키 찾기, 함수적 종속성 검증

---

## 문제 7.1 - 무손실 분해 증명

**문제**: R = (A, B, C, D, E)를 (A, B, C)와 (A, D, E)로 분해. 다음 함수적 종속성 집합 F가 주어질 때 무손실 분해임을 보이시오.

```
F = {
    A → BC
    CD → E
    B → D
    E → A
}
```

**답변**:

### 무손실 분해 조건

분해 {R₁, R₂}가 무손실이려면:
- R₁ ∩ R₂ → R₁ 또는
- R₁ ∩ R₂ → R₂

### 풀이

**주어진 것**:
- R₁ = (A, B, C)
- R₂ = (A, D, E)
- R₁ ∩ R₂ = A

**증명**:
1. A가 후보키인지 확인 (문제 7.6 참조)
2. A⁺ = {A, B, C, D, E} = R
3. A → R이므로 A → R₁
4. R₁ ∩ R₂ = A → R₁ 성립
5. ✅ **무손실 분해**

**핵심**:
- 교집합 속성이 한쪽 릴레이션의 슈퍼키면 무손실
- A가 R의 후보키이므로 당연히 R₁의 슈퍼키

---

## 문제 7.2 - 관계 인스턴스에서 함수적 종속성 찾기

**문제**: 다음 관계에서 만족하는 모든 비자명한 함수적 종속성을 나열하시오.

| A  | B  | C  |
|----|----|----|
| a₁ | b₁ | c₁ |
| a₁ | b₁ | c₂ |
| a₂ | b₁ | c₁ |
| a₂ | b₁ | c₃ |

**답변**:

### 비자명한 함수적 종속성

**찾은 FD**:
1. **A → B** ✅
2. **C → B** ✅
3. **AC → B** (1, 2로부터 유도)

### 검증 과정

**A → B 검증**:
- A = a₁인 튜플들 (1, 2행): B = b₁로 동일 ✅
- A = a₂인 튜플들 (3, 4행): B = b₁로 동일 ✅

**A → C 불성립**:
- A = a₁인 튜플들: C = c₁, c₂ (다름) ❌

**B → A 불성립**:
- B = b₁인 튜플들: A = a₁, a₂ (다름) ❌

**B → C 불성립**:
- B = b₁인 튜플들: C = c₁, c₂, c₃ (다름) ❌

**C → A 불성립**:
- C = c₁인 튜플들 (1, 3행): A = a₁, a₂ (다름) ❌

**C → B 검증**:
- C = c₁인 튜플들 (1, 3행): B = b₁로 동일 ✅
- C = c₂인 튜플들 (2행): B = b₁ ✅
- C = c₃인 튜플들 (4행): B = b₁ ✅

### 자명한 함수적 종속성

형태: α → β (β ⊆ α)
- A → A, B → B, C → C
- AB → A, AB → B, AB → AB
- AC → A, AC → C, AC → AC
- BC → B, BC → C, BC → BC
- ABC → A, ABC → B, ABC → C, ...

**총 19개의 자명한 FD 존재**

---

## 문제 7.3 - FD를 이용한 카디널리티 표현

**문제**: 함수적 종속성으로 다음을 표현하는 방법은?
- student와 instructor 사이의 일대일 관계
- student와 instructor 사이의 다대일 관계

**답변**:

### 일대일 관계 (1:1)

**함수적 종속성**:
```
Pk(student) → Pk(instructor)
Pk(instructor) → Pk(student)
```

**의미**:
- 같은 student 값 → 같은 instructor 값
- 같은 instructor 값 → 같은 student 값
- 양방향 모두 함수적 종속

**예시**:
```sql
advisor(student_id, instructor_id)
-- student_id → instructor_id (한 학생은 한 교수)
-- instructor_id → student_id (한 교수는 한 학생)
```

### 다대일 관계 (M:1)

**함수적 종속성**:
```
Pk(student) → Pk(instructor)
```

**의미**:
- 같은 student 값 → 같은 instructor 값 (한 학생은 한 교수)
- instructor 값 반복 가능 (한 교수는 여러 학생)

**예시**:
```sql
advisor(student_id, instructor_id)
-- student_id → instructor_id
-- 여러 student_id가 같은 instructor_id 가능
```

---

## 문제 7.4 - Union Rule 증명

**문제**: Armstrong's Axioms를 사용하여 Union Rule의 건전성을 증명하시오.

**Union Rule**: α → β이고 α → γ이면 α → βγ

**증명**:

```
1. α → β                         (주어짐)
2. αα → αβ                       (증대 규칙)
3. α → αβ                        (동일 집합의 합집합)
4. α → γ                         (주어짐)
5. αβ → γβ                       (증대 규칙)
6. α → βγ                        (이행 규칙 + 교환법칙)
```

**상세 설명**:

**Step 1**: α → β (가정)

**Step 2**: 증대 규칙 적용
- α → β에 α를 양쪽에 추가
- αα → αβ

**Step 3**: αα = α (멱등성)
- α → αβ

**Step 4**: α → γ (가정)

**Step 5**: 증대 규칙 적용
- α → γ에 β를 양쪽에 추가
- αβ → γβ

**Step 6**: 이행 규칙
- α → αβ (Step 3)
- αβ → γβ (Step 5)
- α → γβ (이행성)
- γβ = βγ (교환법칙)
- ∴ α → βγ ✅

---

## 문제 7.5 - Pseudotransitivity Rule 증명

**문제**: Armstrong's Axioms를 사용하여 Pseudotransitivity Rule의 건전성을 증명하시오.

**Pseudotransitivity Rule**: α → β이고 γβ → δ이면 αγ → δ

**증명**:

```
1. α → β                         (주어짐)
2. αγ → γβ                       (증대 규칙 + 교환법칙)
3. γβ → δ                        (주어짐)
4. αγ → δ                        (이행 규칙)
```

**상세 설명**:

**Step 1**: α → β (가정)

**Step 2**: 증대 규칙
- α → β에 γ를 양쪽에 추가
- αγ → βγ
- 교환법칙: βγ = γβ
- ∴ αγ → γβ

**Step 3**: γβ → δ (가정)

**Step 4**: 이행 규칙
- αγ → γβ (Step 2)
- γβ → δ (Step 3)
- ∴ αγ → δ ✅

---

## 문제 7.6 - 속성 폐포와 후보키 찾기

**문제**: R = (A, B, C, D, E)에 대한 F⁺를 계산하고 후보키를 나열하시오.

```
F = {
    A → BC
    CD → E
    B → D
    E → A
}
```

**답변**:

### 속성 폐포 계산

**A⁺ 계산**:
```
초기: A⁺ = {A}

1단계: A → BC 적용
   A⁺ = {A, B, C}

2단계: B → D 적용 (B ∈ A⁺)
   A⁺ = {A, B, C, D}

3단계: CD → E 적용 (C, D ∈ A⁺)
   A⁺ = {A, B, C, D, E}

결과: A⁺ = R → A는 후보키!
```

**E⁺ 계산**:
```
초기: E⁺ = {E}

1단계: E → A 적용
   E⁺ = {E, A}

2단계: A → BC 적용
   E⁺ = {E, A, B, C}

3단계: B → D 적용
   E⁺ = {E, A, B, C, D}

결과: E⁺ = R → E는 후보키!
```

**BC⁺ 계산**:
```
초기: BC⁺ = {B, C}

1단계: B → D 적용
   BC⁺ = {B, C, D}

2단계: CD → E 적용
   BC⁺ = {B, C, D, E}

3단계: E → A 적용
   BC⁺ = {B, C, D, E, A}

결과: BC⁺ = R → BC는 후보키!
```

**CD⁺ 계산**:
```
초기: CD⁺ = {C, D}

1단계: CD → E 적용
   CD⁺ = {C, D, E}

2단계: E → A 적용
   CD⁺ = {C, D, E, A}

3단계: A → BC 적용
   CD⁺ = {C, D, E, A, B}

결과: CD⁺ = R → CD는 후보키!
```

### F⁺의 주요 원소

**슈퍼키를 포함하는 FD**:
- A → * (A는 후보키)
- E → * (E는 후보키)
- BC → * (BC는 후보키)
- CD → * (CD는 후보키)
- *는 R의 임의의 부분집합

**기타 FD**:
- B → B, B → D, B → BD
- C → C
- D → D
- BD → B, BD → D, BD → BD
- 등등...

### 후보키

**Result**: **A, BC, CD, E**

**검증**:
- A⁺ = R ✅
- BC⁺ = R ✅
- CD⁺ = R ✅
- E⁺ = R ✅
- 모두 최소성 만족 (부분집합이 슈퍼키 아님)

---

## 문제 7.7 - 정준 커버 계산

**문제**: 문제 7.6의 함수적 종속성 집합 F의 정준 커버(Canonical Cover) Fc를 계산하시오.

```
F = {
    A → BC
    CD → E
    B → D
    E → A
}
```

**답변**:

### 정준 커버 계산 알고리즘

**Step 1**: 오른쪽을 단일 속성으로
```
A → BC 분해:
  A → B
  A → C

결과:
F' = {
    A → B
    A → C
    CD → E
    B → D
    E → A
}
```

**Step 2**: 왼쪽의 불필요한 속성 제거

**CD → E 검사**:
- C 제거 가능? D⁺ (F' - {CD → E}) = {D} ≠ E ❌
- D 제거 가능? C⁺ (F' - {CD → E}) = {C} ≠ E ❌
- 둘 다 필요

**기타 FD**: 모두 왼쪽이 단일 속성 → 검사 불필요

**Step 3**: 불필요한 FD 제거

**각 FD 검사**:

1. **A → B 필요?**
   - F' - {A → B}에서 A⁺ 계산
   - A⁺ = {A, C} (A → C만 적용)
   - B ∉ A⁺ → **필요** ✅

2. **A → C 필요?**
   - F' - {A → C}에서 A⁺ 계산
   - A⁺ = {A, B, D} (A → B, B → D 적용)
   - C ∉ A⁺ → **필요** ✅

3. **CD → E 필요?**
   - 명백히 필요 ✅

4. **B → D 필요?**
   - 명백히 필요 ✅

5. **E → A 필요?**
   - 명백히 필요 ✅

### 정준 커버

**Result**:
```
Fc = {
    A → B
    A → C
    CD → E
    B → D
    E → A
}
```

또는 원래 형태로:
```
Fc = {
    A → BC
    CD → E
    B → D
    E → A
}
= F
```

**결론**: F가 이미 정준 커버! ✅

**이유**:
- 왼쪽이 모두 유일
- 왼쪽/오른쪽에 불필요한 속성 없음
- 모든 FD가 필요

---

## 💡 핵심 정리

### Armstrong's Axioms 활용

**기본 규칙** (항상 참):
1. 재귀성: β ⊆ α → α → β
2. 증대성: α → β → γα → γβ
3. 이행성: α → β, β → γ → α → γ

**유도 규칙** (증명 가능):
- 합집합: α → β, α → γ → α → βγ
- 분해: α → βγ → α → β, α → γ
- 의사이행성: α → β, γβ → δ → αγ → δ

### 속성 폐포 계산

**알고리즘**:
```
result := α
repeat
    for each β → γ in F do
        if β ⊆ result then
            result := result ∪ γ
until (result 변화 없음)
return result
```

**용도**:
1. 후보키 찾기: α⁺ = R?
2. FD 검증: α → β ⇔ β ⊆ α⁺
3. 슈퍼키 판단: α⁺ = R?

### 정준 커버

**목표**: 최소 동치 집합

**조건**:
1. 오른쪽이 단일 속성
2. 왼쪽에 불필요한 속성 없음
3. 불필요한 FD 없음

**특징**:
- 유일하지 않을 수 있음
- F⁺ = Fc⁺ (동치)

### 무손실 분해

**조건**: R₁ ∩ R₂ → R₁ 또는 R₁ ∩ R₂ → R₂

**의미**:
- 교집합이 한쪽의 슈퍼키
- 자연 조인 시 정보 손실 없음

### 실무 팁

1. **FD 찾기**: 실제 데이터로 반례 찾기
2. **폐포 계산**: 체계적으로, 누락 없이
3. **후보키**: 모든 속성 조합 시도 (최소성 확인)
4. **정준 커버**: 단계별로 하나씩 제거

---

Part 2에서는 SQL을 이용한 FD 검증, 효율적인 폐포 계산 알고리즘, BCNF와 3NF 분해를 다룹니다!
