# GitHub Copilot 설정 가이드

```text
project-root/
├── AGENTS.md                                  # AI 에이전트 공통 운영 매뉴얼
├── .vscode/
│   └── mcp.json                               # IDE용 MCP 서버 연동 설정
└── .github/
    ├── copilot-instructions.md                # 저장소 전역 지침
    ├── copilotignore                          # 컨텍스트 제외 설정 (보조 수단)
    ├── PULL_REQUEST_TEMPLATE.md               # PR 자동 요약 템플릿
    ├── workflows/
    │   └── copilot-setup-steps.yml            # Copilot Coding Agent 실행 환경 설정
    ├── instructions/                          # 파일/경로별 세부 지침
    │   ├── cmake.instructions.md
    │   ├── core-engine.instructions.md
    │   └── network-module.instructions.md
    ├── prompts/                               # 재사용 가능한 슬래시(/) 프롬프트
    │   └── security-audit.prompt.md
    ├── agents/                                # 역할 맞춤형 커스텀 에이전트
    │   ├── memory-safety-reviewer.agent.md
    │   └── cmake-architect.agent.md
    ├── skills/                                # 작업별 에이전트 스킬 패키지
    │   └── asan-crash-analyzer/
    │       ├── SKILL.md
    │       └── scripts/
    └── hooks/                                 # 에이전트 라이프사이클 훅
        └── safety.json
```

<br />

## 파일 구성 및 상세 역할

---

### 1. `.github/copilot-instructions.md`
- `Copilot Chat`, 인라인 코드 제안, `Copilot Code Review` 수행 시 저장소 전반에 기본으로 적용할 전역 규칙을 정의합니다.
- 프로젝트의 기술 스택, 코딩 표준, 아키텍처 원칙, 테스트 작성 방식 등을 자연어로 작성합니다.

```markdown
<!-- .github/copilot-instructions.md 예시 -->
# GitHub Copilot Instructions

## Language & Build System
- C++20 표준과 CMake (3.25 이상)를 기반으로 코드를 작성합니다.
- 모든 타겟은 Modern CMake 방식(`target_include_directories`, `target_link_libraries`)을 사용합니다.

## Coding Style & Memory Safety
- Raw 포인터(`new`/`delete`) 사용을 금지하며, `std::unique_ptr`와 `std::shared_ptr` 및 RAII 패턴을 준수합니다.
- 들여쓰기는 4칸(spaces)을 사용하며, 클래스명은 `PascalCase`, 함수 및 변수명은 `snake_case`를 따릅니다.

## Testing & Documentation
- 모든 핵심 로직에는 GoogleTest(GTest) 기반의 단위 테스트를 작성합니다.
- 공개 헤더(`.hpp`)의 클래스와 함수 상단에는 Doxygen 형식의 주석을 추가합니다.
```

<br />

---

### 2. `.github/instructions/*.instructions.md`
- 규모가 큰 저장소에서 단일 `copilot-instructions.md` 대신 **특정 폴더나 파일 확장자별로 지침을 분리**하여 관리할 때 사용합니다.
- 경로: `.github/instructions/**/*.instructions.md`
- **핵심 규칙**: 파일 최상단 YAML 프론트매터에 `applyTo` (Glob 패턴)를 지정해야 해당 파일 작업 시 지침이 자동 로드됩니다. `excludeAgent` 속성으로 특정 에이전트(`code-review` 또는 `coding-agent`)에서의 실행을 제외할 수도 있습니다.

```markdown
---
applyTo: "**/CMakeLists.txt,**/*.cmake"
description: "CMake 빌드 스크립트 작성 규칙"
---

# CMake 작성 지침
- 전역 설정(`include_directories`, `link_libraries`)을 절대 사용하지 마세요.
- 외부 라이브러리는 `FetchContent` 또는 `find_package(... CONFIG REQUIRED)`를 통해 가져옵니다.
- 컴파일러 플래그는 `target_compile_options()`에 `PRIVATE` 또는 `INTERFACE`를 명시하여 설정합니다.
```

<br />

---

### 3. `AGENTS.md`
- 저장소 루트(또는 모노레포의 각 서브프로젝트 폴더)에 배치하며, GitHub Copilot뿐만 아니라 다양한 AI 에이전트가 공통으로 읽는 **에이전트 전용 README(운영 매뉴얼)**입니다.
- 추상적인 설명보다 **실행 가능한 빌드/테스트 명령어**와 **절대 건드리면 안 되는 경계(Boundaries)**를 명확히 정의하는 것이 핵심입니다.

```markdown
# AGENTS.md

## Build & Test Commands
- 전체 빌드: `cmake -B build -DCMAKE_BUILD_TYPE=Debug && cmake --build build -j`
- 단위 테스트 실행: `ctest --test-dir build --output-on-failure`
- 메모리 검증(ASan): `cmake -B build-asan -DENABLE_ASAN=ON && cmake --build build-asan && ctest --test-dir build-asan`

## Boundaries (주의사항)
- `third_party/` 및 `certs/` 디렉터리 내부 파일은 절대 수정하지 마세요.
- 새로운 외부 의존성을 추가하기 전에는 반드시 사용자에게 먼저 확인을 받으세요.
```

<br />

---

### 4. `.github/prompts/*.prompt.md`
- IDE의 Copilot Chat에서 `/프롬프트이름` 형태의 슬래시 명령어로 즉시 호출할 수 있는 **재사용 가능한 작업 템플릿**입니다.
- 경로: `.github/prompts/<prompt-name>.prompt.md`

```markdown
---
mode: "agent"
tools: ["read_file", "search_code"]
description: "선택한 코드의 스레드 안전성 및 메모리 누수를 점검합니다."
---

현재 열려 있는 파일(`#${file}`)을 분석하여 다음 항목을 중점적으로 점검하고 개선 코드를 제안해 주세요:
1. 데이터 레이스(Data Race) 발생 가능성 및 `std::mutex` 잠금 범위의 적절성
2. 예외 발생 시 자원 누수 여부 (RAII 준수 확인)
3. 불필요한 깊은 복사(Deep Copy)가 발생하는 구간 (`std::move` 또는 `const&` 권장)
```

<br />

---

### 5. `.github/agents/*.agent.md`
- VS Code, Visual Studio, Copilot CLI, GitHub Cloud Agent에서 호출할 수 있는 **역할 맞춤형 전문가 AI 에이전트**를 정의합니다.
- 경로: `.github/agents/<agent-name>.agent.md`

```markdown
---
name: memory-safety-reviewer
description: C++ 메모리 안전성 및 동시성 이슈를 전문적으로 검토하는 에이전트
tools: ["read_file", "list_dir", "search_code"]
---

당신은 고성능 C++ 시스템 소프트웨어의 시니어 보안 및 안정성 엔지니어입니다.
작성된 코드에서 Dangling Pointer, Buffer Overflow, Deadlock, Use-After-Free 가능성을 최우선으로 탐지하고 수정 방안을 제시하세요.
```

<br />

---

### 6. `.github/skills/<skill-name>/SKILL.md`
- Copilot(클라우드 에이전트, 코드 리뷰, CLI, IDE 에이전트 모드)이 특정 도메인 작업을 수행할 때 **필요에 따라 온디맨드(On-demand)로 불러오는 워크플로·스크립트·리소스 패키지**입니다.
- 경로: `.github/skills/<skill-name>/SKILL.md` (폴더명과 `name` 필드는 소문자와 하이픈(`-`)만 사용)

```markdown
---
name: asan-crash-analyzer
description: AddressSanitizer(ASan) 로그를 분석하고 크래시 원인과 패치 코드를 제시할 때 사용합니다.
---

# ASan 크래시 분석 스킬

1. 제공된 ASan 스택 트레이스에서 크래시 유형(`heap-use-after-free`, `stack-buffer-overflow` 등)을 식별합니다.
2. `./scripts/parse_asan_log.py` 스크립트가 있다면 실행하여 심볼 위치를 추출합니다.
3. 할당 해제 지점과 재접근 지점의 수명 주기(Lifecycle) 충돌 원인을 설명하고 수정 패치를 제안합니다.
```

<br />

---

### 7. `.github/hooks/*.json`
- `Copilot Coding Agent`나 `Copilot CLI`가 동작할 때 특정 라이프사이클 이벤트(`sessionStart`, `preToolUse`, `postToolUse` 등)에 맞춰 셸 스크립트를 자동 실행하거나 위험 동작을 가로채는(`Intercept`) 미들웨어 역할을 수행합니다.
- 경로: `.github/hooks/<hook-name>.json`

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
        "bash": "clang-format -i"
      }
    ]
  }
}
```

<br />

---

### 8. `.github/workflows/copilot-setup-steps.yml`
- GitHub 이슈나 PR에서 동작하는 **Copilot Coding Agent(클라우드 에이전트)**가 코드를 빌드하고 테스트할 수 있도록 가상 환경(Runner)에 컴파일러, 패키지, 의존성을 사전 설치하는 전용 워크플로입니다.
- **주의사항**: Job의 이름은 반드시 `copilot-setup-steps`여야 Copilot이 인식합니다.

```yaml
name: "Copilot Setup Steps"

on:
  workflow_dispatch:
  push:
    paths:
      - .github/workflows/copilot-setup-steps.yml

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install C++ & CMake toolchain
        run: |
          sudo apt-get update
          sudo apt-get install -y cmake ninja-build clang clang-tidy
```

<br />

---

### 9. `.github/PULL_REQUEST_TEMPLATE.md`
- Copilot이 PR 요약(`Copilot PR Summary`)을 자동 생성하거나 Coding Agent가 PR을 올릴 때, 이 템플릿에 정의된 섹션과 체크리스트 구조를 인식하여 알맞게 내용을 채워줍니다.

```markdown
## 📌 Summary
<!-- Copilot will auto-fill a concise summary of changes here -->

## 📦 Affected Subprojects
- [ ] Core Engine
- [ ] Network Module
- [ ] CMake / Build Scripts
- [ ] Unit Tests

## 🔍 Key Changes
<!-- List specific architectural or logic modifications -->

## ⚠️ Memory & Thread-Safety Checklist
- [ ] 메모리 누수 위험이 없는지 (RAII, Smart Pointers 확인)
- [ ] 멀티스레드 동기화(Mutex/Lock)가 적절한지
- [ ] CMake Target 가시성(PUBLIC/PRIVATE)이 올바르게 설정되었는지

## 🧪 Testing Performed
- [ ] CTest 실행 통과
- [ ] AddressSanitizer (ASan) / ThreadSanitizer (TSan) 검증 여부
```

<br />

---

### 10. `.github/copilotignore` 및 Content Exclusion
- 보안, 개인정보, 라이선스 등의 이유로 특정 경로의 파일을 Copilot 컨텍스트에서 제외할 때 사용합니다.
- **실무 팁**: 로컬 파일(`.copilotignore`)만으로는 IDE에 직접 열려 있는 파일의 참조를 완벽히 차단하기 어렵습니다. 보안이 중요한 파일은 반드시 **GitHub 저장소 또는 Organization 설정(`Settings` → `Copilot` → `Content exclusion`)**에서 경로를 등록해 서버/클라이언트 정책 레벨에서 차단하는 것을 권장합니다.

```ignore
# 보안 및 환경 변수 관련
.env*
secrets/
certs/

# 대규모 데이터셋 또는 빌드 산출물
data/*.csv
build/
dist/
```

<br />

---

### 11. `.vscode/mcp.json` (MCP 서버 연동)
- Copilot Agent 모드에서 외부 도구(사내 위키, 이슈 트래커, 데이터베이스, 브라우저 제어 등)를 호출할 수 있도록 **Model Context Protocol(MCP)** 서버를 프로젝트 단위로 설정합니다.
- GitHub 웹 환경(Coding Agent 및 Code Review)에서 사용할 MCP 서버는 저장소의 `Settings` → `Copilot` → `MCP servers`에서 설정합니다.

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "[https://api.githubcopilot.com/mcp/](https://api.githubcopilot.com/mcp/)"
    }
  }
}
```
