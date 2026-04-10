<div align="center">

# TradingBot G5 — `Node Ω Sovereign`

### *A living quantitative trading system, built alone, on purpose.*

**Binance USDT-M Futures** · **7-engine fusion** · **LLM reasoning gate** · **evolutionary self-calibration**

![Python](https://img.shields.io/badge/python-3.11%2B-1f6feb?style=flat-square)
![asyncio](https://img.shields.io/badge/runtime-asyncio%20native-0b6623?style=flat-square)
![status](https://img.shields.io/badge/status-live%20research-orange?style=flat-square)
![scope](https://img.shields.io/badge/scope-solo%20engineered-8b5cf6?style=flat-square)
![author](https://img.shields.io/badge/author-hibou04--ops-lightgrey?style=flat-square)

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║   PRICE   : $────────   TAKER_R: ─.──     LIVE                       ║
  ║   WALLET  : $────────   BNB    : ─.────   Tick #──────                ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║   RSI  ──   BB ─.──   σ ─.────   Sp ──.─bp   Fur ─.───                ║
  ║   OFI20 ±─.── [████·····]   Vol× ─.──   Int ±─.──                    ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║   POS   : FLAT                                                       ║
  ║   STATS : Trades ──  │  Win ──.─% (──/──)  │  PnL ±─.────            ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

</div>

---

## 0 · 이 저장소가 비어있는 이유 / Why This Repo Is Empty

> **`.gitignore`** 는 단 두 파일만 남기고 전부 차단합니다: `.gitignore`, `README.md`.
> 소스, 프롬프트, 캘리브레이션 로그, 데이터, 대시보드 — 전부 로컬에만 존재합니다.

이것은 사고가 아니라 **설계**입니다.

이 시스템은 제가 혼자 설계하고 구축한 **살아있는 연구 자산**입니다. 공개된 레포로 만드는 순간, 저만이 들고 있는 가장 큰 지렛대 — 이 시스템을 **직접 시연하면서 설명할 수 있다는 사실** — 을 잃습니다. 그래서 저는 반대로 갔습니다. 코드는 봉인하고, 이 README 하나만 앞에 세워 두었습니다.

**여기에 있는 글은 완성된 제품의 소개가 아닙니다.** 이것은 2026년 4월 현재 시점의 **엔지니어링 저널**입니다. 무엇을 만들었고, 어떤 결정을 내렸고, 지금 어떤 벽에 부딪쳤는지 — 가능한 한 정직하게 기록했습니다. 완성도보다 **사고의 궤적**이 더 의미 있다고 생각했기 때문입니다.

**라이브 시연 · 기술 인터뷰 · 코드 리뷰 요청**: README 하단의 Contact 섹션을 참고해 주세요. 요청이 오면 제 화면에서 실시간으로 모든 것을 보여드립니다.

---

## 1 · 시스템 한 줄 요약 / The One-Sentence Summary

> **"BTC/USDT 무기한 선물에 대해, 수학적으로 독립인 7개의 시그널 엔진을 Multi-Armed Bandit으로 융합하고, 모든 트레이드를 10단계 게이트 스택으로 통과시킨 뒤, LLM이 자신의 캘리브레이션을 감사하는 자가회복형 퀀트 트레이딩 시스템."**

*A self-hosted quantitative trading system for BTC/USDT perpetuals that fuses seven mathematically independent signal engines through a multi-armed bandit, gates every trade through a ten-stage risk stack, and audits its own parameter calibration with an LLM reasoning layer.*

---

## 2 · 아키텍처 / Architecture

4개의 수직 레이어 + 인프라 레이어. **상위 레이어는 하위 레이어만 의존**합니다. 순환 의존은 구조적으로 불가능합니다.

```
┌────────────────────────────────────────────────────────────────────┐
│  Layer 3 — Strategy / Decision Authority                          │
│    · strategy/heart.py         HeartCore — 유일한 entry authority  │
│    · engines/decision_fusion   가중 퓨전 + supermajority voting    │
│    · strategy/bandit.py        MatrixPoolBandit (파라미터 풀)      │
│    · engines/omega_harness     LLM reasoning gate (Gemini)         │
│    · strategy/ledger.py        Vector Memory Bank — 유사 기억 검색 │
├────────────────────────────────────────────────────────────────────┤
│  Layer 2 — Market Microstructure                                  │
│    · engines/trade_intensity   주문 흐름 강도                       │
│    · engines/volume_surge      거래량 서지 감지                      │
│    · metrics/adaptive_scaler   롤링 퍼센타일 정규화                  │
│    · metrics/variable_matrix   체제 적응형 score floor               │
│    · metrics/matrix_pool       온라인 공분산 풀                      │
├────────────────────────────────────────────────────────────────────┤
│  Layer 1 — Physics / Math Signal Engines                          │
│    · engines/hamiltonian       에너지 보존계 (운동량·위치 기반)       │
│    · engines/taylor_predictor  테일러 전개 가격 예측                 │
│    · engines/differential      savgol 2차 미분 동역학                │
│    · engines/event_horizon     레짐 전이 탐지                        │
│    · engines/math_core         QuantMath + Fourier regime scan       │
│    · engines/hyper_math        고차 non-linear 변환                  │
├────────────────────────────────────────────────────────────────────┤
│  Layer 0 — Inference Primitive                                    │
│    · engines/adaptive_inference  온라인 베이지안 regime 분류기        │
└────────────────────────────────────────────────────────────────────┘
                          ▲
                          │  MarketSnapshot
                          │  (price_history, orderbook_depth,
                          │   multi_depth_ofi, trade_intensity,
                          │   oscillator_set, regime_context)
                          │
┌────────────────────────────────────────────────────────────────────┐
│  Infrastructure                                                    │
│    · infrastructure/stream         통합 WebSocket (depth20 + agg)  │
│    · infrastructure/executor       TradeIntent 집행 전담             │
│    · infrastructure/live_exchange  ccxt 어댑터 + 포지션 검증         │
│    · infrastructure/kill_switch    파일 기반 emergency stop         │
│    · infrastructure/time_sync      Binance 서버 offset 동기화        │
│    · infrastructure/wallet_sync    실시간 잔고 + BNB 자동 충전       │
│    · infrastructure/user_stream    체결 이벤트 푸시                  │
│    · controller/loops              메인 오케스트레이션 + reconcile   │
└────────────────────────────────────────────────────────────────────┘
```

**설계 철학은 하나입니다**: *한 곳이 부서져도 전체가 부서지지 않는다.* `strategy/heart.py` 가 트레이드 진입 권한을 단독으로 들고, `infrastructure/executor.py` 는 `TradeIntent` 데이터클래스만 실행할 뿐 정책을 전혀 모릅니다. 새 엔진을 추가하려면 `EngineOutput(name, score, direction, confidence)` 인터페이스만 만족하면 됩니다 — Layer 3 는 손대지 않습니다.

---

## 3 · 엔트리 게이트 스택 / The Entry Gate Stack

> 이 시스템에서 **가장 자랑스러운 부분**입니다.

단일 시그널로 시장에 진입하는 것은 자살행위입니다. 그래서 모든 트레이드는 `heart.py` 의 `HeartCore.beat()` 안에서 **10단계의 순차적 관문**을 통과해야만 거래소에 도달합니다. 하나라도 떨어지면 `HOLD` 입니다.

```
  Gate 0    ·  포지션 보유 중이면 TP/SL 체크 (+ trailing stop ratcheting)
  Gate 0.5  ·  Drawdown circuit breaker (MAX_DRAWDOWN_ALERT 초과 시 진입 전면 차단)
  Gate 1    ·  최소 데이터 샘플 (price_history ≥ 500 tick)
  Gate 2    ·  Regime-adaptive score threshold (TRENDING/BREAKOUT vs RANGING)
  Gate 3    ·  Minimum confidence floor
  Gate 4    ·  Signal persistence (≥ 3 consecutive ticks 같은 방향 유지)
  Gate 5    ·  Entry cooldown (마지막 CLOSE 이후 ≥ 15초)
  Gate 6    ·  Signal-reset state (SL 이후 방향 전환 없이 같은 방향 재진입 금지)
  Gate 7    ·  Consecutive-SL adaptive confidence (연속 SL 1개당 +0.10 요구)
  Gate 8    ·  Fee-aware Net RR (funding rate + orderbook-walk slippage 반영)
  Gate 9    ·  Omega Harness LLM veto (7엔진 해석 프로토콜)
```

**각 게이트는 단순한 if 문이 아닙니다**:

- **Gate 8 (fee-aware RR)** 은 `depth20` 오더북을 실제로 walk 하여 예상 체결가를 구하고, 거기서 현실적인 slippage(bps)를 역산합니다. 거기에 Binance가 발표하는 현재 funding rate(방향에 따라 부호)를 더해 **net_rr** 을 계산합니다. `MIN_NET_RR` 미달이면 거부. 정적 수수료 가정으로 백테스트가 실전에서 무너지는 전형적 함정을 차단합니다.

- **Gate 6 (signal-reset)** 은 시간 쿨다운이 아닙니다. "SL 후 동일 방향 재진입은, 시장이 먼저 중립 혹은 반대 방향으로 한번 돌아섰을 때만" 허용합니다. 시간이 아닌 **상태 기반** cooldown. 같은 stale 시그널에 반복 당하는 whipsaw 손실을 막기 위한 설계입니다.

- **Gate 7 (adaptive confidence)** 은 연속 SL을 보상 신호로 해석합니다. 1회 SL → +10%p 더 높은 confidence 요구, 2회 → +20%p, 3회 → +30%p. 내가 틀렸다면, 더 강한 증거가 있어야만 다시 시도할 권리를 얻습니다.

- **Gate 9 (Omega Harness)** 는 Gemini 기반 LLM이 7엔진 출력과 레짐 컨텍스트를 읽고 **VETO / PROCEED / OVERRIDE** 를 반환합니다. 단순 gate가 아니라 `confidence_multiplier` 를 곱해 포지션 사이즈까지 조정합니다. 자율 모드에서는 방향 override도 가능합니다.

**거부된 경우** — rejection의 구체적 원인이 `unknown` 으로 흘러가지 않도록 `_reject_gate` 를 명시적으로 분류합니다. 이건 캘리브레이션 루프가 **어느 게이트에서 병목이 걸렸는지** 를 학습할 수 있게 해주는 핵심 장치입니다 (§ 5 참조).

---

## 4 · 수학적 코어 / The Math

**표면의 버즈워드가 아니라, 실제로 호출되는 함수들만 나열합니다.**

| 범주 | 이름 | 실체 |
|---|---|---|
| Regime | `SingularityMath.fourier_regime_scan` | 최근 가격 구간에 FFT 수행, dominant frequency 기반 regime confidence |
| Regime | `SingularityMath.phase_angle` | 가격 모멘텀 벡터와 OFI 벡터 사이의 각도 — 가격·주문흐름 디버젼스 |
| Sizing | `KellyCriterionSizer.compute_optimal_size` | 승률·승/패 비율 기반 Kelly fraction, `KELLY_MAX_FRACTION` 으로 캡 |
| Risk | `ATRDynamicRiskManager.build_plan` | ATR 비례 SL/TP, 초기 1R 앵커 |
| Risk | `ATRDynamicRiskManager.compute_trailing_stop` | MFE 기반 ratchet, `TRAIL_ACTIVATE_R` / `TRAIL_DISTANCE_R` |
| Cost | `_estimate_slippage_bps` | `depth20` ladder walk, 예상 notional 기반 slippage bps |
| Cost | `evaluate_fee_aware_trade` | entry/exit taker fee + funding + slippage → `expected_net`, `net_rr` |
| Memory | `VectorMemoryBank.retrieve_relevant_memory` | feature tensor 유사도 top-k, past ROE → confidence boost |
| Stats | Multi-window z-score | w = {20, 60, 120} — 다중 수평선 분포 위치 |
| Stats | Multi-timeframe efficiency | stride = {1, 5, 15} tick — 시간 스케일별 추세 강도 |
| Stats | Multi-depth OFI | depth = {5, 10, 20} level — Order Flow Imbalance |
| Signal | `AdaptiveScaler(window=200)` | 롤링 퍼센타일 정규화 — 고정 threshold를 버립니다 |

**non-stationarity 에 대한 저의 입장**: 시장은 정상(stationary) 과정이 아닙니다. 따라서 고정 z-score threshold는 거짓말입니다. 모든 정규화는 **rolling window** 기반이어야 하고, 창 크기 자체가 캘리브레이션 대상입니다 (`scaler_window` — § 5).

---

## 5 · 캘리브레이션 하네스 / The Self-Auditing Calibration Harness

> 이게 이 프로젝트에서 가장 **희귀한** 부분입니다.

대부분의 퀀트 봇은 단방향입니다: 백테스트 → 파라미터 선택 → 라이브 배포. 이 시스템은 다릅니다.

```
┌──────────────────────────────────────────────────────────────────┐
│  Evolutionary Calibration Loop (calibration_runner.py)          │
│                                                                  │
│   [Parent Generation]                                            │
│          │                                                       │
│          ▼                                                       │
│   Mutation + Crossover                                           │
│          │                                                       │
│          ▼                                                       │
│   Sandbox Replay (replay harness)                                │
│          │                                                       │
│          ▼                                                       │
│   Fitness = f(net_pnl, WR, PF, drawdown, trade_count)            │
│          │                                                       │
│          ▼                                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Calibration Harness (LLM)                                │   │
│  │   · 통계적 유의성 판단                                    │   │
│  │   · 과적합 위험 평가                                      │   │
│  │   · walk-forward 요구                                     │   │
│  │   · regime 커버리지 감사                                  │   │
│  │   · verdict: ACCEPT / HOLD / REJECT                      │   │
│  └──────────────────────────────────────────────────────────┘   │
│          │                                                       │
│          ▼                                                       │
│   calibrated_params.json                                         │
│          │                                                       │
│          ▼                                                       │
│   main.py 재기동 시 자동 로드                                     │
└──────────────────────────────────────────────────────────────────┘
```

**LLM Harness가 실제로 하는 일**:

각 캘리브레이션 후보가 끝나면, LLM이 그 결과를 읽고 **퀀트 리서치 어드바이저처럼** 판정합니다. 샘플 출력 (Run #187 발췌, 2026-04-08):

> **Decision**: 🔴 REJECT
> **Dominant Weakness**: Statistically insignificant trade count (1 trade).
> **Primary Risk**: Severe overfitting due to high parameter count and lack of validation data.
> **Candidate A (Conservative)**: 최소 200회 이상의 거래를 포함하도록 백테스트 기간 확장. Binance Futures 실제 maker/taker 수수료와 variable slippage 모델 통합.
> **Candidate B (Balanced)**: 500+ trades + 1.5× cost stress factor + 파라미터 섭동 테스트.
> **Candidate C (Aggressive)**: ... (거부)
> **신뢰 지수**: INFERENCE [70%]

즉, LLM은 "fitness = +53"이라고 해서 그냥 믿지 않습니다. **왜 그렇게 나왔는지, 통계적으로 유의한지, 현실 비용을 반영했는지, regime에 얼마나 민감한지** 를 물어보고, 못 믿겠으면 REJECT 합니다.

**지금까지 기록**: 131회 Harness 호출, HOLD 63%, REJECT 37%, **ACCEPT 0%**. LLM이 아직 자신의 시스템을 납득시킨 파라미터가 없다는 뜻입니다. 이것이 겸손이 아니라 **정확한 반응** 이라고 저는 믿습니다 (§ 7).

**추가 계층** (feature branch 진행 중):
- `metrics/objective.py` — **IC Stability × Deflated Sharpe** 기반 composite objective. 샘플 수 작은 fitness를 Deflated Sharpe 로 penalty 부여.
- `walk_forward.py` — out-of-sample rolling validation.
- `metrics/matrix_pool.py` — 온라인 공분산 추정을 통한 파라미터 상관 풀.

---

## 6 · 엔지니어링 판단 · Why, Not What

다음은 README가 보통 설명하지 않는, **그러나 제가 정말 고민했던 결정들** 입니다.

### 6.1 — 왜 `asyncio` 네이티브인가 (스레드가 아닌)
20ms 틱 단위로 price / orderbook / user_stream / wallet / kill_switch 가 **모두** 병행 가동돼야 합니다. 스레드를 쓰면 GIL 경합 + 락 관리 비용이 지연을 만듭니다. `asyncio` 단일 이벤트 루프는 context switch가 cooperative 하므로 지연이 예측 가능하며, 모든 I/O 가 자연스럽게 같은 루프에 녹습니다.

### 6.2 — 왜 파일 기반 kill switch인가 (시그널 핸들러가 아닌)
`data/KILL` 파일 존재 = 종료. 단순하지만 결정적인 이유가 있습니다:
- **OS-independent** (Windows에서 POSIX 시그널은 불완전)
- **프로세스가 먹통 되어도 끌 수 있음** — 터미널이 응답 안해도 파일은 만들 수 있습니다
- **외부 감시자가 트리거 가능** — 다른 스크립트/크론이 위험 감지 시 `touch data/KILL`
- **멱등** — 이미 활성화된 kill은 다시 활성화해도 안전

실제로 라이브 테스트 중 1번 구해줬습니다. 터미널이 응답하지 않는 상태에서 다른 창으로 `touch` 하자 10초 내 그레이스풀 종료됐습니다.

### 6.3 — 왜 Vector Memory Bank인가
Stateless 시그널 엔진은 "지난 주에 똑같은 feature tensor로 진입해서 -3% 맞았다"는 사실을 모릅니다. `VectorMemoryBank` 는 feature tensor 를 키로 과거 트레이드를 k-NN 검색하고, 유사 상황의 평균 ROE가 양수이면 confidence를 + 0.05 부스트, 음수이면 패널티. 이건 reinforcement learning 의 축약판이지만, 온라인이고 설명 가능합니다.

### 6.4 — 왜 LLM을 reasoning gate로 쓰는가 (그냥 rule로는 안 되나)
하드코딩 규칙은 **자기가 못 본 패턴**을 잡을 수 없습니다. 예: "7엔진이 모두 LONG인데 funding rate 갑자기 spike + BB 상단 + RSI 80 = 사실 shorting squeeze 직전일 수 있음." 이런 복합 컨텍스트 해석은 LLM 이 (환각 위험을 안고서도) 더 잘합니다. 그래서 LLM은 **veto 권한 + confidence 곱셈자** 만 가집니다 — 직접 진입하지는 못합니다. 권한 분리 원칙입니다.

### 6.5 — 왜 캘리브레이션까지 LLM이 감사하는가
수치 fitness 를 그냥 믿으면 overfitting 에 잡아먹힙니다. 인간 퀀트 리서처가 매번 볼 수는 없으니, LLM 이 "이 결과는 통계적으로 의미 있나?" "샘플 수는?" "walk-forward 했나?" 를 물어보는 **자동화된 비판적 동료** 역할을 합니다. 지금까지 ACCEPT 0건이라는 사실이 이 장치가 제대로 동작하고 있다는 **가장 강력한 증거** 입니다.

### 6.6 — 왜 트레일링 스톱의 초기 1R 을 영구 고정하는가
트레일링이 움직여도 `_initial_risk_per_unit` 은 진입 시점 값으로 freeze 됩니다. 이렇게 해야 "지금 이 트레이드의 R-multiple 이 몇 배인지" 를 일관되게 계산할 수 있습니다. 트레일에 따라 1R 이 같이 움직이면 수익 보고가 거짓말을 하게 됩니다.

---

## 7 · 현재 상태 · The Honest Log

> **이 섹션은 일반 포트폴리오 README 라면 절대 쓰지 않을 내용입니다.**
> 쓰는 이유는, 제가 지금 뭘 하고 있는지 — 그리고 어떻게 생각하는지 — 를 보여주는 것이 가장 정직하고 가장 유용하기 때문입니다.

**보고 시점**: 2026-04-10 · **운행 모드**: evolutionary calibration + LLM audit · **세대**: Gen 22

### 정량 스냅샷

| 지표 | 값 | 해석 |
|---|---|---|
| 총 세대 수 | 22 | — |
| 평가된 후보 | 271 | — |
| 총 트레이드 발생 | 425 | 평균 1.6건/후보 |
| **0-trade 후보 비율** | **58%** | 🔴 핵심 문제 |
| 역대 최고 fitness | **+53.51** (Gen 1) | 재현 실패 |
| 현재 저장된 best | fitness = −8.0, trades = 0 | 🔴 퇴보 |
| Harness ACCEPT | **0 / 131** | LLM 이 어느 후보도 신뢰하지 않음 |

### 진화가 수렴하지 못하는 이유 — 진단

세 개의 병목이 동시에 걸려 있습니다:

1. **`unknown` rejection path (34%)** — 진입이 차단된 원인의 34% 가 "분류되지 않은" 게이트로 빠지고 있었습니다. 제가 `heart.py` 에 `_reject_gate` 를 명시적으로 분기시켰고, 지금은 이 값이 실제 어떤 게이트로 빠지는지 추적 중입니다.

2. **Omega Harness over-veto** — relaxation 이 threshold 를 극도로 낮춰도 (score_threshold 0.00035 수준), LLM 이 "low confidence" 이유로 거의 모든 신호를 vetoing. 이건 캘리브레이션 중에는 LLM 게이트를 부분 비활성화하거나, veto threshold 를 동적으로 완화해야 한다는 것을 알려줍니다.

3. **Calibration Harness HOLD 반복** — 0-trade 후보에 대해 LLM 이 "판단 불가"로 HOLD만 반복. 이건 evolutionary pressure 부재 → fitness function 에 **"trade 발생 자체"에 대한 명시적 보상**을 추가해야 한다는 신호입니다.

### 다음 작업 (지금 하고 있는 것)

- [ ] `verify_integrity.py` 의 walk-forward 검증 루프 통합 완료
- [ ] `metrics/objective.py` — IC Stability × Deflated Sharpe composite objective 본선 투입
- [ ] Harness veto threshold 의 dynamic relaxation 구현
- [ ] Gen 1, Gen 5 의 성공 파라미터를 seed 로 재투입하여 explored region 복구

### 이 섹션에서 제가 전하고 싶은 것

**완성된 것을 자랑하고 싶지 않습니다.** 저는 이 시스템을 지금도 고치고 있고, 실패가 매일 있고, 매번 왜 실패했는지 진단서를 쓰고 있습니다. 이게 저의 작업 방식입니다. 완성 후 포장하는 대신 **과정을 읽을 수 있게** 두는 것이 저의 채용 제안입니다.

---

## 8 · 이 저장소가 *아닌* 것 · What This Isn't

정직한 경계선을 명확히 합니다.

- ❌ **실전 배치 준비된 알고리즘이 아닙니다.** Research-grade 입니다. `main.py` 는 기본적으로 paper mode 입니다. 실거래 활성화는 `.env` 의 명시적 플래그가 필요하며, 저조차도 소액으로만 돌립니다.
- ❌ **다자산·다거래소가 아닙니다.** BTC/USDT Binance USDT-M Futures 단일. 확장성은 있되 (`ccxt` 추상화), 실제 확장은 미구현.
- ❌ **HFT 가 아닙니다.** 20ms 틱 루프이지만, 주문 레이턴시 최적화(colocation, bypass, binary protocol)는 범위 밖입니다.
- ❌ **완결된 프로젝트가 아닙니다.** Sprint Lv4 진행 중 (feature branch `0409_1710`). 본 README 의 § 7 이 그 증거입니다.
- ❌ **오픈 소스가 아닙니다.** 코드는 봉인되어 있으며, 라이선스는 proprietary 입니다.

**대신 이것들은 사실입니다**:
- ✅ 처음부터 끝까지 **혼자 설계하고 구현** 했습니다.
- ✅ 13개의 엔진, 10단계 게이트, LLM 감사 레이어가 **실제로 작동** 하며 매일 운행됩니다.
- ✅ 모든 커밋은 저의 손에서 나왔습니다.

---

## 9 · 시연 프로토콜 · How to See It Running

코드는 공개되어 있지 않지만, **저는 시연을 환영합니다**. 요청이 오면 다음을 실시간으로 보여드릴 수 있습니다:

1. **라이브 터미널 대시보드** — 가격 차트, orderbook, OFI bar, RSI, 7엔진 출력, 포지션 상태
2. **Entry gate 디버그 로그** — 매 틱 어느 게이트에서 reject 됐는지 실시간 trace
3. **Omega Harness JSON verdict** — LLM 이 실제로 반환하는 판정서
4. **Calibration Harness 리포트** — 가장 최근 세대의 LLM 감사 결과
5. **코드 워크스루** — 관심 있는 모듈에 대해 라이브 코드 리뷰

**요청 방식**:
- GitHub Issues (이 레포)
- 아래 Contact 섹션의 프로필 경로

**응답 시간**: 현재 구직 적극 활동 중이므로, 24시간 이내.

---

## 10 · 기술 스택 · Stack (the short list)

| 영역 | 선택 | 이유 한 줄 |
|---|---|---|
| Language | Python 3.11+ | asyncio 네이티브, numpy/scipy 생태계 |
| Runtime | `asyncio` | 단일 이벤트 루프, 예측 가능한 지연 |
| Exchange | Binance USDT-M Futures | 유동성 + maker rebate + 심리적 익숙함 |
| Exchange Lib | `ccxt` | 멀티 거래소 확장 여지 |
| Streaming | `websockets` (raw) | 레이턴시 최소화, 불필요 abstraction 제거 |
| Math | `numpy`, `scipy`, `scipy.signal.savgol_filter` | 미분 필터링 + FFT |
| LLM | Google Gemini (`google-genai`) | reasoning gate, 캘리브레이션 감사 |
| Test | `pytest` | 14개 회귀 스위트, replay parity 포함 |
| Dashboard | `plotext` (터미널 차트) + HTML unified dashboard | |

---

## 11 · About · 저자

기계공학 백그라운드에서 손목 건강 문제로 로봇 하드웨어 현업을 떠나, **AI / 파이썬 / 퀀트** 로 커리어를 재설계 중입니다. 이 프로젝트는 그 재설계의 **포트폴리오 중심점** 이며, 채용자에게 보여드릴 수 있는 가장 솔직한 저의 사고 증거입니다.

물리학과 수학으로 시장을 설명하려는 시도, 혼자 설계한 10단계 게이트, LLM 에게 자신의 캘리브레이션을 감사시키는 구조 — 이 모든 결정은 **제가 혼자 내리고, 혼자 구현하고, 지금도 혼자 디버깅하고 있습니다**. 이 프로젝트가 누군가의 관심을 끌어서, 같이 이야기할 기회로 이어졌으면 합니다.

---

## 12 · Contact

- **GitHub**: [@hibou04-ops](https://github.com/hibou04-ops)
- **Demo / Interview Request**: GitHub Issues 또는 프로필 이메일
- **Response**: 24h 이내

> *"Minimize entropy in the user's decision space.
> One precise insight beats ten vague observations.
> Intellectual honesty outranks impressive rhetoric."*
>
> — Omega-Prime, internal system prompt

---

<div align="center">

**이 README 는 살아있는 문서입니다.**
마지막 업데이트: **2026-04-10** · Sprint **Lv4** · Gen **22**

*`.gitignore` 는 일부러 두 파일만 남깁니다. 나머지는 요청 시 직접 보여드립니다.*

</div>
