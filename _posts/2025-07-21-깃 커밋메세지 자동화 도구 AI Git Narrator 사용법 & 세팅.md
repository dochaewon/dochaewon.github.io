# AI Git Narrator 사용법 & 세팅 정리 (macOS 전용 Git 커밋 메시지 자동 생성기)

**Git 커밋 메시지 작성, 아직도 직접 고민하시나요?**
OpenAI, Gemini, Ollama 등 다양한 AI 모델을 활용해 커밋 메시지와 PR 설명을 자동으로 생성해주는
**AI Git Narrator** 도구를 macOS에서 직접 설치해 사용해봤습니다.
오늘은 설치부터 사용법까지 꼼꼼히 정리해볼게요.

---

## ✅ AI Git Narrator란?

> AI Git Narrator는 macOS 전용 커맨드라인 도구로, GPT, Gemini, Ollama 등의 LLM을 활용해 Git 커밋 메시지 및 PR 설명을 자동 생성합니다.

* **지원 모델**: OpenAI GPT, Google Gemini, Ollama (로컬 LLM)
* **기능**:

  * 커밋 메시지 자동 생성 (전체, staged, unstaged 변경 기준)
  * Pull Request 설명 자동 생성
  * LLM 파라미터 커스터마이징 (model, temperature, max\_tokens 등)

---

## 🧩 설치 방법 (Homebrew 권장)

```bash
brew tap pmusolino/ai-git-narrator
brew install ai-git-narrator
```

설치 후 확인:

```bash
ai-git-narrator --help
```

---

## 🔐 API 키 준비

### OpenAI API 키 발급

1. [OpenAI Dashboard](https://platform.openai.com/account/api-keys) 접속
2. API Key 생성 → 복사 후 보관
3. 명령어에 `--api-key`로 전달

```bash
ai-git-narrator --api-key sk-xxx --generate-commit-message
```

### Gemini API 키 발급

1. [Google AI Studio](https://aistudio.google.com/) → 로그인
2. API Key 생성 후 복사
3. Gemini 모델 사용 시:

```bash
ai-git-narrator --llm-provider gemini --api-key Xxxx --generate-commit-message
```

---

## 🧠 Ollama 로컬 모델 사용하기

> **인터넷 없이**, **비용 없이**, **개인 정보 유출 걱정 없이** AI를 쓰고 싶다면 Ollama가 정답입니다.

### Ollama 설치 & 실행

```bash
brew install ollama
ollama pull llama3.2
```

### 실행 예시

```bash
ai-git-narrator --llm-provider ollama --model llama3.2 --generate-commit-message
```

---

## 🚀 사용 예시

### 전체 변경사항 기준 커밋 메시지 생성 (OpenAI)

```bash
ai-git-narrator --api-key sk-xxx --generate-commit-message
```

### Staged 파일만 커밋 메시지 생성 (Gemini)

```bash
ai-git-narrator --llm-provider gemini --api-key xxx --generate-staged-commit-message
```

### 로컬 모델 기반 PR 설명 생성 (Ollama)

```bash
ai-git-narrator --llm-provider ollama --model llama3.1:8b --generate-pr
```

---

## 🧷 자주 사용하는 옵션 정리

| 옵션                                         | 설명                                 |
| ------------------------------------------ | ---------------------------------- |
| `--llm-provider`                           | `openai`, `gemini`, `ollama` 중 택 1 |
| `--generate-commit-message`                | 전체 변경사항 기준 커밋 메시지                  |
| `--generate-staged-commit-message`         | `git add` 된 내용만 기준                 |
| `--generate-unstaged-commit-message`       | 변경은 되었으나 staged 안된 것 기준            |
| `--generate-pr`                            | PR 제목 & 설명 생성                      |
| `--model`, `--temperature`, `--max-tokens` | 모델 파라미터 설정                         |

---

## ✨ 직접 써본 후기

* 커밋 메시지가 **매우 자연스럽고 구체적**
* 로컬 Ollama 모델로도 **속도 빠르고 무료**
* PR 설명은 실제로 **코드 리뷰에 도움**이 되는 수준
* macOS에서만 동작한다는 점은 호불호가 갈릴 수 있음

---

## 🏷️ 해시태그

```
#Git #AI커밋 #OpenAI #Gemini #Ollama #macOS전용툴 #개발자동화 #SwiftCLI #개발생산성 #aigitnarrator
```
