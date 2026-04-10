# TradingBot G5 — Omega-Prime

> **Multi-engine quantitative trading bot with LLM-assisted reasoning layer, built for Binance USDT-M Futures.**
> 물리·수학 기반 시그널 엔진 + LLM 추론 게이트를 결합한 실시간 암호화폐 선물 자동매매 시스템.

![Python](https://img.shields.io/badge/Python-3.11+-blue) ![asyncio](https://img.shields.io/badge/asyncio-native-green) ![status](https://img.shields.io/badge/status-research%20%2F%20portfolio-orange) ![exchange](https://img.shields.io/badge/exchange-Binance%20USDT--M-yellow)

---

## 📌 개요

단일 지표 의존을 벗어나, **여러 개의 독립 시그널 엔진을 퓨전**하여 트레이드 의사결정을 만듭니다. 각 엔진은 서로 다른 수학적 프레임(해밀토니안, 테일러 전개, 미분 동역학, 이벤트 호라이즌 등)으로 시장을 해석하고, 상위 레이어의 **DecisionFusion + HeartCore** 가 이를 통합해 포지션 진입 권한을 행사합니다. 그 위에 **LLM(Gemini) 기반 Harness 게이트**가 사후 검증을 수행합니다.

- **대상**: Binance USDT-M Futures (BTC/USDT 기본)
- **인터페이스**: WebSocket 스트리밍 (`fstream.binance.com`) + REST 주문 집행
- **언어/런타임**: Python 3.11+, `asyncio` 네이티브
- **성격**: **연구/포트폴리오 프로젝트**. 실거래 실행 가능한 인프라는 구현돼 있으나, 기본 진입점은 paper/dry-run 경로입니다. 실거래 여부는 `.env` 환경 변수로만 활성화.

---

## 🏗️ 아키텍처

4-레이어 분리 설계. 각 레이어는 아래 레이어에만 의존합니다.

```
┌────────────────────────────────────────────────────────────┐
│  Layer 3 — Strategy / Decision                             │
│    · strategy/heart.py        HeartCore (entry authority)  │
│    · engines/decision_fusion  Engine fusion + weighting    │
│    · engines/omega_harness    LLM (Gemini) reasoning gate  │
├────────────────────────────────────────────────────────────┤
│  Layer 2 — Market Microstructure                           │
│    · engines/trade_intensity  주문 흐름 강도                │
│    · engines/volume_surge     거래량 서지 감지               │
│    · metrics/adaptive_scaler  온라인 정규화                  │
├────────────────────────────────────────────────────────────┤
│  Layer 1 — Physics / Math Signal Engines                   │
│    · engines/hamiltonian      해밀토니안 (에너지 보존계)      │
│    · engines/taylor_predictor 테일러 전개 가격 예측          │
│    · engines/differential     미분 동역학 시그널              │
│    · engines/event_horizon    레짐 전이 탐지                 │
├────────────────────────────────────────────────────────────┤
│  Layer 0 — Inference Primitive                             │
│    · engines/adaptive_inference  온라인 베이지안 학습         │
└────────────────────────────────────────────────────────────┘
           ▲
           │  MarketSnapshot (가격/오더북/트레이드)
           │
┌────────────────────────────────────────────────────────────┐
│  Infrastructure                                             │
│    · infrastructure/stream        WebSocket 통합 스트림      │
│    · infrastructure/executor      TradeIntent 집행           │
│    · infrastructure/kill_switch   파일 기반 킬 스위치         │
│    · infrastructure/time_sync     Binance 시간 동기화         │
│    · infrastructure/live_exchange ccxt 실거래 어댑터          │
│    · controller/loops             메인 오케스트레이션         │
└────────────────────────────────────────────────────────────┘
```

**관심사 분리** (`docs/architecture.md`)
- `strategy/heart.py` — 진입 권한 단일 소스
- `infrastructure/executor.py` — `TradeIntent` 만 실행 (정책 모름)
- `controller/` — 스냅샷 빌드 + 루프 오케스트레이션
- `replay/` + `tests/` — 패리티/회귀 검증

---

## ⚙️ 핵심 설계 포인트

### 1. Multi-Engine Fusion
단일 지표 오버피팅을 방지하기 위해, 물리·수학적 접근이 다른 5+ 개의 엔진 출력을 **DecisionFusion** 이 가중 합성합니다. 각 엔진은 `EngineOutput(direction, confidence, score)` 이라는 동일 인터페이스를 반환해 교체·추가가 쉽습니다.
- `engines/decision_fusion.py`
- `strategy/models.py` — `EngineOutput`, `FusionDecision`, `TradeIntent` 데이터클래스

### 2. Adaptive Online Normalization
시장은 비정상(non-stationary) 시계열입니다. 고정 임계치 대신 **롤링 윈도우 기반 퍼센타일 정규화** 로 체제 변화에 적응.
- `metrics/adaptive_scaler.py` — `AdaptiveScaler(window=200)`
- Hamiltonian 엔진의 `sigma_ratio` 를 vol-adjusted return 과 블렌딩 (`engines/hamiltonian.py:26`)

### 3. LLM Reasoning Gate
하드코딩 규칙만으로는 극단 시장을 걸러내기 어려워, **Gemini 기반 사후 검증 게이트 (Harness)** 를 추가. `HarnessVerdict` 가 trade 를 최종 gate.
- `engines/omega_harness.py`
- `prompts/harness_prompt.md` — 7엔진 해석 프로토콜 정의
- `prompts/omega_core.md` — Omega-Prime 추론 페르소나

### 4. Safety Engineering
- **Kill Switch** — 파일 기반 (`data/KILL` 생성 시 즉시 그레이스풀 종료). `infrastructure/kill_switch.py`
- **Kelly Criterion Sizer** — 과도한 베팅 억제. `strategy/sizing.py`
- **ATR Dynamic Risk Manager** — 변동성 기반 손절/익절 스케일. `strategy/risk.py`
- **Drawdown Circuit Breaker** — Gate 0.5 에서 자동 차단. `strategy/heart.py`
- **Fee Budget / Net RR Gate** — 수수료 제외 순 risk-reward 미달 시 entry 거부
- **Paper / Dry-run 모드** — 기본 경로. `dry_run.py`, `dry_run_timed.py`

### 5. Real-time Infrastructure
- **통합 WebSocket 스트림** — depth20@100ms + aggTrade. `infrastructure/stream.py`
- **Binance 시간 동기화** — 서버 offset 보정. `infrastructure/time_sync.py`
- **지갑 동기화** — 선물/BNB 잔고 실시간 추적. `infrastructure/wallet_sync.py`
- **User Data Stream** — 주문 체결 이벤트 구독. `infrastructure/user_stream.py`

---

## 🧪 테스트

`pytest` 기반 회귀 테스트. 주요 커버리지:

| 파일 | 대상 |
|---|---|
| `tests/test_system_integrity.py` | 엔진 파이프라인 E2E + 엔진 간 계약 |
| `tests/test_heart.py` | HeartCore 진입 로직 |
| `tests/test_execution_chain.py` | TradeIntent → Executor 집행 체인 |
| `tests/test_kill_switch.py` | 킬 스위치 트리거/클리어 |
| `tests/test_bnb_refill.py` | BNB 자동 충전 로직 |
| `tests/test_replay.py` | 리플레이 하네스 패리티 |
| `tests/test_pattern_skills.py` | 패턴 스킬 북 |
| `tests/test_aie.py` | Adaptive Inference Engine |

```bash
pytest tests/ -v
```

---

## 🛠️ 기술 스택

| 영역 | 선택 | 이유 |
|---|---|---|
| Exchange | **Binance USDT-M Futures** | 유동성 + Maker rebate 구조 |
| Library | **ccxt** | 멀티 거래소 확장 여지 |
| Streaming | **websockets** (raw) | 레이턴시 최소화 |
| Math | **numpy, scipy** | 해밀토니안/테일러/signal processing |
| LLM | **google-genai (Gemini)** | 추론 게이트 |
| Async | **asyncio 네이티브** | 단일 이벤트 루프로 스트림·실행·체크 병행 |
| Test | **pytest** | 회귀 보증 |

전체 의존성: [`requirements.txt`](requirements.txt)

---

## 🚀 Quickstart

```bash
# 1. 클론 & 설정
git clone https://github.com/hibou04-ops/TradingBot_G5.git
cd TradingBot_G5
pip install -r requirements.txt

# 2. 환경 변수 (.env)
cp .env.example .env
# BINANCE_API_KEY, BINANCE_SECRET_KEY, GOOGLE_CLOUD_PROJECT 등 입력

# 3. Dry-run (실거래 없음, 스트림만 구독)
python dry_run.py

# 4. 시간제한 Dry-run
python dry_run_timed.py

# 5. 메인 봇 (paper 모드 기본)
python main.py

# 6. 테스트
pytest tests/ -v

# 7. 킬 스위치 (봇 강제 종료)
touch data/KILL
```

`.env` 미설정 시 모든 엔진은 구동되지만 주문은 나가지 않습니다. API 키가 있어도 `main.py` 내부 paper 모드가 기본이며, 실거래 활성화는 명시적 설정이 필요합니다.

---

## 📁 프로젝트 구조

```
TradingBot_G5/
├── config/              # 런타임 설정 (os.getenv 기반)
├── controller/          # 메인 루프 + 봇 오케스트레이션
├── engines/             # 시그널 엔진 (Layer 0~1)
│   ├── hamiltonian.py
│   ├── taylor_predictor.py
│   ├── differential.py
│   ├── event_horizon.py
│   ├── trade_intensity.py
│   ├── volume_surge.py
│   ├── adaptive_inference.py
│   ├── decision_fusion.py
│   └── omega_harness.py      # LLM 게이트
├── strategy/            # 전략 브레인
│   ├── heart.py              # HeartCore — entry authority
│   ├── sizing.py             # Kelly criterion
│   ├── risk.py               # ATR dynamic risk
│   ├── ledger.py             # Vector memory bank
│   ├── bandit.py             # Multi-armed bandit
│   └── models.py             # 데이터클래스
├── infrastructure/      # 거래소 / 스트림 / 안전
│   ├── stream.py             # WebSocket 통합
│   ├── executor.py           # TradeIntent 집행
│   ├── kill_switch.py        # 파일 기반 kill
│   ├── live_exchange.py      # ccxt 어댑터
│   ├── time_sync.py
│   ├── wallet_sync.py
│   ├── user_stream.py
│   └── bnb_refill.py         # BNB 자동 충전
├── metrics/             # 성능/피처/스케일러
├── replay/              # 리플레이 하네스
├── skills/              # 패턴 스킬 북
├── prompts/             # LLM 프롬프트 (harness, omega_core)
├── unified_dashboard/   # 실시간 대시보드 (HTML/JS/Python)
├── docs/                # 아키텍처 & 거버넌스 문서
├── tests/               # pytest 회귀 테스트
├── main.py              # 메인 엔트리 포인트
├── dry_run.py           # Paper-trading 진입점
└── calibration_runner.py  # 파라미터 캘리브레이션 러너
```

---

## 📝 상태 & 로드맵

**현재** — Sprint Lv4 단계. 파라미터 캘리브레이션 시스템 (`calibration_runner.py`) 과 Composite Objective (IC Stability + Deflated Sharpe, `metrics/objective.py` on feature branch) 통합 중.

**완료**
- 7엔진 퓨전 아키텍처
- WebSocket 저지연 스트림
- Kill switch + Kelly sizing + ATR risk
- LLM Harness 게이트 (Gemini)
- BNB 자동 충전
- 회귀 테스트 스위트
- Unified Dashboard (`unified_dashboard/`)

**진행 중** (feature branch `0409_1710`)
- Feature normalization 라이브러리 (vol-adjusted return)
- IC Stability + Deflated Sharpe 기반 목적함수
- Walk-forward validation
- Matrix Pool (온라인 공분산 추정)

---

## ⚠️ 면책

본 프로젝트는 **개인 연구·포트폴리오 목적**으로 공개됩니다. 실거래에 사용할 경우 발생하는 어떠한 손실에 대해서도 저자는 책임지지 않습니다. 암호화폐 선물 거래는 원금을 초과하는 손실이 발생할 수 있습니다.

---

## 👤 About

기계공학 백그라운드에서 AI/Python 개발로 피벗 중인 개발자의 포트폴리오 프로젝트입니다. 수학·물리학적 문제 정의에서 실시간 인프라, LLM 추론 통합까지 전 스택을 직접 설계·구현했습니다.

- **GitHub**: [@hibou04-ops](https://github.com/hibou04-ops)
- **Repo**: [TradingBot_G5](https://github.com/hibou04-ops/TradingBot_G5)
