# OG 프로젝트 스냅샷 템플릿

OG에게 프로젝트를 가져올 때 이 파일을 복사해서 채운다.
없는 항목은 "없음"으로 명시한다 — 그것 자체가 진단 정보다.

---

## 기본 정보

```
프로젝트명:
OpenClaw 버전:         (예: v2026.2.26)
운영 환경:             (예: Mac Mini M4 24GB / MacBook Pro M3 / VPS)
운영체제:              (예: macOS Sequoia 15.6 / Ubuntu 24.04)
모델:                  (예: ollama/qwen2.5:14b / anthropic/claude-sonnet-4-5)
채널:                  (예: Telegram / WhatsApp / TUI)
에이전트 수:           (예: 단일 / 멀티 3개 — Amy·Scarlett·Natalie)
사용 목적:             (예: 뉴스 브리핑 자동화 / CEO 인텔리전스 / 내부 업무 보조)
```

---

## 에이전트 구조

```
에이전트 목록:
  [ID]       모델: [모델명]          역할: [한 줄 설명]
  예) amy     모델: qwen2.5:14b      역할: 라우터 + 사용자 인터페이스
  예) scarlett 모델: qwen2.5:14b     역할: 뉴스 수집 및 브리핑 작성
  예) natalie  모델: deepseek-r1:14b 역할: 전략 분석

멀티에이전트:
  agentToAgent 활성화: 예 / 아니오
  sessions_send 사용:  예 / 아니오
  라우팅 구조:         (예: Amy → Scarlett → Natalie)
```

---

## 파일 구조

```
파일명                           역할
-------------------------------  ------------------------------------
예) openclaw.json                메인 설정 파일
예) agents/amy/SOUL.md           Amy 정체성 및 행동 지침
예) agents/scarlett/SOUL.md      Scarlett 정체성 및 행동 지침
예) MEMORY.md                    장기 기억
예) HEARTBEAT.md                 Heartbeat 조건 정의
```

---

## openclaw.json 핵심 설정 요약

전문은 파일로 첨부한다. 여기에는 현재 상태만:

```
compaction.mode:                  (예: default / safeguard)
memoryFlush:                      활성화 / 비활성화
session.reset.daily:              true / false
hooks.internal.session-memory:    활성화 / 비활성화
Gateway 데몬:                     설치됨 / 미설치
api 타입:                         ollama (네이티브) / openai (호환 레이어)
```

---

## 현재 문제 또는 요청

**문제가 있는 경우:**

```
문제 상황:
  [어떤 동작을 했을 때]
  [실제로 어떻게 됐는가]
  [기대했던 동작은 무엇이었는가]

관련 로그:
  (openclaw logs --limit 50 출력을 여기 붙여넣기 또는 파일 첨부)

발생 빈도:   (예: 항상 / 가끔 / 특정 조건에서만)
시도한 것:   (예: gateway stop→start / doctor --fix)
```

**점검이 목적인 경우:**

```
현재 만족 중, 점검받고 싶은 부분:
```

---

## 요청 사항

```
□ 진단 보고서 (원인 파악 + 해결 방법)
□ 설정 파일 제작/수정 (openclaw.json / SOUL.md)
□ 자동화 설계 (방법 선택 포함)
□ 멀티에이전트 아키텍처 설계
□ 특정 부분 집중 검토: [어느 부분]
□ 기타:
```

---

## 첨부 파일 체크리스트

```
□ openclaw.json (민감 정보 마스킹 후)
□ SOUL.md — Amy
□ SOUL.md — Scarlett (있다면)
□ SOUL.md — Natalie (있다면)
□ HEARTBEAT.md (있다면)
□ MEMORY.md (있다면)
□ 로그 (openclaw logs --limit 50 출력)
□ 에러 메시지 전문
□ 이전 버전 파일 (수정 이력 있다면)
```

---

*OG 프로젝트 스냅샷 템플릿 v1.0 | OpenClaw Grand Master | 2026-03-03*
*github.com/littlebero/og-docs*
