<div align="center">

# TradingBot G5

### *비정상 시장환경에서 파라미터 드리프트와 리스크 제어를 연구하기 위한 비공개 시스템*

### *A private research platform for parameter drift and risk control in non-stationary crypto futures markets*

![Python](https://img.shields.io/badge/python-3.11%2B-1f6feb?style=flat-square)
![runtime](https://img.shields.io/badge/runtime-asyncio-0b6623?style=flat-square)
![scope](https://img.shields.io/badge/scope-research%20platform-orange?style=flat-square)
![code](https://img.shields.io/badge/code-private-red?style=flat-square)
![origin](https://img.shields.io/badge/origin%20of-omega--lock-8b5cf6?style=flat-square)

</div>

---

## 0 · 무엇을 공개하고, 무엇을 비공개하는가

이 저장소는 **실거래용 시그널, 임계값, 실행 로직을 공개하지 않습니다**.

| 공개 ✅ | 비공개 ❌ |
|---|---|
| 시스템 아키텍처 | signal formula |
| 검증 철학 (validation philosophy) | threshold 값 / parameter search space |
| 리스크 제어 구조 (categories) | feature 조합 / 실행 조건 |
| 문제 정의 (non-stationarity) | 실거래 성과 / 운영 수치 |
| omega-lock 으로의 일반화 경로 | exchange/account 운영 세부 |

전략 자산 보호와 오용 방지를 위한 **의도된 설계**입니다. 인터뷰·기술 시연 맥락에서는 화면을 통해 직접 시연합니다 (§ 7).

> **이 README의 목적은 "이 봇이 얼마나 수익을 내는지" 자랑하는 것이 아닙니다.**
> 비정상 시계열에서 의사결정 시스템을 어떻게 설계·검증·감시할 것인가에 대한 **사고의 궤적**을 보여주는 것이 목적입니다.

---

## 1 · Problem — 시장은 고정된 자물쇠가 아니다

암호화폐 무기한 선물 시장은 **non-stationary** 합니다. 백테스트에서 좋아 보이는 파라미터는 시장 체제가 바뀌는 순간 무효화되며, 어제 효과적이었던 임계값이 오늘은 노이즈가 됩니다.

> 시장은 고정된 자물쇠가 아니라, 조합이 계속 바뀌는 잠금장치에 가깝습니다.
> 이 프로젝트의 목표는 **"영구적으로 맞는 파라미터"를 찾는 것이 아니라**,
> 파라미터가 **언제 깨지는지** 추적하고, 깨지기 전에 포지션 크기와 실행 권한을 줄이는 것입니다.

따라서 이 프로젝트의 핵심 질문은 "어떤 파라미터가 가장 수익을 내는가?"가 아니라:

- 현재 파라미터가 **언제** 더 이상 유효하지 않은가?
- 파라미터 붕괴를 어떻게 **사전에 감지**할 수 있는가?
- 백테스트의 좋은 결과가 forward 구간에서도 살아남는다는 것을 **어떻게 검증**하는가?
- 무엇을 신호로 받아들이고, 무엇을 noise로 거부할 것인가?

---

## 2 · Architecture — 4계층 분리 + 단일 권한 채널

상위 레이어는 하위 레이어만 의존합니다. 순환 의존이 구조적으로 불가능하도록 설계했습니다.

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 3 — Decision Authority                              │
│   · 단일 entry authority (모든 진입 결정의 유일 채널)         │
│   · Decision Fusion (다중 시그널 가중 융합)                   │
│   · Parameter pool (Multi-Armed Bandit 기반 후보 선택)       │
│   · Reasoning gate (LLM 기반 컨텍스트 감사)                  │
├─────────────────────────────────────────────────────────────┤
│  Layer 2 — Market Microstructure                           │
│   · 주문 흐름 강도, 거래량 서지, 롤링 분포 정규화            │
├─────────────────────────────────────────────────────────────┤
│  Layer 1 — Signal Engines (비공개)                          │
│   · 수학적으로 독립인 다중 시그널 엔진                        │
├─────────────────────────────────────────────────────────────┤
│  Layer 0 — Inference Primitive                             │
│   · 온라인 베이지안 regime 분류기                             │
└─────────────────────────────────────────────────────────────┘
                           ▲
                           │  MarketSnapshot
                           │
┌─────────────────────────────────────────────────────────────┐
│  Infrastructure                                             │
│   · 통합 WebSocket · 실행 전담 executor · 거래소 어댑터      │
│   · kill switch · 시간 동기화 · 잔고 관리 · 사용자 스트림    │
└─────────────────────────────────────────────────────────────┘
```

**설계 원칙**:

- **권한 분리**: 진입 결정 권한은 단일 모듈에만 있습니다. 실행 모듈은 정책을 모르고, intent 데이터클래스만 실행합니다.
- **장애 격리**: 한 레이어가 부서져도 전체가 부서지지 않습니다. Kill switch는 파일 기반으로 구현하여 OS·프로세스 상태와 독립적으로 작동합니다.
- **확장 인터페이스**: 새 시그널 엔진 추가는 정해진 출력 인터페이스만 만족하면 됩니다 — 상위 레이어는 수정하지 않습니다.

---

## 3 · Validation Philosophy — "최적 파라미터"가 아닌 "붕괴 감지"

대부분의 퀀트 봇은 단방향입니다: 백테스트 → 파라미터 선택 → 라이브 배포. 이 시스템은 그 가정을 거부합니다.

| 검증 축 | 질문 | 도구 |
|---|---|---|
| Walk-forward calibration | 과거 윈도우의 best가 다음 윈도우에서도 살아남는가? | Rolling holdout |
| Parameter sensitivity | 작은 파라미터 섭동에 결과가 흔들리는가? | Coordinate-descent perturbation |
| Regime coverage | 후보가 모든 regime에 노출됐는가, 한 regime에 과적합된 게 아닌가? | Regime-stratified replay |
| Statistical significance | trade 수가 통계적으로 의미 있는가, 운으로 좋게 나온 게 아닌가? | Sample-size penalty (e.g. Deflated Sharpe) |
| Cost realism | funding · slippage · taker fee 가 현실적으로 반영됐는가? | Orderbook-walk slippage 시뮬 |
| LLM critical review | 위 항목을 사람이 매번 검토할 수 없을 때, 자동화된 비판적 동료 | LLM verdict (ACCEPT / HOLD / REJECT) |

**핵심 원칙**: 통과한 후보는 "수익이 났다"가 아니라, **"위 모든 축에서 살아남았다"** 여야 합니다. 어느 한 축이 깨지면 ACCEPT 하지 않습니다.

> 수치 fitness 만 보고 통과시키면 overfitting에 잡아먹힙니다. 인간 퀀트 리서처가 매번 볼 수는 없으니, LLM 이 "이 결과는 통계적으로 의미 있나?", "샘플 수는?", "walk-forward 했나?" 를 물어보는 **자동화된 비판적 동료** 역할을 합니다.

---

## 4 · Risk Control Layer — 모든 진입은 다단계 게이트를 통과한다

단일 시그널로 시장에 진입하는 것은 자살행위입니다. 모든 트레이드는 여러 단계의 순차적 관문을 통과해야만 거래소에 도달합니다. **하나라도 떨어지면 진입하지 않습니다.**

**게이트 카테고리** *(구체적 임계값과 순서는 비공개)*:

- **Position state gate** — 포지션 보유 중일 때는 TP/SL · trailing stop 관리만 허용
- **Drawdown circuit breaker** — 누적 손실이 임계 초과 시 진입 전면 차단
- **Data sufficiency gate** — 통계적 안정에 필요한 최소 샘플 미달 시 차단
- **Regime-adaptive threshold gate** — 체제별로 신호 임계값이 다르게 적용 (TRENDING vs RANGING)
- **Confidence floor gate** — 최소 확신도 미달 시 차단
- **Signal persistence gate** — 일시적 노이즈가 아닌, 일관된 신호만 통과
- **State-based cooldown** — 시간 기반이 아닌 *시장 상태* 기반 cooldown — 같은 stale 시그널의 whipsaw 차단
- **Adaptive confidence gate** — 연속 손절 발생 시 다음 진입에 더 강한 증거 요구
- **Fee-aware net RR gate** — funding · slippage · taker fee 를 반영한 실효 손익비 검사 (정적 수수료 가정으로 백테스트가 실전에서 무너지는 함정 차단)
- **Reasoning veto gate** — LLM 기반 컨텍스트 감사. 복합 패턴(예: "모든 시그널이 LONG 인데 funding spike + RSI 과열")에 대한 거부 권한. 직접 진입 권한은 없고 *veto + confidence multiplier* 만 가집니다 — **권한 분리**.

**거부 사유의 명시적 분류**: 각 거부가 "어느 게이트에서 차단됐는지" 명시적으로 추적합니다. `unknown` 으로 흘러가는 거부는 캘리브레이션 루프가 학습할 수 없는 dead zone이기 때문입니다.

**리스크 관리 추가 레이어**:

- ATR 비례 동적 SL/TP
- MFE 기반 trailing stop ratchet (*초기 1R 영구 고정* — 진입 시점 risk 단위가 trailing 따라 움직이면 R-multiple 보고가 거짓말이 됩니다)
- Kelly fraction 사이징 (승률·승패비 기반, 명시적 cap)
- 파일 기반 kill switch (OS·프로세스 상태와 독립적, 외부 감시자가 트리거 가능)
- Recovery logic — 비정상 종료 후 포지션·잔고 reconciliation

---

## 5 · Audit Event Logger — 시스템은 자기 자신을 기록한다

모든 결정은 **append-only 감사 로그**로 기록됩니다. 사후에 "왜 이 진입이 차단됐나?", "왜 이 캘리브레이션이 거부됐나?" 를 재구성할 수 있어야 합니다.

- **Tick-level decision trace** — 각 틱의 시그널 출력, 거부 게이트, 최종 의사결정
- **Calibration verdict log** — 후보별 fitness · 통계 진단 · LLM verdict
- **Trade audit** — 진입·청산 시 비용 분해(수수료 · slippage · funding) 및 R-multiple
- **Reject distribution** — 게이트별 거부율 — evolutionary loop가 어느 병목에 막혀 있는지 진단

> "시장이 안 좋아서" 는 진단이 아닙니다. 시스템 결정은 **시스템 로그로 설명**되어야 합니다.

---

## 6 · omega-lock과의 관계 — 이 프로젝트의 가장 중요한 산출물

**TradingBot G5 의 가장 중요한 산출물은 트레이딩 그 자체가 아니라, 파라미터 검증 방법론입니다.**

이 프로젝트에서 가장 자주 마주친 문제는 단 하나였습니다:

> 과거 데이터에서 좋아 보이는 파라미터가, **정말 forward 구간에서도 살아남는가?**

수익률을 자랑하기 위해서가 아니라, 이 질문에 정직하게 답하기 위해 만든 것이:

- **walk-forward calibration**
- **sensitivity analysis**
- **hard-constraint compliance**
- **append-only audit trail**

이 방법론을 트레이딩이라는 도메인에서 **분리하여 일반화**한 결과물이 다음 도구들입니다:

| 산출물 | 역할 |
|---|---|
| **[omega-lock](https://github.com/hibou04-ops/omega-lock)** | Method-agnostic calibration audit + sensitivity-driven search framework. Walk-forward gates · hard-constraint compliance · append-only trail |
| **[omegaprompt](https://github.com/hibou04-ops/omegaprompt)** | Calibration discipline applied to Claude API prompts (omega-lock 포팅) |
| **[antemortem-cli](https://github.com/hibou04-ops/antemortem-cli)** · **[Antemortem](https://github.com/hibou04-ops/Antemortem)** | Pre-implementation reconnaissance methodology |
| **[mini-omega-lock](https://github.com/hibou04-ops/mini-omega-lock)** · **[mini-antemortem-cli](https://github.com/hibou04-ops/mini-antemortem-cli)** | omega-lock / antemortem 캘리브레이션 검증용 empirical · analytical preflight probes |

즉, **TradingBot G5는 이 도구들의 origin story** 입니다. 비정상 시장에서 파라미터를 검증해야 했던 경험이, 도메인-중립적인 ML/LLM 캘리브레이션 감사 도구로 일반화됐습니다. 현재 작업의 무게중심은 위 도구들에 있고, TradingBot G5 는 그 방법론이 처음 만들어진 **연구 환경**으로 운영됩니다.

---

## 7 · 시연 · How to See It Running

코드는 비공개이지만, **실시간 시연은 환영합니다**. 요청 시 다음을 보여드립니다:

1. 라이브 터미널 대시보드 — 가격 · orderbook · 시그널 출력 · 포지션 상태
2. 게이트 거부 trace — 매 틱 어느 게이트에서 차단됐는지 실시간 로그
3. 캘리브레이션 verdict 리포트 — 가장 최근 세대의 LLM 감사 결과
4. 관심 모듈에 대한 라이브 코드 워크스루

**Contact**:

- GitHub: [@hibou04-ops](https://github.com/hibou04-ops)
- Demo / Interview Request: GitHub Issues 또는 프로필 이메일
- Response: 24h 이내

---

## 8 · About · 저자

기계공학 백그라운드에서 손목 건강 문제로 로봇 하드웨어 현업을 떠나, **AI · Python · 퀀트 리서치 / ML 캘리브레이션 방법론** 으로 커리어를 재설계 중입니다.

이 프로젝트의 의의는 자동화된 매매가 아니라, **비정상 환경에서 의사결정 시스템을 어떻게 설계하고 검증할 것인가**에 대한 답을 만들어가는 데 있습니다. 그 답의 일반화된 형태가 [omega-lock](https://github.com/hibou04-ops/omega-lock) · [omegaprompt](https://github.com/hibou04-ops/omegaprompt) · [antemortem-cli](https://github.com/hibou04-ops/antemortem-cli) 입니다.

---

<div align="center">

**이 README 는 살아있는 문서입니다.** · 마지막 업데이트: **2026-04-30**

*공개 README 는 시스템 구조와 검증 철학만 다룹니다.*
*실거래 시그널 · 임계값 · 실행 로직은 비공개입니다.*

</div>
