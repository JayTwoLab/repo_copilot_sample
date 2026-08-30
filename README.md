# Copilot 설정

```
.github/
├── copilot-instructions.md          
├── copilotignore                    
├── PULL_REQUEST_TEMPLATE.md         
├── instructions/                    
│   ├── cmake.instructions.md        
│   ├── core-engine.instructions.md  
│   └── network-module.instructions.md
├── agents/                          
│   ├── memory-safety-reviewer.agent.md
│   └── cmake-architect.agent.md     
└── hooks/                           
    └── safety.json                  
```

<br />

## 파일 구성

---

### `.github/copilot-instructions.md`
- `Copilot Chat`과 코드 제안 시 저장소 전반에 적용할 규칙을 정의합니다.
- 프로젝트의 기술 스택, 코딩 표준, 아키텍처 규칙, 테스트 작성 방식 등을 자연어로 작성하면 Copilot이 응답 및 코드 생성 시 이를 참조합니다.

```markdown
<!-- .github/copilot-instructions.md 예시 -->
# GitHub Copilot Instructions

## Language & Framework
- TypeScript, Next.js (App Router), Tailwind CSS를 기반으로 코드를 작성합니다.
- 함수형 컴포넌트와 Hooks를 우선 사용합니다.

## Coding Style
- 세미콜론(;)을 사용하지 않습니다.
- 들여쓰기는 2칸(spaces)을 사용합니다.
- 파일 및 컴포넌트 이름은 파스칼 케이스(PascalCase)를 따릅니다.

## Testing & Documentation
- 모든 유틸리티 함수에는 Vitest를 이용한 단위 테스트를 작성합니다.
- 함수 상단에는 JSDoc 형식으로 설명을 추가합니다.
```

<br />

---

### `.github/copilotignore`
- 보안, 개인정보, 라이선스 등의 이유로 특정 경로의 코드를 Copilot이 읽지 못하도록 차단합니다.
- `.gitignore`와 동일한 패턴 매칭 방식을 사용

```ignore
# .github/copilotignore 예시

# 보안 및 환경 변수 관련
.env*
secrets/
certs/

# 대규모 데이터셋 또는 자동 생성 파일
data/*.csv
build/
dist/
```

<br />

---

### `.github/PULL_REQUEST_TEMPLATE.md`
- `Copilot`이 `PR`(`Pull Request`) 생성 시 설명(`PR Summary`)을 자동 작성할 때 `.github/pull_request_template.md`에 정의된 섹션과 체크리스트 형식을 그대로 인식하여 템플릿에 맞게 내용을 채워줍니다.

<br />

---

### `.github/instructions/`
- 규모가 큰 저장소에서는 단일 `copilot-instructions.md` 파일 대신 영역별로 지침 파일을 나누어 관리할 수 있습니다.
- 경로: `.github/instructions/**/*.instructions.md`
- 예시 구조
   - `.github/instructions/backend.instructions.md`: 백엔드/API 작성 규칙
   - `.github/instructions/frontend.instructions.md`: UI 컴포넌트 및 CSS 규칙
   - `.github/instructions/testing.instructions.md`: 테스트 코드 표준

<br />

---

### `.github/agents/`
- `VS Code`나 `Visual Studio`, `Copilot CLI`의 `Agent` 모드에서 불러와 사용할 역할 맞춤형 `AI` 에이전트를 정의할 수 있습니다.
   - 경로: `.github/agents/<agent-name>.agent.md`
   - 예시: `.github/agents/code-reviewer.agent.md`
```markdown
---
name: CodeReviewer
description: 보안 및 성능 관점에서 코드를 리뷰하는 에이전트
tools: ['read_file', 'list_dir']
---

당신은 시니어 보안 엔지니어입니다.
작성된 코드의 SQL Injection, XSS, 권한 검증 누락 여부를 중점적으로 검토하세요.
```

<br />

---

### `.github/hooks/`
- `Copilot Coding Agent`나 `CLI`가 동작할 때 특정 라이프사이클 이벤트(시작 전, 도구 실행 전/후 등)에 맞춰 셸 스크립트를 자동 실행하거나 위험 동작을 가로채는(`Intercept`) 미들웨어 역할을 설정합니다.
   - 경로: `.github/hooks/<hook-name>.json`
   - 예시: `.github/hooks/safety.json`
```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "bash": "./scripts/verify-command.sh"
      }
    ],
    "postToolUse": [
      {
        "type": "command",
        "bash": "npm run lint"
      }
    ]
  }
}
```

<br />

---

### `.github/copilot/skills/`
- `Copilot`이 반복적으로 수행해야 하는 특정 워크플로(예: 특정 배포 스크립트 실행, 사내 `API` 검증 등)를 `Agent Skill` 형태로 패키징해 저장소에 배치할 수 있습니다.
   - 경로: `.github/copilot/skills/<skill-name>/`

