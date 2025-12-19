---
title: "[Chess AI] 1. 프로젝트 환경 설정 및 데이터셋 생성"
date: 2024-12-19 11:50:00 +0900
categories: [Machine Learning, Chess AI]
tags: [machine-learning, neural-network, chess, supervised-learning, pytorch, project]
pin: false
math: true
mermaid: true
---

# Chess AI 프로젝트 - Phase 1: 환경 설정 및 데이터셋

석사 진학을 위한 개인 프로젝트로 **체스 AI**를 Supervised Learning 방식으로 구현하는 과정을 기록합니다.

## 📋 프로젝트 개요

### 목표
- **Neural Network를 사용한 체스 포지션 평가 모델** 학습
- **Minimax 탐색 알고리즘**과 결합하여 최적의 수를 찾는 AI 구현
- Kaggle/자체 생성 데이터셋을 활용한 Supervised Learning

### 핵심 구성요소
1. **Position Evaluation Model**: NN을 사용하여 보드 상태 평가
2. **Move Search Engine**: 평가 함수를 사용하여 최적의 수 탐색
3. **Chess Game Interface**: 테스트를 위한 게임 인터페이스

```mermaid
graph LR
    A[Chess Position<br/>FEN String] --> B[Neural Network<br/>Evaluation Model]
    B --> C[Position Score<br/>Centipawns]
    C --> D[Minimax Search<br/>with Alpha-Beta]
    D --> E[Best Move<br/>UCI Format]

    style A fill:#ffcccc
    style C fill:#ccffcc
    style E fill:#ccccff
```

---

## Step 1.1: 개발 환경 설정

### 시스템 요구사항 확인

```bash
python --version
# Python 3.12.1 ✓
```

### 필수 라이브러리 설치

```bash
pip install numpy pandas torch torchvision python-chess jupyter matplotlib seaborn scikit-learn
```

**설치된 주요 라이브러리**:
- **PyTorch 2.9.1+cpu**: 딥러닝 프레임워크
- **Python-chess 1.11.2**: 체스 규칙 엔진
- **NumPy 2.2.3**: 수치 연산
- **Pandas 2.3.1**: 데이터 처리
- **Matplotlib & Seaborn**: 시각화

### 프로젝트 디렉토리 구조

```
chess_ai/
├── data/              # 데이터셋
├── models/            # 학습된 모델
├── src/
│   ├── data_processing/  # 데이터 전처리
│   ├── model/            # 모델 정의
│   ├── engine/           # 체스 엔진
│   └── evaluation/       # 평가
├── notebooks/         # Jupyter notebooks
└── tests/             # 테스트 코드
```

### python-chess 라이브러리 테스트

```python
import chess

# 체스판 생성
board = chess.Board()
print(board)
```

**출력**:
```
r n b q k b n r
p p p p p p p p
. . . . . . . .
. . . . . . . .
. . . . . . . .
. . . . . . . .
P P P P P P P P
R N B Q K B N R
```

**테스트 결과**:
- ✅ 보드 생성 및 표시
- ✅ FEN 표기법 변환
- ✅ 합법 수 생성 (시작 위치: 20개)
- ✅ 수 실행 및 보드 업데이트

---

## Step 1.2: 데이터셋 획득 및 이해

### 데이터셋 전략

**선택한 방식**: 자체 데이터셋 생성

**이유**:
1. **완전한 통제**: 데이터 품질 및 분포 제어 가능
2. **균형잡힌 데이터셋**: 다양한 평가 범위 커버
3. **빠른 시작**: 외부 의존성 없이 즉시 시작 가능

### 데이터셋 생성 방법

```mermaid
graph TD
    A[랜덤 게임 플레이] --> B[체스 포지션 수집]
    B --> C[Simple Evaluation<br/>Material + Mobility]
    C --> D[FEN + Evaluation<br/>데이터셋]
    D --> E[CSV 파일 저장]

    style A fill:#ffe6e6
    style C fill:#e6f3ff
    style E fill:#e6ffe6
```

### 평가 함수 (Simple Evaluation)

현재는 간단한 Material-based 평가 사용:

```python
def simple_evaluation(board):
    """
    Material-based evaluation

    Piece values:
    - Pawn: 100
    - Knight: 320
    - Bishop: 330
    - Rook: 500
    - Queen: 900
    """
    piece_values = {
        chess.PAWN: 100,
        chess.KNIGHT: 320,
        chess.BISHOP: 330,
        chess.ROOK: 500,
        chess.QUEEN: 900,
        chess.KING: 0
    }

    score = 0
    for square in chess.SQUARES:
        piece = board.piece_at(square)
        if piece:
            value = piece_values[piece.piece_type]
            if piece.color == chess.WHITE:
                score += value
            else:
                score -= value

    # Add mobility bonus
    mobility_bonus = (white_mobility - black_mobility) * 2

    return score + mobility_bonus
```

> **참고**: 프로덕션 환경에서는 Stockfish 평가로 대체 예정

### 데이터셋 생성 실행

```bash
cd src/data_processing
python create_sample_dataset.py
```

**생성 결과**:
```
Playing 1000 random games...
Total unique positions: 56,874

Evaluation statistics (centipawns):
  count: 56,874
  mean: -14.13
  std: 588.23
  min: -2,978
  max: +2,900
```

---

## 📊 데이터셋 분석 (EDA)

### 기본 통계

| Metric | Value |
|--------|-------|
| **총 포지션 수** | 56,874 |
| **메모리 사용량** | 6.50 MB |
| **결측값** | 0 |
| **평균 평가** | -14.13 centipawns |
| **표준편차** | 588.23 |

### 평가 분포

```python
# 평가 범위별 분류
bins = [-inf, -1000, -500, -200, -50, 50, 200, 500, 1000, inf]
```

| 평가 범위 | 포지션 수 | 비율 |
|----------|----------|------|
| 매우 불리 (less than -1000) | 3,334 | 5.86% |
| 불리 (-1000 to -500) | 5,999 | 10.55% |
| 약간 불리 (-500 to -200) | 7,068 | 12.43% |
| 작은 불리 (-200 to -50) | 4,888 | 8.59% |
| **균형 (-50 to 50)** | **15,129** | **26.60%** |
| 작은 유리 (50 to 200) | 5,610 | 9.86% |
| 유리 (200 to 500) | 6,481 | 11.40% |
| 매우 유리 (500 to 1000) | 5,401 | 9.50% |
| 압도적 유리 (greater than 1000) | 2,964 | 5.21% |

### 데이터 품질 검증

```python
# FEN 유효성 검증
sample_size = 100
invalid_count = 0

for fen in sample_fens:
    board = chess.Board(fen)
    if not board.is_valid():
        invalid_count += 1

print(f"Valid positions: {sample_size - invalid_count}/{sample_size}")
```

**결과**: ✅ 100/100 유효 (100%)

### 데이터 시각화

![Dataset Analysis](/assets/img/chess-ai/step1-dataset-analysis.png)
_데이터셋 분석 결과: 평가 분포, Box Plot, 카테고리별 분포, 누적 분포_

### 샘플 포지션 예시

**매우 불리한 포지션 (-948 centipawns)**:
```
r . . . . b n .
. b . p . . k r
. q n . p p . .
p . p . . . . .
. P P P P . . N
. . . . . . P P
P . . . B P . .
R N . . K . . R
```

**균형 포지션 (0 centipawns)**:
```
r n . q k b n r
p . . p p . p .
b . . . . . . .
. . p . . p . .
. p B P . . . p
. P P . P Q P .
P . . . . P . P
R N B . K . N R
```

**매우 유리한 포지션 (+830 centipawns)**:
```
. n . . k . n .
. b . . p . . .
. p q . . . . r
. N . p . p p p
. P p P . . . .
B . . . K P . .
. . . P P . P P
. . R Q . B N R
```

---

## 🎯 주요 성과

### ✅ 완료된 작업
1. **개발 환경 구축**
   - Python 3.12.1 설치
   - PyTorch, python-chess 등 필수 라이브러리 설치
   - 프로젝트 디렉토리 구조 생성

2. **데이터셋 생성**
   - 56,874개 유니크 체스 포지션 생성
   - Material-based 평가 함수 구현
   - CSV 형식으로 저장

3. **데이터 분석**
   - 기본 통계 및 분포 확인
   - 데이터 품질 검증 (100% 유효)
   - 시각화 생성

### 📈 데이터셋 특징
- ✅ 균형잡힌 평가 분포 (균형 포지션 26.60%)
- ✅ 넓은 평가 범위 커버 (-2,978 ~ +2,900)
- ✅ 100% 유효한 FEN 문자열
- ✅ 결측값 없음

---

## 🔜 다음 단계: Phase 2 - Data Preprocessing

### Step 2.1: Position Representation Design
- **FEN → Tensor 변환** 구현
- 8×8×13 텐서 형식 (12 piece types + meta info)
- Edge case 처리

### Step 2.2: Evaluation Score Normalization
- Centipawns → [-1, 1] 정규화
- tanh/sigmoid 변환 전략
- Checkmate 스코어 처리

### Step 2.3: Dataset Split & Preparation
- Train/Validation/Test 분할 (70/15/15)
- DataLoader 구현
- 데이터 캐싱

---

## 📚 참고 자료

- [Python-chess Documentation](https://python-chess.readthedocs.io/)
- [PyTorch Documentation](https://pytorch.org/docs/)
- [Lichess Database](https://database.lichess.org/)
- [Chess Programming Wiki](https://www.chessprogramming.org/)

---

## 💡 배운 점

1. **데이터셋 생성의 중요성**
   - 직접 생성함으로써 데이터 품질 완전 제어
   - 프로젝트 요구사항에 맞춘 데이터 분포 조정 가능

2. **Python-chess 라이브러리 활용**
   - 체스 규칙 구현 없이 바로 AI 개발 집중 가능
   - FEN 표기법과 UCI 이동 표기 자동 변환

3. **Simple Evaluation의 한계**
   - Material만으로는 tactical 포지션 평가 부족
   - 향후 Stockfish 평가나 더 정교한 heuristic 필요

---

**다음 포스트**: Phase 2 - 데이터 전처리 및 모델 아키텍처 설계
