# OpenClaw 신규 프로젝트 세팅 가이드

> 처음 시작할 때 한 번 읽고 세팅하면 된다.
> 모르는 게 생기면 OG에게: claude.ai → OG 프로젝트

작성: OG (OpenClaw Grand Master) | 2026-03-03

---

## 1. 환경 요구사항

```
□ Node.js v22 LTS 이상
  (v18/v20 — AsyncLocalStorage 미지원으로 침묵 실패)

□ OpenClaw v2026.2.26 이상

□ 에이전트 모델: 14B 이상 필수
  (8B 이하 — OpenClaw 시스템 프롬프트 17K 수용 불가)

□ Ollama 설치 완료 (로컬 모델 사용 시)
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

## 2. openclaw.json 기본 설정

아래를 기본값으로 사용한다.
**경로와 에이전트 ID는 실제 환경에 맞게 수정한다.**

```json
{
  "bindings": [
    { "agentId": "amy", "match": { "channel": "telegram" } }
  ],
  "commands": { "restart": true },
  "compaction": {
    "mode": "default",
    "reserveTokensFloor": 40000,
    "memoryFlush": {
      "enabled": true,
      "softThresholdTokens": 40000,
      "prompt": "Distill this session to memory/YYYY-MM-DD.md. Focus on decisions, state changes, lessons, blockers.",
      "systemPrompt": "Extract only what is worth remembering. No fluff."
    }
  },
  "session": { "reset": { "daily": false } },
  "skills": { "allowBundled": [] },
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true }
      }
    }
  },
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://127.0.0.1:11434",
        "apiKey": "ollama-local",
        "api": "ollama",
        "models": [
          {
            "id": "qwen2.5:14b",
            "contextWindow": 32768,
            "maxTokens": 8192,
            "reasoning": false,
            "cost": { "input": 0, "output": 0 }
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": { "primary": "ollama/qwen2.5:14b", "fallbacks": [] }
    },
    "list": [
      {
        "id": "amy",
        "model": "ollama/qwen2.5:14b",
        "tools": {
          "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"]
        }
      }
    ]
  }
}
```

> ⚠️ **`openclaw config set` 절대 사용 금지** (BUG-S — LaunchAgent 자동 해제)
> openclaw.json은 반드시 직접 편집한다.

---

## 3. SOUL.md 기본 구조

**규칙은 5개 이하.** 초과하면 준수율이 급격히 떨어진다.
가장 중요한 규칙은 **맨 마지막**에 배치한다 (recency effect).

```markdown
# [에이전트 이름] — [역할 한 줄]

## 정체성
나는 [이름]이다. [이 프로젝트에서 하는 일 2~3문장.]

## 핵심 규칙
1. [규칙] — 이유: [왜]
2. [규칙] — 이유: [왜]
3. [규칙] — 이유: [왜]
4. [규칙] (필요 시)
5. [가장 중요한 규칙] ← 항상 마지막

## 루프 종료
sessions_send 후 루프를 끝낼 때: REPLY_SKIP으로만 응답한다.
NO_REPLY는 절대 사용하지 않는다.

## 전문가 에스컬레이션
이 에이전트 혼자 해결하지 않는 것들:

**OG에게 가져갈 것:**
- OpenClaw 설정 변경, 버그 대응, 자동화 설계, 멀티에이전트 아키텍처
→ claude.ai에서 OG 프로젝트를 열고 질문한다.

**클마(CGM)에게 가져갈 것:**
- 클로드 프로젝트 시스템 프롬프트 수정, KB 구성, 클로드 활용 자문
→ claude.ai에서 CGM 프로젝트를 열고 질문한다.
```

---

## 4. 설정 변경 후 필수 절차

openclaw.json 또는 SOUL.md를 변경하면 반드시 이 순서로:

```bash
# ⚠️ openclaw gateway stop 절대 사용 금지 (BUG-R — LaunchAgent 해제)

launchctl stop gui/$UID/ai.openclaw.gateway
echo "✅ stop 완료 (LaunchAgent 유지됨)"

openclaw gateway start
echo "✅ start 완료"

# 텔레그램 또는 TUI에서 /new

openclaw logs --limit 50
echo "✅ 변경 반영 확인"
```

---

## 5. 멀티에이전트 추가 시 — 3종 세트

하나라도 빠지면 에러 없이 조용히 실패한다.

```
□ 1. openclaw.json — agentToAgent.enabled: true
□ 2. 라우터(Amy) tools.allow에 sessions 툴 추가
□ 3. SOUL.md에 대상 에이전트 세션 키 명시

세션 키 형식:
  agent:amy:main
  agent:scarlett:main
  agent:natalie:main
```

멀티에이전트 openclaw.json 추가:

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["amy", "scarlett", "natalie"]
    }
  },
  "agents": {
    "list": [
      {
        "id": "amy",
        "tools": {
          "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"]
        }
      },
      {
        "id": "scarlett",
        "tools": { "allow": ["group:fs", "group:runtime"] }
      },
      {
        "id": "natalie",
        "tools": { "allow": ["group:fs"] }
      }
    ]
  }
}
```

---

## 6. 뭔가 안 될 때 — 진단 순서

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

# 4. 채널 상태
openclaw channels status
echo "✅ 채널 확인"

# 5. 설정 원본 (요약본 신뢰 금지)
cat ~/.openclaw/openclaw.json
echo "✅ 설정 원본 확인"
```

멀티에이전트 전달 확인:
```bash
# sessions_send 툴콜 존재 여부
openclaw logs --limit 50 | grep "sessions_send"

# 올바른 lane 라우팅 여부
openclaw logs --limit 50 | grep "lane=session:agent:scarlett:main"

# 네이티브 API 사용 여부 (/v1/chat/completions면 BUG-A 위험)
openclaw logs --limit 50 | grep "/api/chat"
```

---

## 7. 절대 금지 명령어

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

## 8. 모르면 OG에게

이 가이드로 해결 안 되는 것은 OG에게 가져온다.

```
claude.ai → OG 프로젝트 열기

첨부:
  OG_project_snapshot_template.md (채워서)
  openclaw.json
  SOUL.md
  로그

첫 대화:
  안녕 OG, [프로젝트명]이야.
  스냅샷 채워서 가져왔어.
  문제: [증상 + 로그]
```

OG는 추측으로 답하지 않는다.
모르면 리서치하고, 검증된 명령어만 제시한다.

---

*OpenClaw 신규 프로젝트 세팅 가이드 v1.0 | OG (OpenClaw Grand Master) | 2026-03-03*
*github.com/littlebero/og-docs*
