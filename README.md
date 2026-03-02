🔷 1️⃣ ML Layer (의료 판단 레이어)
입력

RR, Vte, MinVent, Leak, Dyn C

7일 이동 평균

표준편차

이벤트 빈도

PRO 이벤트

출력
risk_score = 0.72
triage = "YELLOW"
top_contributors = [
  ("RR_variability", +0.18),
  ("MinVent_drop", +0.12),
  ("Leak_spike", +0.07)
]

이 레이어는:

LightGBM 또는 LSTM+LightGBM

SHAP 기반 설명 가능

SaMD 인허가 대상

🔷 2️⃣ Orchestration Layer (핵심)

이 레이어가 진짜 중요합니다.

역할:

Risk score threshold 판단

응답 템플릿 선택

LLM에 전달할 structured context 생성

LLM이 “의료 판단”을 하지 못하도록 제어

예:

{
  "triage": "YELLOW",
  "risk_score": 0.72,
  "primary_factors": ["RR 증가", "MinVent 감소"],
  "trend": "최근 48시간 점진적 악화",
  "alarm_events": 2,
  "instruction_policy": "no_diagnosis"
}
🔷 3️⃣ LLM Layer (비의료 판단 인터페이스)

LLM이 하는 일:

수치 → 자연어 설명

환자 친화적 표현 변환

과도한 의료적 표현 차단

보호자 질의응답

LLM이 하지 않는 일:

진단

처방

위험도 계산

🔷 🔒 안전 구조 (매우 중요)

의료 앱에서는 이 부분이 핵심입니다.

1️⃣ Prompt Guardrail

시스템 프롬프트 예:

You are a clinical communication assistant.
You DO NOT provide medical diagnosis or treatment advice.
You only explain ML-generated risk scores.
If user requests medical advice, instruct to consult clinician.
2️⃣ Output Validator

LLM 응답 후 필터링:

“진단”, “치료”, “처방” 포함 여부 탐지

특정 키워드 차단

Red 상태일 경우 강제 의료진 연결 문구 추가

3️⃣ Risk-Level Override

만약 triage == RED:

LLM이 어떤 문장을 생성하든
최종 출력은 템플릿 강제 적용:

"위험 신호가 감지되었습니다.
즉시 의료진과 연락하시기 바랍니다."

LLM 자유 생성 금지.

🏗 최적 구조 (실제 SaaS 적용형)
                 ┌──────────────────┐
                 │ Ventilator Data  │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │  ML Risk Engine  │
                 │ (LightGBM/LSTM)  │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Orchestration    │
                 │  - Rule Engine   │
                 │  - Policy Layer  │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │   LLM Interface  │
                 │  (Explanation)   │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Patient App UI   │
                 └──────────────────┘
🎯 환자용 앱에 맞춘 역할 분리
레이어	인허가 대상	역할
ML Engine	✅ 대상	위험도 계산
Rule Engine	✅ 대상	트리아지 분류
LLM	❌ 비대상	설명 및 인터페이스

이렇게 분리하면 규제 리스크 최소화됩니다.
