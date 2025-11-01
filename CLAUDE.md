# GitHub Pages 블로그 프로젝트 문서

## 프로젝트 개요

**프로젝트 유형**: Jekyll Chirpy 테마를 사용한 GitHub Pages 블로그  
**저장소**: tmdals0611.github.io  
**테마**: jekyll-theme-chirpy v7.2.4  
**라이선스**: MIT  
**상태**: 샘플 포스트와 함께 초기 설정 완료

## 블로그 목적 및 컨셉

### 🎯 **주요 목적**
- **CS 강의 요약**: 대학교에서 배운 컴퓨터 과학 강의 내용 정리 및 공유
- **프로젝트 포트폴리오**: 개인 프로젝트 소개 및 개발 과정 문서화
- **학습 기록**: 개발 과정에서 배운 내용과 경험 아카이브

### 📚 **예상 콘텐츠 카테고리**
- **Computer Science**: 자료구조, 알고리즘, 운영체제, 네트워크, 데이터베이스 등
- **Projects**: 개인/팀 프로젝트 소개, 기술 스택, 구현 과정, 회고
- **Development**: 개발 팁, 도구 사용법, 트러블슈팅 경험
- **Study Notes**: 강의 노트, 논문 리뷰, 기술 동향 정리

### 🎨 **컨셉 및 톤앤매너**
- **학술적이면서도 접근하기 쉬운** 글쓰기 스타일
- **코드와 다이어그램**을 활용한 시각적 설명
- **실무 경험과 이론의 연결**을 통한 실용적인 내용
- **깔끔하고 체계적인** 정보 구조  

## 아키텍처 및 기술 스택

### 핵심 기술
- **Jekyll**: 정적 사이트 생성기
- **Ruby**: 런타임 환경
- **JavaScript/CSS**: 프론트엔드 자산
- **GitHub Pages**: 호스팅 플랫폼
- **PWA**: 프로그레시브 웹 앱 지원

### 테마 기능
✅ **다크/라이트 테마 토글**  
✅ **반응형 디자인**  
✅ **목차 (TOC)**  
✅ **구문 강조**  
✅ **수학 표현식**  
✅ **Mermaid 다이어그램**  
✅ **검색 기능**  
✅ **댓글 시스템** (Disqus, Utterances, Giscus)  
✅ **SEO 최적화**  
✅ **분석 도구 통합**  
✅ **PWA 지원**  

## 프로젝트 구조

```
📁 루트 디렉토리
├── 📁 _config.yml              # 메인 설정 파일
├── 📁 _data/                   # 데이터 파일
│   ├── authors.yml             # 작성자 정보
│   ├── contact.yml             # 연락처 방법
│   ├── 📁 locales/            # 언어 파일 (23개 이상 언어)
│   └── share.yml              # 소셜 공유
├── 📁 _includes/              # 템플릿 부분
│   ├── 📁 analytics/          # 분석 도구 제공자
│   ├── 📁 comments/           # 댓글 시스템
│   └── 📁 embed/              # 미디어 임베드
├── 📁 _layouts/               # 페이지 레이아웃
├── 📁 _posts/                 # 블로그 포스트 (마크다운)
├── 📁 _sass/                  # 스타일시트
├── 📁 _tabs/                  # 정적 페이지
├── 📁 assets/                 # 정적 자산
└── 📁 tools/                  # 빌드 스크립트
```

## 설정 상태

### ✅ **완료된 설정**
- **언어**: 한국어 (ko-KR)
- **시간대**: Asia/Shanghai
- **사이트 제목**: "CS Study & Project Archive"
- **부제목**: "컴퓨터 과학 학습 기록 및 프로젝트 아카이브"
- **URL**: https://tmdals0611.github.io
- **GitHub 사용자명**: tmdals0611
- **CDN**: 활성화 (chirpy-img.netlify.app)
- **PWA**: 활성화
- **페이지네이션**: 페이지당 10개 포스트
- **목차**: 전역 활성화

## 개발 워크플로우

### 빌드 명령어
```bash
# 의존성 설치
bundle install
npm install

# 개발 서버 실행
bundle exec jekyll serve

# 프로덕션 빌드
npm run build

# SCSS 린트 검사
npm run lint:scss
```

### 콘텐츠 관리

#### 포스트 작성
```bash
# 위치: _posts/
# 형식: YYYY-MM-DD-제목.md
# 예시: 2024-01-01-첫번째-포스트.md
```

#### 포스트 Front Matter 템플릿
```yaml
---
title: "포스트 제목"
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [카테고리1, 카테고리2]
tags: [태그1, 태그2]
pin: true          # 상단 고정
math: true         # 수학 표현식 활성화
mermaid: true      # 다이어그램 활성화
image: 
  path: /path/to/image.jpg
  alt: 대체 텍스트
---
```

## 배포 전략

### GitHub Pages 통합
1. **자동 배포** - GitHub Actions 통해
2. **커스텀 도메인** 지원 가능
3. **SSL/HTTPS** 기본 활성화
4. **CDN** 캐싱 - Netlify 통해

### SEO 설정
- **메타 태그** 자동화
- **사이트맵** 생성
- **RSS 피드** 제공
- **소셜 미디어** 미리보기 카드

## 분석 및 모니터링 옵션

**지원 제공자**:
- Google Analytics
- GoatCounter  
- Umami
- Matomo
- Cloudflare Web Analytics
- Fathom

## 댓글 시스템 옵션

**사용 가능한 제공자**:
- **Disqus**: 전통적인 댓글 시스템
- **Utterances**: GitHub Issues 기반
- **Giscus**: GitHub Discussions 기반

## 다국어 지원

**지원 언어**: 23개 이상 포함:
- 영어 (en)
- 한국어 (ko-KR) 
- 중국어 (zh-CN, zh-TW)
- 일본어 (ja)
- 스페인어 (es-ES)
- 독일어 (de-DE)
- 기타 다수...

## Git 브랜치 관리

### 🌿 **브랜치 구조**
- **main**: 기본 브랜치 (GitHub Pages 배포용)
- **master**: 이전 작업 브랜치 (현재 사용 안 함)

### 📝 **배포 과정**
1. `main` 브랜치에서 작업
2. 변경사항 커밋
3. `git push origin main`으로 배포
4. GitHub Actions 자동 빌드 및 배포

## 즉시 수행할 작업 목록

### ✅ **완료된 작업**
1. **_config.yml 업데이트**:
   - ✅ 사이트 제목과 부제목 설정
   - ✅ GitHub 사용자명 구성
   - ✅ 사이트 URL 설정
   - ✅ 개인 정보 추가

2. **콘텐츠 정리**:
   - ✅ About 페이지 업데이트
   - ✅ 프로젝트 문서화 (CLAUDE.md)

### 📋 **다음 단계**
1. **콘텐츠 생성**:
   - 첫 번째 오리지널 블로그 포스트 작성
   - 카테고리/태그 전략 구성
   - 콘텐츠 캘린더 계획

2. **기능 설정**:
   - 분석 도구 선택 및 설정
   - 댓글 시스템 구성
   - 소셜 미디어 링크 설정

3. **브랜딩 커스터마이징**:
   - 아바타 이미지 교체
   - 파비콘 세트 업데이트

## 자주 사용하는 명령어 참조

```bash
# 로컬 개발
bundle exec jekyll serve --drafts --livereload

# 프로덕션 빌드
JEKYLL_ENV=production bundle exec jekyll build

# HTML 테스트
bundle exec htmlproofer ./_site

# 의존성 업데이트
bundle update
npm update

# 테마 버전 확인
bundle info jekyll-theme-chirpy

# Git 작업
git add .
git commit -m "변경사항 설명"
git push origin main
```

## 문제 해결 가이드

### 일반적인 문제
1. **빌드 실패**: Ruby/Jekyll 버전 확인
2. **의존성 누락**: `bundle install` 실행
3. **CSS/JS 문제**: `npm run build` 실행
4. **GitHub Pages 배포**: Actions 탭 확인
5. **브랜치 문제**: main 브랜치에서 작업 확인

### 디버그 모드
```bash
# 상세한 빌드 출력
bundle exec jekyll build --verbose

# 설정 확인
bundle exec jekyll doctor
```

## 리소스 및 문서

- **테마 위키**: https://github.com/cotes2020/jekyll-theme-chirpy/wiki
- **Jekyll 문서**: https://jekyllrb.com/docs/
- **GitHub Pages**: https://docs.github.com/pages
- **마크다운 가이드**: https://www.markdownguide.org/

## 콘텐츠 계획

### 📚 **데이터베이스시스템및응용 강의 시리즈**

**목표**: 대학 강의 내용을 체계적으로 정리하여 학습 기록 및 복습 자료로 활용

#### ✅ 업로드 완료된 포스트 (4개)
1. **[2024-10-09] 데이터베이스 시스템 개요**
   - Data, Database, DBMS의 개념과 관계
   - 데이터 모델 (관계형, E-R, 반구조화)
   - 스키마와 인스턴스
   - 데이터베이스 언어 (DDL, DML, SQL)
   - 데이터베이스 엔진 구조

2. **[2024-10-10] 관계형 모델**
   - 관계, 튜플, 속성의 개념
   - 관계 스키마와 인스턴스
   - 키의 종류 (슈퍼키, 후보키, 기본키, 외래키)
   - 관계 대수 연산자 (Select, Project, Join 등)
   - 집합 연산 (Union, Intersection, Difference)

3. **[2024-10-11] SQL 기초**
   - DDL (CREATE TABLE, 무결성 제약조건)
   - 기본 쿼리 구조 (SELECT-FROM-WHERE)
   - 문자열 연산 (LIKE, ESCAPE)
   - 집합 연산 (UNION, INTERSECT, EXCEPT)
   - NULL 값 처리
   - 집계 함수와 GROUP BY/HAVING
   - 중첩 부질의 (IN, EXISTS, SOME, ALL)
   - 데이터 수정 (INSERT, UPDATE, DELETE)

4. **[2024-10-12] E-R 모델을 이용한 데이터베이스 설계**
   - 데이터베이스 설계 과정 (개념적/논리적/물리적 설계)
   - 개체 집합과 관계 집합
   - 복합 속성 (단순/복합, 단일값/다중값, 유도 속성)
   - 매핑 카디널리티 제약조건 (1:1, 1:N, N:1, M:N)
   - 참여 제약조건 (전체/부분 참여)
   - 약한 개체 집합
   - E-R 다이어그램의 관계형 스키마 변환
   - 특수화/일반화, 집약

#### 📝 다음 예정 포스트 (PDF 기반)

**5. 관계형 데이터베이스 설계 (정규화) - Part 1**
   - 출처: `7.Relational_Database_Design-1.pdf`
   - 예상 내용:
     - 함수적 종속성 (Functional Dependencies)
     - 정규형의 개념
     - 제1정규형 (1NF)
     - 제2정규형 (2NF)
     - 제3정규형 (3NF)

**6. 관계형 데이터베이스 설계 (정규화) - Part 2**
   - 출처: `7.Relational_Database_Design-2.pdf`
   - 예상 내용:
     - 보이스-코드 정규형 (BCNF)
     - 정규화 과정과 알고리즘
     - 무손실 분해
     - 함수적 종속성 보존
     - 비정규화와 실무 적용

**7. 데이터 저장 구조 (Data Storage Structures)**
   - 출처: `13.Data_Storage_Structures.pdf`
   - 예상 내용:
     - 물리적 저장 매체 (디스크, SSD, 메모리)
     - 파일 구조와 레코드 구조
     - 버퍼 관리
     - 페이지와 블록 구조
     - 저장 공간 할당

#### 향후 계획된 주제
7. **고급 SQL 기능**
   - 조인의 다양한 형태
   - 뷰(Views)와 인덱스(Indexes)
   - 저장 프로시저와 함수
   - 트랜잭션과 동시성 제어

8. **데이터베이스 저장 및 인덱싱**
   - 저장 구조와 파일 구조
   - 인덱싱 기법 (B-트리, 해시)
   - 쿼리 최적화

9. **트랜잭션 관리**
   - ACID 특성
   - 동시성 제어
   - 회복 시스템

10. **현대 데이터베이스 기술**
    - NoSQL 데이터베이스
    - 분산 데이터베이스
    - 빅데이터와 데이터 웨어하우스

#### 포스트 작성 원칙
- **이론과 실습의 균형**: 개념 설명 + 실제 SQL 예제
- **시각적 설명**: ER 다이어그램, 스키마 구조도 등 적극 활용
- **단계별 학습**: 기초부터 고급까지 점진적 난이도 증가
- **실무 연계**: 이론을 실제 프로젝트에 어떻게 적용할지 고민
- **연속성 유지**: 각 포스트는 이전 내용을 기반으로 진행

### 📝 **데이터베이스 문제 해설 시리즈**

**목표**: 강의 내용을 바탕으로 한 예시 문제와 상세 해설을 제공하여 학습 내용 확인 및 심화 학습 지원

#### 문제 해설 포스트 목록 (PDF 기반)

**1. 문제 해설 #1 - 데이터베이스 기초**
   - 출처: `1.pdf`
   - 연관 강의: 데이터베이스 시스템 개요
   - 예상 내용:
     - 데이터, 데이터베이스, DBMS 개념 확인 문제
     - 데이터 모델 비교 문제
     - 스키마와 인스턴스 구분 문제
     - 데이터베이스 언어 (DDL, DML) 관련 문제

**2. 문제 해설 #2 - 관계형 모델**
   - 출처: `2.pdf`
   - 연관 강의: 관계형 모델
   - 예상 내용:
     - 관계, 튜플, 속성 개념 문제
     - 키 식별 및 설정 문제
     - 관계 대수 연산 문제
     - 실제 데이터에서 슈퍼키/후보키 찾기

**3. 문제 해설 #3 - SQL 기초**
   - 출처: `3.pdf`
   - 연관 강의: SQL 기초
   - 예상 내용:
     - DDL 작성 문제
     - SELECT 쿼리 작성 문제
     - 집계 함수와 GROUP BY 활용 문제
     - 중첩 부질의 작성 문제
     - 실제 시나리오 기반 SQL 작성

**4. 문제 해설 #6 - E-R 모델**
   - 출처: `6.pdf`
   - 연관 강의: E-R 모델을 이용한 데이터베이스 설계
   - 예상 내용:
     - E-R 다이어그램 작성 문제
     - 카디널리티 결정 문제
     - 약한 개체 집합 식별 문제
     - E-R 모델의 관계형 스키마 변환 문제
     - 실제 시나리오 기반 데이터베이스 설계

**5. 문제 해설 #7 - 정규화**
   - 출처: `7.pdf`
   - 연관 강의: 관계형 데이터베이스 설계와 정규화
   - 예상 내용:
     - 함수적 종속성 찾기 문제
     - 정규형 판단 문제 (1NF, 2NF, 3NF, BCNF)
     - 정규화 과정 수행 문제
     - 무손실 분해 증명 문제
     - 실제 테이블 정규화 연습

**6. 문제 해설 #13 - 데이터 저장 구조**
   - 출처: `13.pdf`
   - 연관 강의: 데이터 저장 구조
   - 예상 내용:
     - 저장 매체 특성 비교 문제
     - 파일 구조와 레코드 구조 설계 문제
     - 버퍼 관리 전략 문제
     - 인덱스 구조 이해 문제
     - 저장 공간 계산 문제

#### 문제 해설 포스트 작성 원칙
- **문제 제시**: 원본 문제를 명확하게 제시
- **단계별 풀이**: 문제 해결 과정을 단계별로 설명
- **핵심 개념 연결**: 해당 문제가 다루는 강의 내용과 연결
- **오답 노트**: 흔히 하는 실수나 헷갈리는 부분 설명
- **확장 문제**: 유사한 문제나 응용 문제 제시
- **실무 관련성**: 해당 개념이 실무에서 어떻게 활용되는지 설명

#### 문제 해설 포스트 Front Matter 템플릿
```yaml
---
title: "[데이터베이스] 문제 해설 #1 - 데이터베이스 기초"
date: 2025-01-XX 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [database-basics, problem-solving, exercise, theory]
pin: false
math: true
mermaid: true
---
```

## 카테고리 및 태그 전략

### 📂 **카테고리 구조**
```
Computer Science/
├── Database Systems          # 데이터베이스 관련 모든 포스트
├── Algorithms               # 알고리즘 및 자료구조
├── Operating Systems        # 운영체제
├── Networks                # 네트워크 및 통신
├── Software Engineering    # 소프트웨어 공학
└── Programming Languages   # 프로그래밍 언어별 학습

Projects/
├── Web Development         # 웹 개발 프로젝트
├── Mobile Apps            # 모바일 앱 개발
├── Data Science           # 데이터 사이언스 프로젝트
├── System Design          # 시스템 설계 프로젝트
└── Open Source           # 오픈소스 기여

Development/
├── Tools & Environment    # 개발 도구 및 환경 설정
├── Best Practices        # 코딩 베스트 프랙티스
├── Troubleshooting       # 문제 해결 경험
└── Tech Trends          # 기술 동향 및 리뷰
```

### 🏷️ **태그 전략**

#### 데이터베이스 시리즈 태그
```yaml
기초 개념: [database-basics, data-modeling, dbms]
관계형 DB: [relational-model, sql, relational-algebra]
설계: [er-diagram, normalization, schema-design]
고급 개념: [transaction, indexing, query-optimization]
실무: [database-practice, sql-examples, performance-tuning]
```

#### 학습 단계별 태그
```yaml
난이도: [beginner, intermediate, advanced]
학습 유형: [theory, practice, hands-on, review]
과목: [database-systems, algorithms, os, networks]
```

#### 프로젝트 태그
```yaml
기술 스택: [python, javascript, react, node.js, mysql, postgresql]
프로젝트 유형: [web-app, mobile-app, api, full-stack]
개발 단계: [planning, development, deployment, maintenance]
```

### 📝 **포스트 Front Matter 템플릿**

#### 강의 정리 포스트
```yaml
---
title: "[데이터베이스] 1. 데이터베이스 기초 개념"
date: 2025-01-09 14:30:00 +0900
categories: [Computer Science, Database Systems]
tags: [database-basics, data-modeling, dbms, theory]
pin: false
math: true
mermaid: true
image: 
  path: /assets/img/database/db-concepts.png
  alt: 데이터베이스 기초 개념 다이어그램
---
```

#### 프로젝트 포스트
```yaml
---
title: "[프로젝트] 도서관 관리 시스템 - 데이터베이스 설계"
date: 2025-01-09 16:00:00 +0900
categories: [Projects, Web Development]
tags: [database-practice, er-diagram, schema-design, mysql, hands-on]
pin: false
math: false
mermaid: true
image: 
  path: /assets/img/projects/library-system.png
  alt: 도서관 관리 시스템 ER 다이어그램
---
```

### 🔧 **컴파일러설계 강의 시리즈**

**목표**: 컴파일러의 동작 원리와 구조를 체계적으로 학습하여 프로그래밍 언어의 깊은 이해와 실무 응용 능력 확보

#### PDF 자료 목록

사용 가능한 PDF:
1. `01. Instruction Set Architecture.pdf` - ISA 기초
2. `02. Lexical Analysis.pdf` - 어휘 분석
3. `03. Syntax Analysis - 1.pdf` - 구문 분석 1부
4. `04. Syntax Analysis - 2.pdf` - 구문 분석 2부
5. `05. Semantic Analysis.pdf` - 의미 분석

#### 예상 포스트 시리즈

**1. [컴파일러] Instruction Set Architecture**
   - 출처: `01. Instruction Set Architecture.pdf`
   - 예상 내용:
     - ISA의 개념과 중요성
     - RISC vs CISC 아키텍처
     - 레지스터와 메모리 구조
     - 명령어 형식과 주소 지정 모드
     - 어셈블리 언어 기초

**2. [컴파일러] 어휘 분석 (Lexical Analysis)**
   - 출처: `02. Lexical Analysis.pdf`
   - 예상 내용:
     - 컴파일러 구조 개요
     - 토큰(Token)과 렉심(Lexeme)
     - 정규 표현식 (Regular Expression)
     - 유한 오토마타 (Finite Automata)
     - Lex/Flex를 이용한 어휘 분석기 구현

**3. [컴파일러] 구문 분석 Part 1 (Syntax Analysis - 1)**
   - 출처: `03. Syntax Analysis - 1.pdf`
   - 예상 내용:
     - 문맥 자유 문법 (Context-Free Grammar)
     - 파스 트리(Parse Tree)와 구문 분석 트리
     - 좌파생과 우파생
     - 모호한 문법 (Ambiguous Grammar)
     - Top-Down 파싱 기법
     - 재귀 하강 파서 (Recursive Descent Parser)

**4. [컴파일러] 구문 분석 Part 2 (Syntax Analysis - 2)**
   - 출처: `04. Syntax Analysis - 2.pdf`
   - 예상 내용:
     - LL(1) 파서
     - FIRST와 FOLLOW 집합
     - Bottom-Up 파싱 기법
     - LR 파서 (LR(0), SLR, LR(1))
     - LALR 파서
     - Yacc/Bison을 이용한 파서 생성

**5. [컴파일러] 의미 분석 (Semantic Analysis)**
   - 출처: `05. Semantic Analysis.pdf`
   - 예상 내용:
     - 의미 분석의 역할
     - 기호 테이블 (Symbol Table)
     - 타입 검사 (Type Checking)
     - 스코프 규칙 (Scope Rules)
     - 중간 코드 생성
     - 추상 구문 트리 (Abstract Syntax Tree)

#### 향후 확장 가능 주제

6. **코드 생성 (Code Generation)**
   - 중간 코드 표현 (Three-Address Code)
   - 기본 블록과 흐름 그래프
   - 타겟 코드 생성
   - 명령어 선택

7. **코드 최적화 (Code Optimization)**
   - 지역 최적화 (Local Optimization)
   - 전역 최적화 (Global Optimization)
   - 레지스터 할당
   - 루프 최적화

8. **실습 프로젝트**
   - 간단한 언어의 컴파일러 구현
   - Lex & Yacc 활용 프로젝트
   - LLVM 프레임워크 소개

#### 포스트 작성 원칙

- **이론과 실습의 결합**: 개념 설명 + 실제 코드 예제 (C/C++, 어셈블리)
- **시각적 설명**: 상태 다이어그램, 파스 트리, 문법 다이어그램 등 적극 활용
- **단계별 학습**: 컴파일러 프론트엔드부터 백엔드까지 순차적 진행
- **도구 활용**: Lex/Flex, Yacc/Bison 등 실무 도구 소개
- **실무 연계**: 실제 프로그래밍 언어의 컴파일러 구현 사례

#### 컴파일러 시리즈 태그

```yaml
기초 개념: [compiler-basics, isa, assembly]
프론트엔드: [lexical-analysis, syntax-analysis, semantic-analysis, parsing]
도구: [lex, flex, yacc, bison, llvm]
이론: [automata, grammar, cfg, regular-expression]
실무: [compiler-implementation, language-design]
```

#### 컴파일러 포스트 Front Matter 템플릿

```yaml
---
title: "[컴파일러] 1. Instruction Set Architecture"
date: 2025-02-XX 14:30:00 +0900
categories: [Computer Science, Compiler Design]
tags: [compiler-basics, isa, assembly, risc, cisc, theory]
pin: false
math: true
mermaid: true
---
```

#### ✅ 작성 완료된 포스트 (5개)

1. **[2025-02-06] Instruction Set Architecture (ISA)** (1,014줄)
   - 컴파일러 vs 인터프리터 비교
   - ISA 정의 및 중요성
   - RISC vs CISC 비교 (ARM vs x86)
   - 레지스터 및 메모리 계층
   - 명령어 형식 및 주소 지정 방식
   - ARM/x86 어셈블리 기초

2. **[2025-02-07] 어휘 분석 (Lexical Analysis)** (1,013줄)
   - 컴파일러 구조 개요
   - 토큰, 렉셈, 패턴 개념
   - 정규 표현식 및 유한 오토마타
   - Lex/Flex 도구 사용법
   - 실제 렉서 구현

3. **[2025-02-08] 구문 분석 Part 1 (Top-Down Parsing)** (763줄)
   - 문맥 자유 문법(CFG)
   - 유도 및 파스 트리
   - 문법의 모호성과 제거
   - 재귀 하강 파서 구현
   - 좌재귀 제거 및 FIRST/FOLLOW 집합

4. **[2025-02-09] 구문 분석 Part 2 (LL & LR Parsing)** (805줄)
   - LL(1) 파서 및 파싱 테이블
   - 상향식 구문 분석
   - LR 파서 종류 (LR(0), SLR, LR(1), LALR)
   - Yacc/Bison 사용법
   - LL vs LR 비교

5. **[2025-02-10] 의미 분석 (Semantic Analysis)** (1,008줄)
   - 심볼 테이블 설계 및 구현
   - 타입 체킹 시스템
   - 추상 구문 트리(AST)
   - 스코프 규칙
   - 중간 코드 생성 (3주소 코드)

### 🔨 **소프트웨어공학 강의 시리즈**

**목표**: 소프트웨어 개발 생명주기 전반을 체계적으로 학습하여 실무 프로젝트 수행 능력 확보

#### PDF 자료 목록

사용 가능한 PDF (18개):
1. `01_Introduction.pdf` - 소프트웨어공학 개론
2. `02_Software_Processes.pdf` - 소프트웨어 프로세스
3. `03_Feasibility_Study.pdf` - 타당성 조사
4. `04_Software_Requirements.pdf` - 소프트웨어 요구사항
5. `05_Requirements_Engineering.pdf` - 요구공학
6. `06_System_Models.pdf` - 시스템 모델링
7. `07_Introduction_to_UML_1.pdf` - UML 입문 1부
8. `08_Introduction_to_UML_2.pdf` - UML 입문 2부
9. `09_Software_Architecture.pdf` - 소프트웨어 아키텍처
10. `10_1_Component_Level_Design.pdf` - 컴포넌트 수준 설계
11. `10_2_Object_Oriented_Design_Principles.pdf` - 객체지향 설계 원칙
12. `11_Usability_and_User_Interface_Design.pdf` - 사용성과 UI 설계
13. `12_Project_Management.pdf` - 프로젝트 관리
14. `13_Process_Improvement_CMMI.pdf` - 프로세스 개선 및 CMMI
15. `14_1_Agile_Software_Development.pdf` - 애자일 소프트웨어 개발
16. `14_2_Scrum.pdf` - 스크럼
17. `15_People_Management.pdf` - 인력 관리
18. `16_Implementation.pdf` - 구현
19. `17_1_SW_Construction_1-1-Creating_High_Quality_Code.pdf` - 고품질 코드 작성 1-1
20. `17_2_SW_Construction_1-2-Creating_High_Quality_Code.pdf` - 고품질 코드 작성 1-2

#### 예상 포스트 시리즈

**Part 1: 소프트웨어공학 기초**

**1. [소프트웨어공학] Introduction**
   - 출처: `01_Introduction.pdf`
   - 예상 내용:
     - 소프트웨어공학의 정의와 필요성
     - 소프트웨어의 특성과 위기
     - 소프트웨어 품질 속성
     - 전문적 책임과 윤리

**2. [소프트웨어공학] Software Processes**
   - 출처: `02_Software_Processes.pdf`
   - 예상 내용:
     - 소프트웨어 프로세스 모델 (폭포수, 나선형, 증분)
     - 프로세스 활동 (명세, 설계, 검증, 진화)
     - 프로세스 개선

**3. [소프트웨어공학] Feasibility Study**
   - 출처: `03_Feasibility_Study.pdf`
   - 예상 내용:
     - 타당성 조사의 목적
     - 기술적/경제적/운영적 타당성
     - 비용-편익 분석

**Part 2: 요구공학**

**4. [소프트웨어공학] Software Requirements**
   - 출처: `04_Software_Requirements.pdf`
   - 예상 내용:
     - 기능적/비기능적 요구사항
     - 사용자 요구사항 vs 시스템 요구사항
     - 요구사항 명세서 작성

**5. [소프트웨어공학] Requirements Engineering**
   - 출처: `05_Requirements_Engineering.pdf`
   - 예상 내용:
     - 요구사항 도출 (Elicitation)
     - 요구사항 분석 및 협상
     - 요구사항 검증 및 관리

**6. [소프트웨어공학] System Models**
   - 출처: `06_System_Models.pdf`
   - 예상 내용:
     - 시스템 모델링의 목적
     - 컨텍스트 모델, 상호작용 모델
     - 구조 모델, 행위 모델

**Part 3: UML과 모델링**

**7. [소프트웨어공학] Introduction to UML Part 1**
   - 출처: `07_Introduction_to_UML_1.pdf`
   - 예상 내용:
     - UML 개요 및 다이어그램 분류
     - Use Case Diagram
     - Class Diagram
     - Sequence Diagram

**8. [소프트웨어공학] Introduction to UML Part 2**
   - 출처: `08_Introduction_to_UML_2.pdf`
   - 예상 내용:
     - Activity Diagram
     - State Machine Diagram
     - Component Diagram
     - Deployment Diagram

**Part 4: 설계**

**9. [소프트웨어공학] Software Architecture**
   - 출처: `09_Software_Architecture.pdf`
   - 예상 내용:
     - 아키텍처 패턴 (Layered, Client-Server, MVC)
     - 아키텍처 뷰 (4+1 View)
     - 아키텍처 설계 결정

**10. [소프트웨어공학] Component Level Design**
   - 출처: `10_1_Component_Level_Design.pdf`
   - 예상 내용:
     - 컴포넌트 기반 설계
     - 인터페이스 설계
     - 컴포넌트 다이어그램

**11. [소프트웨어공학] Object-Oriented Design Principles**
   - 출처: `10_2_Object_Oriented_Design_Principles.pdf`
   - 예상 내용:
     - SOLID 원칙 (SRP, OCP, LSP, ISP, DIP)
     - 디자인 패턴 (생성, 구조, 행위 패턴)
     - 객체지향 설계 기법

**12. [소프트웨어공학] Usability and UI Design**
   - 출처: `11_Usability_and_User_Interface_Design.pdf`
   - 예상 내용:
     - 사용성 원칙 (Nielsen's Heuristics)
     - UI 설계 프로세스
     - 사용자 중심 설계

**Part 5: 프로젝트 관리**

**13. [소프트웨어공학] Project Management**
   - 출처: `12_Project_Management.pdf`
   - 예상 내용:
     - 프로젝트 계획 및 일정 관리
     - 위험 관리
     - 인력 및 비용 추정 (COCOMO)

**14. [소프트웨어공학] Process Improvement & CMMI**
   - 출처: `13_Process_Improvement_CMMI.pdf`
   - 예상 내용:
     - 프로세스 성숙도 모델
     - CMMI 레벨 (Initial, Managed, Defined, Quantitatively Managed, Optimizing)
     - 프로세스 개선 전략

**Part 6: 애자일 개발**

**15. [소프트웨어공학] Agile Software Development**
   - 출처: `14_1_Agile_Software_Development.pdf`
   - 예상 내용:
     - 애자일 선언문 및 원칙
     - XP (Extreme Programming)
     - TDD (Test-Driven Development)

**16. [소프트웨어공학] Scrum**
   - 출처: `14_2_Scrum.pdf`
   - 예상 내용:
     - 스크럼 프레임워크
     - 스크럼 역할 (Product Owner, Scrum Master, Team)
     - 스크럼 이벤트 (Sprint, Daily Scrum, Review, Retrospective)
     - 스크럼 산출물 (Product Backlog, Sprint Backlog, Increment)

**Part 7: 구현 및 코드 품질**

**17. [소프트웨어공학] People Management**
   - 출처: `15_People_Management.pdf`
   - 예상 내용:
     - 팀 구성 및 관리
     - 동기부여 및 리더십
     - 조직 문화

**18. [소프트웨어공학] Implementation**
   - 출처: `16_Implementation.pdf`
   - 예상 내용:
     - 구현 전략 및 코딩 표준
     - 버전 관리
     - 빌드 자동화

**19. [소프트웨어공학] Creating High-Quality Code Part 1**
   - 출처: `17_1_SW_Construction_1-1-Creating_High_Quality_Code.pdf`
   - 예상 내용:
     - 코드 품질 기준
     - 가독성 및 유지보수성
     - 네이밍 규칙

**20. [소프트웨어공학] Creating High-Quality Code Part 2**
   - 출처: `17_2_SW_Construction_1-2-Creating_High_Quality_Code.pdf`
   - 예상 내용:
     - 코드 리팩토링
     - 코드 리뷰 프로세스
     - 정적 분석 도구

#### 포스트 작성 원칙

- **이론과 실무의 연결**: 개념 설명 + 실제 프로젝트 적용 사례
- **시각적 자료 활용**: UML 다이어그램, 프로세스 흐름도, 아키텍처 다이어그램
- **단계별 학습**: 요구사항 → 설계 → 구현 → 관리 순차적 진행
- **실무 도구 소개**: UML 도구, 프로젝트 관리 도구, 협업 도구
- **사례 연구**: 실제 프로젝트 성공/실패 사례 분석

#### 소프트웨어공학 시리즈 태그

```yaml
기초 개념: [software-engineering, software-process, quality-attributes]
요구공학: [requirements, requirements-engineering, use-case, user-story]
설계: [software-design, architecture, design-patterns, solid-principles]
모델링: [uml, system-modeling, class-diagram, sequence-diagram]
관리: [project-management, risk-management, estimation, cmmi]
애자일: [agile, scrum, xp, tdd, sprint]
구현: [implementation, coding-standards, code-quality, refactoring]
팀워크: [team-management, collaboration, leadership]
```

#### 소프트웨어공학 포스트 Front Matter 템플릿

```yaml
---
title: "[소프트웨어공학] 1. Introduction"
date: 2025-02-XX 14:30:00 +0900
categories: [Computer Science, Software Engineering]
tags: [software-engineering, software-process, quality-attributes, theory]
pin: false
math: true
mermaid: true
---
```

## 프로젝트 상태

**최종 업데이트**: 2025-02-10
**현재 상태**: 컴파일러 시리즈 완료 (5개 포스트), 소프트웨어공학 시리즈 계획 완료
**배포 상태**: https://tmdals0611.github.io 에서 확인 가능