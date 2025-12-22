---
title: "[Chess AI] 3. CNN 모델 설계 및 학습"
date: 2024-12-22 16:00:00 +0900
categories: [Machine Learning, Chess AI]
tags: [machine-learning, neural-network, chess, cnn, pytorch, deep-learning, project, cuda, gpu]
pin: false
math: true
mermaid: true
---

# Chess AI 프로젝트 - Phase 3-4: CNN 모델 설계 및 학습

Phase 2에서 데이터 전처리 파이프라인을 완성한 후, **Phase 3-4에서 CNN 모델을 설계하고 학습을 완료**했습니다.

## 📋 완료된 작업

1. ✅ **CNN Architecture**: 2.36M 파라미터 CNN 모델
2. ✅ **GPU Setup**: CUDA 11.8 + RTX 3060 설치
3. ✅ **Full Training**: 50 epochs (28 epochs early stopped)
4. ✅ **Model Evaluation**: 최종 성능 분석 완료
5. ✅ **실험**: 개선 학습 및 12M 데이터셋 분석

```mermaid
graph TD
    A[13x8x8 Input] --> B[Conv Layer 1<br/>64 filters]
    B --> C[BatchNorm + ReLU]
    C --> D[Conv Layer 2<br/>128 filters]
    D --> E[BatchNorm + ReLU]
    E --> F[Conv Layer 3<br/>128 filters]
    F --> G[BatchNorm + ReLU]
    G --> H[Flatten<br/>8192 features]
    H --> I[FC: 256]
    I --> J[ReLU + Dropout 0.3]
    J --> K[FC: 128]
    K --> L[ReLU]
    L --> M[FC: 1]
    M --> N[Tanh Output]

    style A fill:#ccffcc
    style N fill:#ffcccc
    style B fill:#cce5ff
    style D fill:#cce5ff
    style F fill:#cce5ff
```

---

## 🎯 최종 모델 성능

### Full Training Results (28 Epochs)

**학습 환경**:
- GPU: NVIDIA RTX 3060 Laptop (6GB, CUDA 11.8)
- Batch Size: 128
- Learning Rate: 0.001 → 0.0005 (epoch 24에서 감소)
- Early Stopping: Patience 10 (epoch 18에서 best)

**학습 곡선**:
```
Epoch 1:   Train 0.0367 | Val 0.0249
Epoch 5:   Train 0.0209 | Val 0.0209
Epoch 10:  Train 0.0179 | Val 0.0203
Epoch 18:  Train 0.0146 | Val 0.0187 ✅ Best
Epoch 28:  Train 0.0094 | Val 0.0190

Improvement:
  Train Loss: 74.4% reduction
  Val Loss: 24.8% reduction
```

### 최종 성능 지표

| Metric | Value (Normalized) | Value (Centipawns) | Target | Status |
|--------|-------------------|-------------------|--------|--------|
| **MAE** | 0.064 | **257.5 cp** | < 60 cp | ❌ |
| **RMSE** | 0.136 | **549.2 cp** | - | - |
| **R²** | **0.614** | - | > 0.80 | ❌ |
| **Correlation** | **0.786** | - | - | ✅ |

**해석**:
- 평균적으로 **2.5 pawn (257cp)** 오차
- 설명력 **61.4%** (R²)
- 강한 상관관계 **78.6%**

### 범위별 성능 분석

| Evaluation Range | Count | MAE (norm) | MAE (cp) | 성능 |
|------------------|-------|-----------|----------|------|
| **Equal** (±500cp) | 33,171 | 0.032 | **~128 cp** | ✅ 우수 |
| **Moderate** (500-2000cp) | 15,190 | 0.078-0.085 | **~320 cp** | ⚠️ 보통 |
| **Large** (>2000cp) | 2,175 | 0.432-0.435 | **~1,730 cp** | ❌ 나쁨 |

**관찰**:
- ✅ **Equal 포지션에서 우수한 성능** (65% 데이터, MAE 128cp)
- ⚠️ **Moderate advantage는 보통** (30% 데이터)
- ❌ **Extreme advantage는 부정확** (5% 데이터만)

**원인**: 데이터 불균형
- Equal positions: 65% (220,000개)
- Extreme positions: 4% (14,000개)
- 모델이 majority class(equal)에 편향됨

---

## 💻 GPU 가속 적용

### CUDA 설치

**하드웨어**: NVIDIA GeForce RTX 3060 Laptop GPU (6GB)

**설치 과정**:
```bash
# CPU 버전 제거
pip uninstall torch torchvision

# CUDA 11.8 버전 설치
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
```

**검증**:
```python
import torch
print(torch.cuda.is_available())  # True
print(torch.cuda.get_device_name(0))  # RTX 3060 Laptop GPU
```

### 성능 향상

| 작업 | CPU | GPU (RTX 3060) | 향상 |
|------|-----|----------------|------|
| **1 Epoch 학습** | ~5-10분 | **~30초-1분** | **5-10×** |
| **50 Epochs** | ~3-4시간 | **~30-60분** | **5-10×** |
| **추론 (position)** | ~5ms | **~0.5ms** | **10×** |

**효과**:
- 빠른 실험 반복 가능
- 하이퍼파라미터 튜닝 효율적
- 실시간 게임 플레이 가능

---

## 🔬 실험 1: 개선 학습 시도 (실패)

### 시도한 개선 방법

**목적**: 데이터 불균형 해결 및 성능 향상

**변경사항**:
1. **Weighted MSE Loss**: Extreme positions에 3배 가중치
2. **더 많은 Epochs**: 50 → 100 epochs
3. **더 큰 Batch Size**: 128 → 256
4. **더 강한 Regularization**: weight_decay 1e-5 → 1e-4

### 결과: 성능 악화 ❌

| 지표 | 원래 모델 | 개선 모델 | 변화 |
|------|----------|----------|------|
| **MAE** | 257.5 cp | **282.3 cp** | ❌ +9.6% 악화 |
| **R²** | 0.614 | **0.566** | ❌ -7.8% 악화 |
| **Correlation** | 0.786 | **0.768** | ❌ -2.3% 악화 |

**문제점**:
- Weighted loss가 **overfitting 유발**
- Train loss 0.013, Val loss 0.043 (3배 차이)
- Equal positions 성능은 유지, Extreme은 여전히 나쁨

**교훈**:
- **Weighted loss가 항상 답은 아님**
- 간단한 MSE가 더 나을 수 있음
- Data imbalance는 근본적으로 해결해야 함

→ **원래 모델로 롤백** ✅

---

## 🔬 실험 2: 12M 데이터셋 분석 (사용 안 함)

### 새로운 데이터셋 발견

- **크기**: 759MB (기존 21MB의 36배)
- **예상 포지션 수**: ~12M개 (기존 336K의 36배)

### 분포 분석 결과

| 지표 | 기존 (336K) | 신규 (12M) | 평가 |
|------|------------|-----------|------|
| **Equal (±500cp)** | 81.2% | **91.9%** | ❌ 더 불균형 |
| **Extreme (>2000cp)** | 3.6% | **1.1%** | ❌ 훨씬 더 나쁨 |
| **Balance Score** | 68.8/100 | **58.1/100** | ❌ 악화 |

**예상 결과**:
- Equal positions만 더 잘 예측
- Extreme positions는 더 못 예측
- MAE: 257cp → 300cp+ 예상
- 학습 시간: 6-8시간 (36배)

**결론**: **사용하지 않기로 결정** ❌
- 불균형이 더 심함
- 시간 낭비 가능성 높음
- 현재 데이터셋이 상대적으로 더 균형잡힘

---

## 💡 핵심 교훈

### 1. 데이터 품질 > 데이터 양

**12M 데이터셋 실험**:
- 36배 큰 데이터셋 ≠ 더 나은 성능
- **분포가 더 중요**
- 불균형하면 오히려 악화

**교훈**: 데이터셋 추가 전 반드시 분포 분석

### 2. Weighted Loss가 항상 답은 아님

**개선 학습 실험**:
- 3배 가중치 → overfitting
- 간단한 MSE가 더 나음
- 근본적 해결 필요 (data augmentation 등)

**교훈**: 복잡한 방법보다 간단한 방법 먼저

### 3. GPU는 필수

**CUDA 설치 효과**:
- 5-10배 빠른 학습
- 빠른 실험 반복
- 실시간 추론 가능

**교훈**: 초기에 GPU 환경 구축하면 시간 절약

### 4. Early Stopping은 중요

**Training 결과**:
- 50 epochs 계획 → 28 epochs 실제
- Best epoch: 18
- Overfitting 방지

**교훈**: Patience 설정으로 자동 최적화

---

## 🔜 다음 단계: Phase 5-8

### Phase 5: Chess Engine Implementation

**Phase 5.1 - Move Generation**:
- python-chess를 활용한 합법적인 수 생성
- 특수 수 처리 (캐슬링, 앙파상, 프로모션)

**Phase 5.2 - Minimax + Alpha-Beta Pruning**:
```python
def minimax_alpha_beta(board, depth, alpha, beta, maximizing):
    """Minimax search with alpha-beta pruning"""
    if depth == 0 or game_over:
        return evaluate_position(board, model)

    if maximizing:
        for move in ordered_moves:
            value = minimax(make_move, depth-1, alpha, beta, False)
            alpha = max(alpha, value)
            if beta <= alpha:
                break  # Beta cutoff
        return alpha
```

**Phase 5.3 - 평가 함수 통합**:
- CNN 모델을 Minimax에 통합
- Checkmate/stalemate 처리
- 예상 성능: Depth 3 (~0.5초), Depth 4 (~2초)

### Phase 6: Testing & Validation

**전술 퍼즐 테스트**:
- 50+ 전술 문제로 정확도 측정
- 체크메이트, 포크, 핀 등 전술 패턴

**자가 대국 테스트**:
- AI vs AI 100게임
- 평균 게임 길이, 승률 분석

### Phase 7: 속도 최적화

**목표**: depth=4를 <0.5초에

**최적화 방법**:
1. Move ordering (캡처 우선)
2. Transposition table
3. Batch evaluation
4. Iterative deepening

### Phase 8: GUI 구현

**웹 인터페이스 (Flask + chessboard.js)**:
- 드래그 앤 드롭으로 플레이
- 실시간 평가 표시
- 크로스 플랫폼

---

## 📂 프로젝트 구조

```
claude_project/
├── 📄 CLAUDE.md                    # 프로젝트 계획
├── 📄 train.py                     # 학습 스크립트
│
├── 📁 data/
│   ├── fen_analysis.csv            # 336,903 positions
│   ├── model_evaluation.png        # 평가 결과
│   └── training_history.png        # 학습 곡선
│
├── 📁 models/
│   └── chess_cnn/                  # 최종 모델
│       ├── best_model.pth          # 28MB (2.36M params)
│       ├── training_history.json   # 28 epochs 기록
│       └── config.json
│
├── 📁 src/
│   ├── data_processing/            # 전처리 (4 files)
│   ├── engine/                     # Phase 5 (예정)
│   └── model/                      # 모델 (4 files)
│
└── 📁 tests/                       # 테스트 코드
```

---

## 🎯 요약

### 완료된 작업 ✅

1. ✅ CNN 모델 설계 (2.36M params)
2. ✅ GPU 환경 구축 (CUDA 11.8, RTX 3060)
3. ✅ Full training (28 epochs, early stopped)
4. ✅ 최종 성능: MAE 257cp, R² 0.614
5. ✅ 개선 실험 (weighted loss - 실패)
6. ✅ 12M 데이터셋 분석 (사용 안 함)

### 다음 작업 🚀

1. Phase 5: Chess Engine Implementation
   - Move generation
   - Minimax + Alpha-Beta pruning
   - 평가 함수 통합
2. Phase 6: Testing & Validation
3. Phase 7: 속도 최적화
4. Phase 8: GUI 구현

**예상 완료**: 2-3주 이내

---

**다음 포스트**: Phase 5 - Chess Engine Implementation
