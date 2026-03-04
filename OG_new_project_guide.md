# OpenClaw 신규 프로젝트 세팅 가이드

> 처음 시작할 때 한 번 읽고 세팅하면 된다.
> 이 가이드는 특정 사양을 강요하지 않는다. 환경에 맞는 선택 기준을 제시한다.
> 모르는 게 생기면 OG에게: claude.ai → OG 프로젝트

작성: OG (OpenClaw Grand Master) | 2026-03-03

---

## 1. 환경 요구사항 — 최소 기준

사양과 무관하게 아래는 반드시 충족해야 한다.

```
□ Node.js v22 LTS 이상
  (v18/v20 — AsyncLocalStorage 미지원으로 침묵 실패)

□ OpenClaw v2026.2.26 이상

□ 에이전트 모델: 14B 이상 필수
  (8B 이하 — OpenClaw 시스템 프롬프트 17K 수용 불가. 에러 없이 침묵 실패.)
```

설치 확인:
```bash
openclaw --version
echo "✅ 버전 확인"

openclaw gateway status
echo "✅ Gateway 확인"

openclaw health
echo "✅ 헬스 확인"
```

---

## 2. 모델 선택 기준

로컬 모델과 클라우드 모델 중 선택한다.

### 로컬 모델 (Ollama)

| 램 | 권장 모델 | 비고 |
|----|----------|------|
| 16GB | qwen2.5:14b-q4 | 기본 에이전트 작업 가능 |
| 24GB | qwen2.5:14b | 여유 있게 운영 가능 |
| 32GB+ | qwen2.5:32b 또는 32B급 | 복잡한 파이프라인 가능 |

> ⚠️ Ollama 기본 컨텍스트는 2048이다. 반드시 `contextWindow` 명시 필요.
> 추천값: 16384 (보수적) ~ 32768 (여유 있는 환경)

### 클라우드 모델 (Anthropic / OpenRouter 등)

로컬 GPU가 없거나 성능이 중요한 경우 선택.
- `anthropic/claude-sonnet-4-5` — 균형형
- `anthropic/claude-opus-4` — 고성능, 비용 높음
- 비용 관리가 필요하면 클라우드 primary + 로컬 fallback 조합 검토

### 선택 기준 요약

```
로컬 only       → 비용 0, 속도는 사양에 따라 다름, 인터넷 불필요
클라우드 only   → 비용 발생, 속도 빠름, 인터넷 필요
혼합 (권장)     → 클라우드 primary + 로컬 fallback → 비용 절감 + 안정성
```

---

## 3. 에이전트 구조 선택

### 단일 에이전트

용도가 단순하거나 처음 시작하는 경우 권장.

```
장점: 설정 단순, 디버깅 쉬움, 안정성 높음
단점: 역할 분리 불가, 복잡한 파이프라인 어려움
적합: CEO 인텔리전스, 단순 자동화, 개인 비서
```

### 멀티에이전트

여러 역할을 분리해서 운영해야 할 때.

```
장점: 역할 분리, 병렬 처리 가능, 전문화
단점: 설정 복잡, sessions_send 디버깅 필요
적합: 수집 + 분석 + 리포트 파이프라인, 대규모 자동화
주의: 처음 시작이라면 단일 에이전트로 먼저 안정화 후 전환 권장
```

멀티에이전트 선택 시 → **반드시 3종 세트 동시 설정:**
```
□ agentToAgent.enabled: true
□ 라우터 에이전트 tools.allow에 sessions 툴 추가
□ SOUL.md에 대상 에이전트 세션 키 명시 (예: agent:scarlett:main)
하나라도 빠지면 에러 없이 조용히 실패한다.
```

---

## 4. openclaw.json 핵심 설정

전체 설정을 나열하지 않는다. **프로젝트마다 반드시 결정해야 할 항목**만 다룬다.

### 4-1. compaction (컨텍스트 보호)

장시간 운영하는 프로젝트라면 필수. 설정 안 하면 컨텍스트 소실 위험.

```json
"compaction": {
  "mode": "default",
  "reserveTokensFloor": 40000,
  "memoryFlush": {
    "enabled": true,
    "softThresholdTokens": 40000,
    "prompt": "Distill this session to memory/YYYY-MM-DD.md. Focus on decisions, state changes, lessons, blockers.",
    "systemPrompt": "Extract only what is worth remembering. No fluff."
  }
}
```

> `compaction`은 루트 레벨 키다. `agents.defaults` 안에 넣지 않는다.

### 4-2. session.reset.daily

야간 자동화 파이프라인이 있다면 반드시 false로.

```json
"session": { "reset": { "daily": false } }
```

### 4-3. api 타입 (로컬 Ollama 사용 시)

```json
"api": "ollama"
```

네이티브 `/api/chat` 사용. 빠지면 OpenAI 호환 레이어 사용 → tool calling 버그 위험.

### 4-4. contextWindow (로컬 모델 사용 시)

```json
"contextWindow": 32768
```

명시하지 않으면 Ollama 기본값 2048로 동작. 반드시 명시.

### 4-5. session-memory hook

/new 실행 시 세션 자동 저장. compaction 안전망으로 권장.

```json
"hooks": {
  "internal": {
    "enabled": true,
    "entries": {
      "session-memory": { "enabled": true }
    }
  }
}
```

---

## 5. SOUL.md 작성 원칙

규칙 수와 배치가 준수율을 결정한다.

```
□ 규칙은 5개 이하 (10개 이상 → 준수율 15% 이하)
□ 가장 중요한 규칙은 맨 마지막에 배치 (recency effect)
□ "~하지 마라" → "대신 ~해라" 긍정형으로
□ 규칙마다 이유 1문장 추가 → 준수율 대폭 향상
```

에스컬레이션 블록은 모든 에이전트 SOUL.md 맨 하단에 필수:

```markdown
## 전문가 에스컬레이션

**OG에게 가져갈 것:**
- OpenClaw 설정 변경, 버그 대응, 자동화 설계, 멀티에이전트 아키텍처
→ claude.ai에서 OG 프로젝트를 열고 질문한다.

**클마(CGM)에게 가져갈 것:**
- 클로드 프로젝트 시스템 프롬프트 수정, KB 구성, 클로드 활용 자문
→ claude.ai에서 CGM 프로젝트를 열고 질문한다.
```

sessions_send를 사용하는 에이전트라면 추가:

```markdown
## 루프 종료
루프를 끝낼 때: REPLY_SKIP으로만 응답한다.
NO_REPLY는 절대 사용하지 않는다.
```

---

## 6. 설정 변경 후 필수 절차

openclaw.json 또는 SOUL.md를 변경하면 반드시 이 순서로:

```bash
# ⚠️ openclaw gateway stop 절대 사용 금지 (BUG-R — LaunchAgent 해제)
# ⚠️ openclaw config set 절대 사용 금지 (BUG-S — LaunchAgent 자동 해제)

launchctl stop gui/$UID/ai.openclaw.gateway
echo "✅ stop 완료 (LaunchAgent 유지됨)"

openclaw gateway start
echo "✅ start 완료"

# 텔레그램 또는 TUI에서 /new

openclaw logs --limit 50
echo "✅ 변경 반영 확인"
```

> Linux (systemd) 환경이라면 `launchctl` 대신 `systemctl --user stop openclaw-gateway`

---

## 7. 뭔가 안 될 때 — 진단 순서

```bash
# 1. 로그 — 항상 여기서 시작
openclaw logs --limit 50
echo "✅ 로그 확인"

# 2. Gateway 상태
openclaw gateway status
echo "✅ Gateway 확인"

# 3. 헬스
openclaw health
echo "✅ 헬스 확인"

# 4. 설정 원본 (요약본 신뢰 금지)
cat ~/.openclaw/openclaw.json
echo "✅ 설정 원본 확인"
```

멀티에이전트 전달 확인:
```bash
openclaw logs --limit 50 | grep "sessions_send"
openclaw logs --limit 50 | grep "lane=session:agent"
openclaw logs --limit 50 | grep "/api/chat"
# /v1/chat/completions가 나오면 api: "ollama" 설정 누락
```

---

## 8. 절대 금지

| 명령어 | 이유 | 대안 |
|--------|------|------|
| `openclaw gateway stop` | BUG-R — LaunchAgent 해제 | `launchctl stop gui/$UID/ai.openclaw.gateway` |
| `openclaw gateway restart` | 상태 꼬임 | launchctl stop → gateway start |
| `openclaw config set` | BUG-S — LaunchAgent 자동 해제 | openclaw.json 직접 편집 |
| `openclaw channels login telegram` | 작동 안 함 | `config set channels.telegram.botToken` |
| 8B 이하 모델 agent work | contextWindow 부족 | 14B 이상 필수 |
| SOUL.md 규칙 10개+ | 준수율 15% 이하 | 5개 이하로 압축 |
| sessions_send에서 NO_REPLY | BUG-O — 무한 핑퐁 | REPLY_SKIP만 사용 |

---

## 9. 모르면 OG에게

이 가이드로 해결 안 되는 것은 OG에게 가져온다.

```
claude.ai → OG 프로젝트 열기

첨부:
  OG_project_snapshot_template.md (채워서)
  openclaw.json
  SOUL.md
  로그 (openclaw logs --limit 50)
```

OG는 추측으로 답하지 않는다.
모르면 리서치하고, 검증된 명령어만 제시한다.

---

*OpenClaw 신규 프로젝트 세팅 가이드 v1.1 | OG (OpenClaw Grand Master) | 2026-03-03*
*github.com/littlebero/og-docs*
