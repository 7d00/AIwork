# Ollama 명령어 레퍼런스

> 이 PC에는 아직 Ollama가 설치되어 있지 않습니다. 설치 후 `ollama --version`으로 버전을 확인하세요.
> 모든 명령어의 상세 옵션은 `ollama help <명령어>` 또는 `ollama <명령어> --help`로 볼 수 있습니다.

Ollama는 Llama, Gemma, Qwen, DeepSeek, Mistral 같은 **오픈 LLM을 내 컴퓨터에서 바로 실행**하게 해 주는 도구입니다.
모델 다운로드, 실행, 로컬 API 서버(`http://localhost:11434`)까지 한 번에 제공합니다.

| 하고 싶은 일 | Ollama 명령어 |
|---|---|
| 모델 내려받기 | `ollama pull` |
| 모델과 대화하기 | `ollama run` |
| 설치된 모델 보기 | `ollama list` |
| 실행 중인 모델 보기 | `ollama ps` |
| 내 설정으로 모델 만들기 | `ollama create` (Modelfile) |
| 프로그램에서 사용하기 | REST API / OpenAI 호환 API / `ollama` Python 라이브러리 |

---

## 목차

1. [설치와 업데이트](#1-설치와-업데이트)
2. [모델 관리](#2-모델-관리)
3. [모델 실행 (`ollama run`)](#3-모델-실행-ollama-run)
4. [대화 중 명령어 (`/`)](#4-대화-중-명령어-)
5. [서버 (`ollama serve`)](#5-서버-ollama-serve)
6. [Modelfile로 나만의 모델 만들기](#6-modelfile로-나만의-모델-만들기)
7. [REST API](#7-rest-api)
8. [OpenAI 호환 API](#8-openai-호환-api)
9. [Python에서 사용하기](#9-python에서-사용하기)
10. [환경 변수](#10-환경-변수)
11. [모델 이름 규칙과 추천 모델](#11-모델-이름-규칙과-추천-모델)
12. [자주 쓰는 작업 흐름](#12-자주-쓰는-작업-흐름)
13. [자주 만나는 문제와 해결](#13-자주-만나는-문제와-해결)

---

## 1. 설치와 업데이트

```powershell
# Windows (winget)
winget install --id Ollama.Ollama -e

# 또는 https://ollama.com/download 에서 설치 파일 다운로드

# macOS
brew install ollama

# Linux
curl -fsSL https://ollama.com/install.sh | sh
```

> 💡 Windows·macOS에서는 설치하면 Ollama 앱이 백그라운드(트레이 아이콘)에서 서버를 자동으로 실행합니다.
> ⚠️ 설치 직후 `ollama` 명령을 찾을 수 없다고 나오면 **VS Code나 터미널을 완전히 닫았다가 다시 여세요.**

| 명령어 | 설명 |
|---|---|
| `ollama --version` | Ollama 버전 확인 |
| `ollama help` | 전체 명령어 목록 |
| `ollama help <명령어>` | 특정 명령어의 상세 도움말 |
| `winget upgrade Ollama.Ollama` | 최신 버전으로 업데이트 (Windows) |

---

## 2. 모델 관리

| 명령어 | 설명 |
|---|---|
| `ollama pull llama3.2` | 모델 내려받기 (태그 생략 시 `:latest`) |
| `ollama pull qwen3:8b` | 특정 크기(태그)로 내려받기 |
| `ollama pull llama3.2` (다시 실행) | 이미 있는 모델을 최신으로 업데이트 (바뀐 부분만 받음) |
| `ollama list` (= `ollama ls`) | 설치된 모델 목록과 용량 |
| `ollama show llama3.2` | 모델 정보 (파라미터 수, 컨텍스트 길이, 양자화 등) |
| `ollama show llama3.2 --modelfile` | 모델의 Modelfile 보기 |
| `ollama show llama3.2 --parameters` | 기본 파라미터 보기 |
| `ollama show llama3.2 --template` | 프롬프트 템플릿 보기 |
| `ollama show llama3.2 --system` | 시스템 프롬프트 보기 |
| `ollama cp llama3.2 my-llama` | 모델 복사 (새 이름으로) |
| `ollama rm llama3.2` | 모델 삭제 (디스크 공간 확보) |
| `ollama push 아이디/my-model` | 내 모델을 ollama.com에 업로드 (계정 필요) |

> 💡 모델 파일은 기본적으로 `C:\Users\<사용자>\.ollama\models`에 저장됩니다. 모델 하나가 수 GB이므로 디스크 공간을 확인하세요.

---

## 3. 모델 실행 (`ollama run`)

| 명령어 | 설명 |
|---|---|
| `ollama run llama3.2` | 대화 모드 시작 (모델이 없으면 자동으로 pull) |
| `ollama run llama3.2 "하늘은 왜 파래?"` | 질문 한 번만 하고 답변 출력 후 종료 |
| `Get-Content 글.txt \| ollama run llama3.2 "요약해줘"` | 파일 내용을 넘겨서 처리 (PowerShell) |
| `ollama run llava "이 사진 설명해줘 ./photo.jpg"` | 이미지 입력 (비전 모델) |
| `ollama run llama3.2 --verbose` | 답변 후 속도(토큰/초) 등 통계 표시 |
| `ollama run llama3.2 --format json "..."` | JSON 형식으로만 답변 |
| `ollama run llama3.2 --keepalive 30m` | 대화 후 모델을 메모리에 30분 유지 |
| `ollama run qwen3 --think=false` | 생각(thinking) 과정 끄기 (추론 모델) |
| `ollama run qwen3 --hidethinking` | 생각 과정은 하되 화면에는 숨기기 |
| `ollama ps` | 지금 메모리에 올라가 있는 모델, CPU/GPU 사용 비율 |
| `ollama stop llama3.2` | 실행 중인 모델을 메모리에서 내리기 |

> 💡 `ollama ps`의 `PROCESSOR` 항목이 `100% GPU`면 GPU로 빠르게 실행 중, `CPU`가 섞여 있으면 메모리가 부족해 느려진 상태입니다.

---

## 4. 대화 중 명령어 (`/`)

`ollama run`으로 대화 모드에 들어가면 `>>>` 프롬프트에서 다음 명령을 쓸 수 있습니다.

| 명령어 | 설명 |
|---|---|
| `/?` 또는 `/help` | 사용 가능한 명령 목록 |
| `/bye` | 대화 종료 (`Ctrl + D`도 가능) |
| `/clear` | 대화 기록(문맥) 지우기 |
| `/show info` | 현재 모델 정보 |
| `/set system "너는 친절한 한국어 선생님이야"` | 시스템 프롬프트 설정 |
| `/set parameter temperature 0.2` | 파라미터 변경 (낮을수록 일관된 답변) |
| `/set parameter num_ctx 8192` | 컨텍스트 길이 변경 |
| `/set verbose` | 답변마다 속도 통계 표시 |
| `/set format json` | JSON으로만 답변 |
| `/set think` / `/set nothink` | 생각 과정 켜기/끄기 (추론 모델) |
| `/save my-assistant` | 현재 설정(시스템 프롬프트 등)을 새 모델로 저장 |
| `/load llama3.2` | 다른 모델로 바꾸기 |
| `"""` | 여러 줄 입력 시작·끝 (`"""`로 감싸기) |

> 💡 `Ctrl + C`는 답변 생성 중단, `↑` 키는 이전 입력 불러오기입니다.

---

## 5. 서버 (`ollama serve`)

Ollama는 모든 기능을 로컬 서버(`http://localhost:11434`)를 통해 처리합니다.
Windows·macOS 앱은 이 서버를 자동으로 띄우므로 보통은 직접 실행할 필요가 없습니다.

| 명령어 | 설명 |
|---|---|
| `ollama serve` | 서버를 직접 실행 (앱이 꺼져 있거나 Linux일 때) |
| `curl http://localhost:11434` | 서버 동작 확인 (`Ollama is running` 출력) |
| `curl http://localhost:11434/api/version` | 서버 버전 확인 |

> ⚠️ 앱이 이미 서버를 실행 중인데 `ollama serve`를 또 실행하면 `address already in use` 오류가 납니다. 트레이 아이콘에서 Ollama를 종료한 뒤 실행하세요.

---

## 6. Modelfile로 나만의 모델 만들기

`Modelfile`이라는 텍스트 파일에 기본 모델·시스템 프롬프트·파라미터를 적어 **나만의 모델**을 만들 수 있습니다.

```dockerfile
# Modelfile
FROM llama3.2

# 파라미터
PARAMETER temperature 0.3
PARAMETER num_ctx 8192

# 시스템 프롬프트
SYSTEM """
너는 Python을 가르치는 친절한 선생님이야.
항상 한국어로, 예제 코드와 함께 설명해.
"""
```

```powershell
ollama create py-teacher -f Modelfile   # 모델 만들기
ollama run py-teacher                   # 실행
```

### Modelfile 지시어

| 지시어 | 설명 |
|---|---|
| `FROM` | 기본 모델 (필수). 모델 이름 또는 `.gguf` 파일 경로 |
| `PARAMETER` | 실행 파라미터 설정 |
| `SYSTEM` | 시스템 프롬프트 |
| `TEMPLATE` | 프롬프트 템플릿 (Go 템플릿 문법, 보통은 기본값 사용) |
| `ADAPTER` | LoRA 어댑터 파일 경로 |
| `MESSAGE` | 예시 대화 (`MESSAGE user ...`, `MESSAGE assistant ...`) |
| `LICENSE` | 라이선스 문구 |

### 자주 쓰는 `PARAMETER`

| 파라미터 | 설명 |
|---|---|
| `temperature` | 창의성 (0 ~ 2, 낮을수록 일관됨, 기본 약 0.8) |
| `num_ctx` | 컨텍스트 길이(토큰). 크게 하면 긴 문서를 다루지만 메모리를 더 씀 |
| `top_p` / `top_k` | 다음 단어 후보를 고르는 범위 |
| `repeat_penalty` | 같은 말 반복을 줄임 |
| `seed` | 고정하면 같은 질문에 같은 답 |
| `stop` | 이 문자열이 나오면 생성 중단 |
| `num_predict` | 최대 생성 토큰 수 |

### GGUF 파일 가져오기 (Hugging Face 등)

```dockerfile
FROM ./my-model-Q4_K_M.gguf
```

```powershell
ollama create my-model -f Modelfile

# 또는 Hugging Face의 GGUF 저장소에서 바로 실행
ollama run hf.co/<사용자>/<저장소>:Q4_K_M
```

| 명령어 | 설명 |
|---|---|
| `ollama create my-model -f Modelfile` | Modelfile로 모델 생성 |
| `ollama create my-model -f Modelfile -q q4_K_M` | 생성하면서 양자화 (FP16 모델을 작게) |

---

## 7. REST API

서버가 켜져 있으면 어떤 언어에서든 HTTP로 사용할 수 있습니다. 기본 주소: `http://localhost:11434`

| 엔드포인트 | 설명 |
|---|---|
| `POST /api/generate` | 프롬프트 하나로 답변 생성 |
| `POST /api/chat` | 대화(메시지 목록)로 답변 생성 |
| `POST /api/embed` | 텍스트 임베딩(벡터) 생성 |
| `GET /api/tags` | 설치된 모델 목록 (`ollama list`) |
| `GET /api/ps` | 실행 중인 모델 (`ollama ps`) |
| `POST /api/show` | 모델 정보 (`ollama show`) |
| `POST /api/pull` | 모델 내려받기 |
| `POST /api/create` | 모델 생성 |
| `POST /api/copy` | 모델 복사 |
| `DELETE /api/delete` | 모델 삭제 |
| `GET /api/version` | 서버 버전 |

### 예시 (PowerShell)

```powershell
# 채팅 (stream: false → 답변을 한 번에 받음)
$body = @{
  model    = "llama3.2"
  messages = @(@{ role = "user"; content = "안녕? 자기소개 해줘" })
  stream   = $false
} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Uri http://localhost:11434/api/chat -Method Post -Body $body -ContentType "application/json; charset=utf-8"
```

### 예시 (curl)

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "하늘은 왜 파래?",
  "stream": false
}'
```

자주 쓰는 요청 옵션:

| 옵션 | 설명 |
|---|---|
| `"stream": false` | 답변을 조각이 아니라 한 번에 받기 (기본값은 `true`) |
| `"format": "json"` | JSON으로 답변. JSON 스키마 객체를 넣으면 그 구조대로 답변 (구조화 출력) |
| `"options": {"temperature": 0.2, "num_ctx": 8192}` | 파라미터 지정 |
| `"keep_alive": "10m"` | 요청 후 모델을 메모리에 유지할 시간 (`0`이면 바로 내림, `-1`이면 계속 유지) |
| `"think": true` | 생각 과정 포함 (추론 모델) |
| `"images": ["<base64>"]` | 이미지 입력 (비전 모델) |
| `"tools": [...]` | 도구(함수) 호출 정의 (도구 지원 모델) |

---

## 8. OpenAI 호환 API

Ollama는 OpenAI API와 같은 형식의 엔드포인트(`/v1/...`)도 제공합니다.
**OpenAI SDK나 OpenAI용으로 만든 프로그램을 주소만 바꿔서** 로컬 모델로 돌릴 수 있습니다.

| 엔드포인트 | 설명 |
|---|---|
| `POST /v1/chat/completions` | 채팅 |
| `POST /v1/completions` | 텍스트 완성 |
| `POST /v1/embeddings` | 임베딩 |
| `GET /v1/models` | 모델 목록 |

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama",   # 아무 값이나 넣으면 됨 (필수 항목이라 형식상 입력)
)

response = client.chat.completions.create(
    model="llama3.2",
    messages=[{"role": "user", "content": "파이썬 리스트 컴프리헨션 예제 보여줘"}],
)
print(response.choices[0].message.content)
```

---

## 9. Python에서 사용하기

공식 라이브러리 `ollama`를 사용합니다.

```powershell
uv add ollama          # uv 프로젝트
# 또는
pip install ollama
```

### 기본 채팅

```python
import ollama

response = ollama.chat(
    model="llama3.2",
    messages=[{"role": "user", "content": "하늘은 왜 파래?"}],
)
print(response.message.content)
```

### 스트리밍 (글자가 나오는 대로 출력)

```python
import ollama

stream = ollama.chat(
    model="llama3.2",
    messages=[{"role": "user", "content": "짧은 시 하나 써줘"}],
    stream=True,
)
for chunk in stream:
    print(chunk.message.content, end="", flush=True)
```

### 대화 이어가기

```python
import ollama

messages = [{"role": "system", "content": "너는 친절한 한국어 비서야."}]

while True:
    user = input("나: ")
    if user == "exit":
        break
    messages.append({"role": "user", "content": user})
    reply = ollama.chat(model="llama3.2", messages=messages).message.content
    messages.append({"role": "assistant", "content": reply})
    print("AI:", reply)
```

### 구조화 출력 (Pydantic)

```python
from pydantic import BaseModel
import ollama

class Country(BaseModel):
    name: str
    capital: str
    population: int

response = ollama.chat(
    model="llama3.2",
    messages=[{"role": "user", "content": "대한민국 정보를 알려줘"}],
    format=Country.model_json_schema(),
)
country = Country.model_validate_json(response.message.content)
print(country)
```

### 임베딩

```python
import ollama

result = ollama.embed(model="nomic-embed-text", input=["첫 번째 문장", "두 번째 문장"])
print(len(result.embeddings), len(result.embeddings[0]))   # 문장 수, 벡터 차원
```

### 그 밖의 함수

| 함수 | 설명 |
|---|---|
| `ollama.generate(model, prompt)` | 프롬프트 하나로 생성 |
| `ollama.list()` | 설치된 모델 목록 |
| `ollama.pull("llama3.2")` | 모델 내려받기 |
| `ollama.show("llama3.2")` | 모델 정보 |
| `ollama.ps()` | 실행 중인 모델 |
| `ollama.delete("llama3.2")` | 모델 삭제 |
| `ollama.Client(host="http://다른PC:11434")` | 다른 컴퓨터의 Ollama 서버에 연결 |
| `ollama.AsyncClient()` | 비동기(`async`/`await`) 사용 |

---

## 10. 환경 변수

Windows에서는 **설정 → 시스템 → 정보 → 고급 시스템 설정 → 환경 변수**에서 추가하거나 아래처럼 설정한 뒤, **Ollama 앱을 종료했다가 다시 실행**해야 적용됩니다.

```powershell
[Environment]::SetEnvironmentVariable("OLLAMA_MODELS", "D:\ollama\models", "User")
```

| 환경 변수 | 설명 |
|---|---|
| `OLLAMA_MODELS` | 모델 저장 폴더 변경 (C 드라이브 공간이 부족할 때) |
| `OLLAMA_HOST` | 서버 주소·포트 (기본 `127.0.0.1:11434`). `0.0.0.0`이면 다른 PC에서도 접속 가능 |
| `OLLAMA_CONTEXT_LENGTH` | 기본 컨텍스트 길이 |
| `OLLAMA_KEEP_ALIVE` | 모델을 메모리에 유지하는 기본 시간 (기본 `5m`) |
| `OLLAMA_MAX_LOADED_MODELS` | 동시에 메모리에 올릴 수 있는 모델 수 |
| `OLLAMA_NUM_PARALLEL` | 모델 하나가 동시에 처리하는 요청 수 |
| `OLLAMA_ORIGINS` | 접속을 허용할 웹 주소(CORS) |
| `OLLAMA_FLASH_ATTENTION=1` | Flash Attention 사용 (메모리 절약) |
| `OLLAMA_DEBUG=1` | 디버그 로그 출력 |

> ⚠️ `OLLAMA_HOST=0.0.0.0`으로 열면 같은 네트워크의 누구나 접속할 수 있습니다. 공용 네트워크에서는 주의하세요.

---

## 11. 모델 이름 규칙과 추천 모델

모델 이름은 `이름:태그` 형식입니다. 태그는 보통 **크기**와 **양자화** 방식을 뜻합니다.

```
qwen3:8b            # 80억(8B) 파라미터
llama3.2:3b         # 30억 파라미터
gemma3:12b-it-q4_K_M   # 12B, 명령 튜닝(it), 4비트 양자화
```

| 표기 | 뜻 |
|---|---|
| `1b`, `3b`, `8b`, `70b` | 파라미터 수 (클수록 똑똑하지만 무겁고 느림) |
| `q4_K_M`, `q8_0`, `fp16` | 양자화 수준 (숫자가 작을수록 가볍고 품질은 약간 낮음, 기본은 보통 `q4_K_M`) |
| `instruct`, `it` | 대화·지시 따르기용으로 튜닝된 버전 |
| `latest` | 태그 생략 시 기본값 |

### 메모리별 대략적인 모델 크기 (4비트 양자화 기준)

| 사용 가능한 GPU 메모리(VRAM) 또는 RAM | 적당한 모델 크기 |
|---|---|
| 8 GB | 1B ~ 8B |
| 16 GB | 8B ~ 14B |
| 24 GB | 14B ~ 32B |
| 48 GB 이상 | 70B |

### 용도별 모델 예시

| 용도 | 모델 예시 |
|---|---|
| 일반 대화 | `llama3.2`, `gemma3`, `qwen3`, `mistral` |
| 추론(생각하는 모델) | `deepseek-r1`, `qwen3` |
| 코딩 | `qwen2.5-coder`, `codellama` |
| 이미지 이해(비전) | `llava`, `gemma3`, `llama3.2-vision` |
| 임베딩(RAG, 검색) | `nomic-embed-text`, `mxbai-embed-large`, `bge-m3` |

> 💡 전체 모델 목록과 태그는 <https://ollama.com/library>에서 검색하세요. 모델마다 라이선스가 다르니 상업적으로 쓸 때는 확인이 필요합니다.

---

## 12. 자주 쓰는 작업 흐름

### 처음 시작하기

```powershell
ollama --version              # 설치 확인
ollama pull llama3.2          # 작은 모델부터 받아 보기
ollama run llama3.2           # 대화해 보기 (/bye로 종료)
ollama ps                     # GPU로 돌고 있는지 확인
```

### uv 프로젝트에서 Ollama 쓰기

```powershell
uv init ollama-test
cd ollama-test
uv add ollama
# main.py 작성 (9. Python에서 사용하기 참고)
uv run main.py
```

### 나만의 비서 모델 만들기

```powershell
# Modelfile 작성 (6. Modelfile 참고)
ollama create my-assistant -f Modelfile
ollama run my-assistant
```

### OpenAI용 코드를 로컬 모델로 바꾸기

```python
# 기존
client = OpenAI()
# 변경
client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")
# model="gpt-..." 부분을 model="llama3.2" 처럼 Ollama 모델 이름으로 변경
```

### 간단한 RAG (문서 검색 + 답변)

```python
import ollama

docs = ["Ollama는 로컬에서 LLM을 실행하는 도구다.", "uv는 Python 패키지 관리 도구다."]
doc_vecs = ollama.embed(model="nomic-embed-text", input=docs).embeddings

question = "Ollama가 뭐야?"
q_vec = ollama.embed(model="nomic-embed-text", input=question).embeddings[0]

# 코사인 유사도로 가장 비슷한 문서 찾기
def cos(a, b):
    dot = sum(x * y for x, y in zip(a, b))
    return dot / (sum(x * x for x in a) ** 0.5 * sum(y * y for y in b) ** 0.5)

best = max(range(len(docs)), key=lambda i: cos(q_vec, doc_vecs[i]))

answer = ollama.chat(
    model="llama3.2",
    messages=[{"role": "user", "content": f"다음 자료를 참고해 답해줘.\n자료: {docs[best]}\n질문: {question}"}],
)
print(answer.message.content)
```

### 디스크 정리

```powershell
ollama list                   # 용량 확인
ollama rm 안쓰는모델           # 삭제
```

---

## 13. 자주 만나는 문제와 해결

| 증상·오류 | 원인과 해결 |
|---|---|
| `'ollama' is not recognized...` | 설치 후 PATH 미반영 → VS Code·터미널을 완전히 다시 시작 |
| `could not connect to ollama app, is it running?` | 서버가 꺼져 있음 → Ollama 앱 실행 또는 `ollama serve` |
| `address already in use` / `bind: Only one usage...` | 서버가 이미 실행 중 → 그대로 사용하거나 트레이에서 종료 후 `ollama serve` |
| `model "xxx" not found` | 모델 미설치·이름 오타 → `ollama list`로 확인, `ollama pull xxx` |
| 답변이 매우 느림 | `ollama ps`에서 CPU 비율 확인 → 더 작은 모델·양자화 사용, `num_ctx` 줄이기 |
| `model requires more system memory` | 메모리 부족 → 더 작은 모델(예: `8b` → `3b`) 사용 |
| GPU를 사용하지 않음 | 그래픽 드라이버(NVIDIA 등) 최신으로 업데이트 후 Ollama 재시작 |
| 긴 문서를 넣으면 앞부분을 잊음 | 컨텍스트 초과 → `num_ctx`를 늘리기 (`/set parameter num_ctx 16384` 또는 API `options`) |
| C 드라이브 용량 부족 | `OLLAMA_MODELS`로 저장 폴더를 다른 드라이브로 변경 |
| 다른 PC에서 접속 안 됨 | `OLLAMA_HOST=0.0.0.0` 설정 + 방화벽에서 11434 포트 허용 |
| 한글이 깨짐 (PowerShell API 호출) | `-ContentType "application/json; charset=utf-8"` 지정 |

로그 위치 (Windows): `%LOCALAPPDATA%\Ollama\server.log`

---

## 참고 자료

- 공식 사이트: <https://ollama.com>
- 모델 라이브러리: <https://ollama.com/library>
- 공식 문서: <https://docs.ollama.com>
- GitHub: <https://github.com/ollama/ollama>
- Python 라이브러리: <https://github.com/ollama/ollama-python>
- 명령어 상세: `ollama help <명령어>`
