# uv 명령어 레퍼런스

> 기준 버전: **uv 0.13.0** (`uv --version`으로 확인)
> 모든 명령어의 상세 옵션은 `uv help <명령어>` 또는 `uv <명령어> --help`로 볼 수 있습니다.

uv는 Rust로 작성된 매우 빠른 Python 패키지·프로젝트 관리 도구입니다.
`pip`, `pip-tools`, `virtualenv`, `pyenv`, `pipx`, `poetry`의 역할을 하나로 합쳐 놓았다고 보면 됩니다.

| 기존 도구 | uv에서 대응되는 기능 |
|---|---|
| `pip install` | `uv add` (프로젝트) / `uv pip install` (pip 호환) |
| `python -m venv` / `virtualenv` | `uv venv` |
| `pyenv` | `uv python install` / `uv python pin` |
| `pipx` | `uv tool install` / `uvx` |
| `pip-tools` (`pip-compile`) | `uv lock` / `uv pip compile` |
| `poetry` | `uv init` + `uv add` + `uv sync` + `uv build` |

---

## 목차

1. [설치와 업데이트](#1-설치와-업데이트)
2. [프로젝트 관리 (가장 많이 사용)](#2-프로젝트-관리-가장-많이-사용)
3. [Python 버전 관리](#3-python-버전-관리)
4. [가상환경 (`uv venv`)](#4-가상환경-uv-venv)
5. [도구 실행·설치 (`uvx`, `uv tool`)](#5-도구-실행설치-uvx-uv-tool)
6. [단일 스크립트 실행](#6-단일-스크립트-실행)
7. [pip 호환 인터페이스 (`uv pip`)](#7-pip-호환-인터페이스-uv-pip)
8. [코드 품질: 포맷·검사·보안 감사](#8-코드-품질-포맷검사보안-감사)
9. [빌드와 배포](#9-빌드와-배포)
10. [워크스페이스](#10-워크스페이스)
11. [캐시 관리](#11-캐시-관리)
12. [인증](#12-인증)
13. [전역 옵션과 환경 변수](#13-전역-옵션과-환경-변수)
14. [자주 쓰는 작업 흐름](#14-자주-쓰는-작업-흐름)

---

## 1. 설치와 업데이트

```powershell
# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# pip로 설치하는 방법도 있음
pip install uv
```

| 명령어 | 설명 |
|---|---|
| `uv --version` | uv 버전 확인 |
| `uv self version` | uv 버전 확인 (상세) |
| `uv self update` | uv를 최신 버전으로 업데이트 (설치 스크립트로 설치한 경우) |
| `uv help` | 전체 명령어 목록 |
| `uv help <명령어>` | 특정 명령어의 상세 도움말 |

---

## 2. 프로젝트 관리 (가장 많이 사용)

uv 프로젝트는 `pyproject.toml`(의존성 선언), `uv.lock`(정확한 버전 고정), `.venv/`(가상환경)로 구성됩니다.
`uv add` / `uv run` / `uv sync`를 실행하면 **lock 파일과 `.venv`가 자동으로 맞춰지므로** 따로 가상환경을 활성화하지 않아도 됩니다.

### 2.1 `uv init` — 새 프로젝트 만들기

| 명령어 | 설명 |
|---|---|
| `uv init` | 현재 폴더를 프로젝트로 초기화 |
| `uv init my-project` | `my-project` 폴더를 새로 만들어 초기화 |
| `uv init --app` | 애플리케이션용 프로젝트 (기본값) |
| `uv init --lib` | 라이브러리(패키지)용 프로젝트 (`src/` 구조) |
| `uv init --package` | 빌드 가능한 패키지 구조로 생성 |
| `uv init --python 3.12` | 사용할 Python 버전을 지정해 생성 |
| `uv init --bare` | `pyproject.toml`만 생성 (예제 파일 없이) |

생성되는 파일:

```
my-project/
├── .python-version   # 프로젝트의 Python 버전
├── README.md
├── main.py           # 예제 코드
└── pyproject.toml    # 프로젝트 메타데이터, 의존성
```

### 2.2 `uv add` / `uv remove` — 의존성 추가·제거

| 명령어 | 설명 |
|---|---|
| `uv add requests` | 패키지 추가 (`pyproject.toml`, `uv.lock`, `.venv` 모두 갱신) |
| `uv add "fastapi>=0.110"` | 버전 조건을 지정해 추가 |
| `uv add numpy pandas matplotlib` | 여러 개를 한 번에 추가 |
| `uv add --dev pytest ruff` | 개발용 의존성(`dev` 그룹)으로 추가 |
| `uv add --group lint ruff` | 특정 의존성 그룹으로 추가 |
| `uv add --optional gpu torch` | 선택 의존성(extra `gpu`)으로 추가 |
| `uv add -r requirements.txt` | `requirements.txt`의 패키지를 한꺼번에 추가 |
| `uv add "git+https://github.com/user/repo"` | Git 저장소에서 추가 |
| `uv add --editable ./libs/mylib` | 로컬 패키지를 편집 가능 모드로 추가 |
| `uv remove requests` | 패키지 제거 |
| `uv remove --dev pytest` | 개발용 의존성 제거 |

### 2.3 `uv sync` — 환경을 lock 파일과 동기화

| 명령어 | 설명 |
|---|---|
| `uv sync` | `uv.lock` 기준으로 `.venv` 생성·갱신 (lock 파일에 없는 패키지는 제거) |
| `uv sync --no-dev` | 개발용 의존성은 빼고 설치 (배포 환경용) |
| `uv sync --all-extras` | 모든 선택 의존성 포함 |
| `uv sync --extra gpu` | 특정 extra 포함 |
| `uv sync --all-groups` | 모든 의존성 그룹 포함 |
| `uv sync --locked` | `uv.lock`이 최신이 아니면 오류 (CI에서 사용) |
| `uv sync --frozen` | `uv.lock`을 갱신하지 않고 그대로 설치 |
| `uv sync --upgrade` | 허용된 범위 안에서 모든 패키지를 최신으로 올림 |

> 💡 다른 사람의 프로젝트를 `git clone` 받은 뒤에는 `uv sync` 한 번이면 환경이 준비됩니다.

### 2.4 `uv run` — 프로젝트 환경에서 실행

| 명령어 | 설명 |
|---|---|
| `uv run main.py` | 프로젝트 환경에서 스크립트 실행 (필요하면 먼저 자동 sync) |
| `uv run python` | 프로젝트 환경의 Python REPL 실행 |
| `uv run pytest` | 프로젝트에 설치된 명령어 실행 |
| `uv run --with rich main.py` | 이번 실행에만 패키지를 임시로 추가 |
| `uv run --python 3.11 main.py` | 특정 Python 버전으로 실행 |
| `uv run --no-sync main.py` | 동기화 없이 바로 실행 |
| `uv run -m http.server` | 모듈 실행 (`python -m`과 동일) |

> 💡 `uv run`을 쓰면 `.venv\Scripts\activate`로 가상환경을 활성화할 필요가 없습니다.

### 2.5 `uv lock` — lock 파일 관리

| 명령어 | 설명 |
|---|---|
| `uv lock` | `pyproject.toml`을 기준으로 `uv.lock` 생성·갱신 |
| `uv lock --upgrade` | 모든 패키지를 허용 범위 안의 최신 버전으로 다시 계산 |
| `uv lock --upgrade-package requests` | 특정 패키지만 업그레이드 |
| `uv lock --check` | lock 파일이 최신인지 확인만 함 |

### 2.6 `uv tree` / `uv export` / `uv version`

| 명령어 | 설명 |
|---|---|
| `uv tree` | 의존성 트리 출력 |
| `uv tree --depth 1` | 직접 의존성만 출력 |
| `uv tree --outdated` | 최신 버전이 있는 패키지 표시 |
| `uv tree --invert --package urllib3` | 해당 패키지를 누가 쓰는지 역방향으로 출력 |
| `uv export --format requirements-txt -o requirements.txt` | lock 파일을 `requirements.txt`로 내보내기 |
| `uv export --no-dev -o requirements.txt` | 개발용 의존성을 빼고 내보내기 |
| `uv version` | 프로젝트 버전 확인 |
| `uv version 1.2.0` | 프로젝트 버전 직접 지정 |
| `uv version --bump minor` | 버전 올리기 (`major` / `minor` / `patch`) |

---

## 3. Python 버전 관리

uv는 Python 자체도 내려받아 관리합니다. 시스템에 Python이 없어도 됩니다.

| 명령어 | 설명 |
|---|---|
| `uv python list` | 설치된 버전과 설치 가능한 버전 목록 |
| `uv python list --only-installed` | 설치된 버전만 표시 |
| `uv python install` | 최신 Python 설치 |
| `uv python install 3.12` | 특정 버전 설치 |
| `uv python install 3.11 3.12 3.13` | 여러 버전 한 번에 설치 |
| `uv python upgrade` | 설치된 Python을 최신 패치 버전으로 업그레이드 |
| `uv python find` | 현재 사용될 Python 경로 확인 |
| `uv python find 3.12` | 특정 버전의 경로 찾기 |
| `uv python pin 3.12` | 현재 폴더에 `.python-version` 파일 생성 (버전 고정) |
| `uv python dir` | uv가 Python을 설치하는 폴더 확인 |
| `uv python uninstall 3.11` | 특정 버전 삭제 |
| `uv python update-shell` | Python 실행 경로를 `PATH`에 추가 |

> 💡 `.python-version` 파일이 있으면 `uv run`, `uv sync`, `uv venv`가 그 버전을 자동으로 사용합니다.

---

## 4. 가상환경 (`uv venv`)

| 명령어 | 설명 |
|---|---|
| `uv venv` | 현재 폴더에 `.venv` 생성 |
| `uv venv myenv` | 원하는 이름으로 생성 |
| `uv venv --python 3.12` | 특정 Python 버전으로 생성 |
| `uv venv --seed` | `pip`, `setuptools`까지 함께 설치 |
| `uv venv --clear` | 기존 가상환경을 지우고 새로 생성 |

가상환경 활성화 (직접 활성화해야 할 때만):

```powershell
# Windows PowerShell
.venv\Scripts\activate

# Windows cmd
.venv\Scripts\activate.bat

# macOS / Linux
source .venv/bin/activate

# 비활성화
deactivate
```

> ⚠️ PowerShell에서 "스크립트를 실행할 수 없으므로…" 오류가 나면 한 번만 실행:
> `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`

---

## 5. 도구 실행·설치 (`uvx`, `uv tool`)

`ruff`, `black`, `httpie`처럼 **CLI 도구로 쓰는 패키지**를 프로젝트와 분리해서 실행·설치합니다 (`pipx`와 같은 역할).

| 명령어 | 설명 |
|---|---|
| `uvx ruff check .` | 설치하지 않고 임시 환경에서 바로 실행 (`uv tool run`의 줄임말) |
| `uvx ruff@0.6.0 check .` | 특정 버전으로 실행 |
| `uvx --from httpie http GET example.com` | 패키지 이름과 명령어 이름이 다를 때 |
| `uvx --with pandas jupyter lab` | 추가 패키지와 함께 실행 |
| `uv tool install ruff` | 도구를 전역으로 설치 (어디서나 `ruff` 실행 가능) |
| `uv tool install --python 3.12 black` | 특정 Python 버전으로 설치 |
| `uv tool list` | 설치된 도구 목록 |
| `uv tool upgrade ruff` | 특정 도구 업그레이드 |
| `uv tool upgrade --all` | 모든 도구 업그레이드 |
| `uv tool uninstall ruff` | 도구 삭제 |
| `uv tool audit` | 설치된 도구의 보안 취약점 검사 |
| `uv tool dir` | 도구 설치 폴더 확인 |
| `uv tool update-shell` | 도구 실행 경로를 `PATH`에 추가 |

---

## 6. 단일 스크립트 실행

프로젝트를 만들지 않고 **파일 하나**에 의존성을 적어 두고 실행할 수 있습니다 (PEP 723 인라인 메타데이터).

| 명령어 | 설명 |
|---|---|
| `uv run script.py` | 스크립트 실행 |
| `uv run --with requests script.py` | 의존성을 지정해 실행 |
| `uv init --script script.py --python 3.12` | 메타데이터가 들어간 스크립트 파일 생성 |
| `uv add --script script.py requests` | 스크립트 파일 안에 의존성 기록 |
| `uv remove --script script.py requests` | 스크립트 파일에서 의존성 제거 |
| `uv lock --script script.py` | 스크립트 전용 lock 파일 생성 |

`uv add --script`를 실행하면 파일 맨 위에 다음 내용이 추가됩니다:

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#     "requests",
# ]
# ///

import requests
print(requests.get("https://httpbin.org/get").status_code)
```

이후 `uv run script.py`만 실행하면 필요한 패키지가 임시 환경에 자동으로 설치됩니다.

---

## 7. pip 호환 인터페이스 (`uv pip`)

기존 `pip` 사용법을 그대로 쓰면서 속도만 빠르게 하고 싶을 때 사용합니다.
**`pyproject.toml`이나 `uv.lock`을 갱신하지 않으므로**, uv 프로젝트에서는 `uv add`를 쓰는 것이 좋습니다.

| 명령어 | 설명 |
|---|---|
| `uv pip install requests` | 현재 가상환경에 설치 |
| `uv pip install -r requirements.txt` | requirements 파일로 설치 |
| `uv pip install -e .` | 현재 프로젝트를 편집 가능 모드로 설치 |
| `uv pip install --system requests` | 가상환경이 아닌 시스템 Python에 설치 |
| `uv pip uninstall requests` | 제거 |
| `uv pip list` | 설치된 패키지 목록 |
| `uv pip list --outdated` | 업데이트 가능한 패키지 목록 |
| `uv pip show requests` | 패키지 정보 |
| `uv pip freeze` | 설치된 패키지를 `requirements` 형식으로 출력 |
| `uv pip freeze > requirements.txt` | 파일로 저장 |
| `uv pip tree` | 설치된 패키지의 의존성 트리 |
| `uv pip check` | 설치된 패키지 간 호환성 검사 |
| `uv pip compile requirements.in -o requirements.txt` | 버전이 고정된 requirements 파일 생성 (`pip-compile` 대체) |
| `uv pip sync requirements.txt` | 환경을 파일 내용과 정확히 일치시킴 (나머지 패키지 제거) |

---

## 8. 코드 품질: 포맷·검사·보안 감사

| 명령어 | 설명 |
|---|---|
| `uv format` | 프로젝트 코드 포맷 정리 (내부적으로 Ruff 사용) |
| `uv format --check` | 변경하지 않고 포맷이 맞는지 확인만 함 |
| `uv format --diff` | 바뀔 내용을 diff로 보여줌 |
| `uv format -- --line-length 100` | `--` 뒤의 인자는 Ruff에 그대로 전달 |
| `uv check` | 프로젝트 타입 검사 실행 |
| `uv check --fix` | 타입 검사 오류 중 안전하게 고칠 수 있는 것을 자동 수정 |
| `uv audit` | 프로젝트 의존성의 보안 취약점 검사 |
| `uv audit --no-dev` | 개발용 의존성은 빼고 검사 |
| `uv audit --output-format json` | 결과를 JSON으로 출력 (`text` / `json` / `sarif`) |
| `uv audit --ignore <취약점ID>` | 특정 취약점은 무시 |

---

## 9. 빌드와 배포

| 명령어 | 설명 |
|---|---|
| `uv build` | 소스 배포판(`.tar.gz`)과 휠(`.whl`)을 `dist/`에 생성 |
| `uv build --wheel` | 휠만 생성 |
| `uv build --sdist` | 소스 배포판만 생성 |
| `uv publish` | `dist/`의 파일을 PyPI에 업로드 |
| `uv publish --token <토큰>` | API 토큰으로 업로드 |
| `uv publish --publish-url https://test.pypi.org/legacy/` | TestPyPI에 업로드 |

---

## 10. 워크스페이스

여러 패키지를 하나의 저장소에서 함께 관리할 때(모노레포) 사용합니다.
루트 `pyproject.toml`에 `[tool.uv.workspace]`로 멤버를 지정합니다.

| 명령어 | 설명 |
|---|---|
| `uv workspace list` | 워크스페이스 멤버 목록 |
| `uv workspace dir` | 워크스페이스 멤버 경로 표시 |
| `uv workspace metadata` | 워크스페이스 메타데이터 출력 |
| `uv sync --all-packages` | 모든 멤버 패키지 동기화 |
| `uv run --package mylib pytest` | 특정 멤버 패키지 환경에서 실행 |

---

## 11. 캐시 관리

uv는 내려받은 패키지를 전역 캐시에 저장해서 다시 설치할 때 매우 빠르게 처리합니다.

| 명령어 | 설명 |
|---|---|
| `uv cache dir` | 캐시 폴더 위치 확인 |
| `uv cache size` | 캐시 용량 확인 |
| `uv cache clean` | 캐시 전체 삭제 |
| `uv cache clean requests` | 특정 패키지의 캐시만 삭제 |
| `uv cache prune` | 사용하지 않는 캐시 항목만 정리 |

---

## 12. 인증

사설 패키지 저장소(private index)를 쓸 때 사용합니다.

| 명령어 | 설명 |
|---|---|
| `uv auth login <서비스 URL>` | 서비스에 로그인 |
| `uv auth logout <서비스 URL>` | 로그아웃 |
| `uv auth token <서비스 URL>` | 저장된 인증 토큰 확인 |
| `uv auth dir` | 인증 정보 저장 폴더 확인 |

---

## 13. 전역 옵션과 환경 변수

모든 명령어에 붙일 수 있는 옵션입니다.

| 옵션 | 환경 변수 | 설명 |
|---|---|---|
| `-q`, `--quiet` | | 출력 줄이기 |
| `-v`, `--verbose` | | 자세한 출력 (디버깅용) |
| `-n`, `--no-cache` | `UV_NO_CACHE` | 캐시 사용 안 함 |
| `--cache-dir <경로>` | `UV_CACHE_DIR` | 캐시 폴더 지정 |
| `--offline` | `UV_OFFLINE` | 네트워크 사용 안 함 |
| `--directory <경로>` | `UV_WORKING_DIR` | 해당 폴더로 이동한 뒤 실행 |
| `--project <경로>` | `UV_PROJECT` | 해당 폴더의 프로젝트를 대상으로 실행 |
| `--managed-python` | `UV_MANAGED_PYTHON` | uv가 설치한 Python만 사용 |
| `--no-managed-python` | `UV_NO_MANAGED_PYTHON` | 시스템 Python만 사용 |
| `--no-python-downloads` | `UV_PYTHON_DOWNLOADS=never` | Python 자동 다운로드 금지 |
| `--system-certs` | `UV_SYSTEM_CERTS` | OS 인증서 저장소 사용 (사내망 SSL 오류 시) |
| `--allow-insecure-host <호스트>` | `UV_INSECURE_HOST` | SSL 검증 없이 접속 허용 |
| `--no-progress` | `UV_NO_PROGRESS` | 진행 표시줄 숨기기 |

그 밖에 자주 쓰는 환경 변수:

| 환경 변수 | 설명 |
|---|---|
| `UV_INDEX_URL` / `UV_DEFAULT_INDEX` | 기본 패키지 저장소 변경 (예: 사내 미러) |
| `UV_PYTHON` | 사용할 Python 버전·경로 지정 |
| `UV_PROJECT_ENVIRONMENT` | 프로젝트 가상환경 경로 변경 (기본값 `.venv`) |
| `UV_LINK_MODE=copy` | 캐시에서 파일을 가져오는 방식 (하드링크 경고가 뜰 때 `copy`로 설정) |

---

## 14. 자주 쓰는 작업 흐름

### 새 프로젝트 시작

```powershell
uv init llm-project --python 3.12
cd llm-project
uv add openai anthropic python-dotenv
uv add --dev pytest ruff
uv run main.py
```

### 다른 사람의 프로젝트 받아서 실행

```powershell
git clone https://github.com/user/repo.git
cd repo
uv sync          # .venv 생성 + 의존성 설치
uv run main.py
```

### 기존 `requirements.txt` 프로젝트를 uv로 옮기기

```powershell
uv init
uv add -r requirements.txt
# 이후 requirements.txt 대신 pyproject.toml + uv.lock으로 관리
```

### 패키지 업데이트

```powershell
uv tree --outdated                  # 업데이트할 패키지 확인
uv lock --upgrade-package openai    # 특정 패키지만
uv lock --upgrade                   # 전부
uv sync
```

### Jupyter 사용

```powershell
uv add --dev ipykernel
uv run jupyter lab                       # jupyter가 프로젝트에 있을 때
uvx --with ipykernel jupyter lab         # 프로젝트에 설치하지 않고 실행
```

### 배포 서버용 requirements 만들기

```powershell
uv export --no-dev --format requirements-txt -o requirements.txt
```

---

## 참고 자료

- 공식 문서: <https://docs.astral.sh/uv/>
- GitHub: <https://github.com/astral-sh/uv>
- 명령어 상세: `uv help <명령어>`
