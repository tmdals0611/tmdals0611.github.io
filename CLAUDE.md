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

#### 계획된 포스트 시리즈
1. **데이터베이스 기초**
   - 데이터베이스 개념과 발전사
   - 데이터베이스 시스템 아키텍처
   - 데이터 모델링 기초

2. **관계형 데이터베이스**
   - 관계형 모델의 이해
   - 관계 대수와 관계 해석
   - SQL 기초와 고급 쿼리

3. **데이터베이스 설계**
   - ER 모델과 ER 다이어그램
   - 정규화 이론과 실습
   - 스키마 설계 실무

4. **고급 데이터베이스 개념**
   - 트랜잭션과 동시성 제어
   - 인덱싱과 쿼리 최적화
   - 저장 시스템과 파일 구조

5. **현대 데이터베이스 기술**
   - NoSQL 데이터베이스
   - 분산 데이터베이스
   - 빅데이터와 데이터 웨어하우스

#### 포스트 작성 원칙
- **이론과 실습의 균형**: 개념 설명 + 실제 SQL 예제
- **시각적 설명**: ER 다이어그램, 스키마 구조도 등 적극 활용
- **단계별 학습**: 기초부터 고급까지 점진적 난이도 증가
- **실무 연계**: 이론을 실제 프로젝트에 어떻게 적용할지 고민

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

## 프로젝트 상태

**최종 업데이트**: 2025-01-09  
**현재 상태**: main 브랜치에서 기본 설정 완료, 데이터베이스 포스트 시리즈 준비 중  
**배포 상태**: https://tmdals0611.github.io 에서 확인 가능