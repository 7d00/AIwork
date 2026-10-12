# Git 명령어 레퍼런스

> 기준 버전: **Git 2.56.0** (`git --version`으로 확인)
> 모든 명령어의 상세 옵션은 `git help <명령어>` 또는 `git <명령어> -h`로 볼 수 있습니다.

Git은 파일의 변경 이력을 기록하고 관리하는 **분산 버전 관리 시스템**입니다.
GitHub는 Git 저장소를 인터넷에 올려 두고 다른 사람과 공유·협업하는 서비스입니다.

| 용어 | 뜻 |
|---|---|
| 저장소 (repository, repo) | Git으로 관리되는 프로젝트 폴더 (`.git/` 폴더에 이력이 저장됨) |
| 작업 폴더 (working tree) | 실제로 파일을 편집하는 폴더 |
| 스테이징 영역 (staging area, index) | 다음 커밋에 포함할 변경 사항을 모아 두는 곳 |
| 커밋 (commit) | 스테이징된 변경 사항을 하나의 기록으로 저장한 것 |
| 브랜치 (branch) | 독립적인 작업 흐름 (기본 브랜치는 보통 `main`) |
| 원격 저장소 (remote) | GitHub 등 다른 곳에 있는 저장소 (기본 이름은 `origin`) |
| HEAD | 현재 작업 중인 커밋(브랜치)을 가리키는 포인터 |

파일은 다음 단계를 거쳐 GitHub에 올라갑니다:

```
작업 폴더 ──git add──▶ 스테이징 영역 ──git commit──▶ 로컬 저장소 ──git push──▶ 원격 저장소(GitHub)
                                                       ◀──git pull / git fetch──
```

---

## 목차

1. [설치와 초기 설정](#1-설치와-초기-설정)
2. [저장소 만들기·가져오기](#2-저장소-만들기가져오기)
3. [기본 작업: 상태 확인·추가·커밋](#3-기본-작업-상태-확인추가커밋)
4. [변경 내용·이력 보기](#4-변경-내용이력-보기)
5. [브랜치](#5-브랜치)
6. [병합과 리베이스](#6-병합과-리베이스)
7. [원격 저장소 (GitHub)](#7-원격-저장소-github)
8. [되돌리기·취소](#8-되돌리기취소)
9. [임시 저장 (`git stash`)](#9-임시-저장-git-stash)
10. [태그](#10-태그)
11. [`.gitignore` — 추적 제외 파일](#11-gitignore--추적-제외-파일)
12. [GitHub 인증](#12-github-인증)
13. [GitHub CLI (`gh`)](#13-github-cli-gh)
14. [자주 쓰는 작업 흐름](#14-자주-쓰는-작업-흐름)
15. [자주 만나는 오류와 해결](#15-자주-만나는-오류와-해결)

---

## 1. 설치와 초기 설정

```powershell
# Windows (winget)
winget install --id Git.Git -e --source winget

# macOS
brew install git

# Ubuntu / Debian
sudo apt install git
```

> ⚠️ 설치 직후 `git` 명령을 찾을 수 없다고 나오면 **VS Code나 터미널을 완전히 닫았다가 다시 여세요.** (PATH가 새로 반영되어야 함)

처음 한 번은 커밋에 기록될 이름과 이메일을 설정해야 합니다 (GitHub 계정의 이메일 사용).

| 명령어 | 설명 |
|---|---|
| `git --version` | Git 버전 확인 |
| `git config --global user.name "이름"` | 커밋 작성자 이름 설정 |
| `git config --global user.email "메일@example.com"` | 커밋 작성자 이메일 설정 |
| `git config --global init.defaultBranch main` | 새 저장소의 기본 브랜치 이름을 `main`으로 |
| `git config --global core.autocrlf true` | (Windows) 줄바꿈 문자 자동 변환 |
| `git config --global core.quotepath false` | 한글 파일 이름이 `\354\225...`처럼 깨져 보이지 않게 |
| `git config --global core.editor "code --wait"` | 커밋 메시지 편집기를 VS Code로 |
| `git config --global pull.rebase false` | `git pull` 시 병합(merge) 방식 사용 |
| `git config --list` | 현재 설정 전체 보기 |
| `git config --global --edit` | 전역 설정 파일 직접 편집 |
| `git config --global alias.st status` | 별칭 만들기 (`git st` = `git status`) |

> 💡 `--global`은 내 컴퓨터 전체에 적용, 빼면 현재 저장소에만 적용됩니다.

---

## 2. 저장소 만들기·가져오기

| 명령어 | 설명 |
|---|---|
| `git init` | 현재 폴더를 Git 저장소로 만들기 (`.git/` 생성) |
| `git init my-project` | `my-project` 폴더를 새로 만들어 저장소로 초기화 |
| `git clone <URL>` | 원격 저장소를 통째로 내려받기 |
| `git clone <URL> 폴더이름` | 원하는 폴더 이름으로 내려받기 |
| `git clone --depth 1 <URL>` | 최신 커밋만 내려받기 (빠름, 이력 없음) |
| `git clone -b dev <URL>` | 특정 브랜치로 내려받기 |

```powershell
git clone https://github.com/user/repo.git
```

---

## 3. 기본 작업: 상태 확인·추가·커밋

### 3.1 `git status` — 현재 상태 확인

| 명령어 | 설명 |
|---|---|
| `git status` | 변경된 파일, 스테이징된 파일, 추적하지 않는 파일 표시 |
| `git status -s` | 짧은 형식으로 표시 (`M` 수정, `A` 추가, `??` 미추적) |

### 3.2 `git add` — 스테이징

| 명령어 | 설명 |
|---|---|
| `git add 파일.py` | 특정 파일 스테이징 |
| `git add .` | 현재 폴더 아래의 모든 변경 사항 스테이징 |
| `git add -A` | 저장소 전체의 모든 변경 사항(삭제 포함) 스테이징 |
| `git add *.md` | 패턴에 맞는 파일 스테이징 |
| `git add -p` | 변경 내용을 조각(hunk)별로 골라서 스테이징 |
| `git restore --staged 파일.py` | 스테이징 취소 (파일 내용은 그대로) |

### 3.3 `git commit` — 기록 저장

| 명령어 | 설명 |
|---|---|
| `git commit -m "메시지"` | 스테이징된 내용을 커밋 |
| `git commit -am "메시지"` | 이미 추적 중인 파일의 변경을 add + commit 한 번에 (새 파일은 제외) |
| `git commit` | 편집기를 열어 긴 메시지 작성 |
| `git commit --amend -m "새 메시지"` | 직전 커밋의 메시지 수정 |
| `git commit --amend --no-edit` | 빠뜨린 파일을 직전 커밋에 추가 (먼저 `git add`) |

> ⚠️ `--amend`는 커밋을 새로 만들기 때문에 **이미 push한 커밋에는 쓰지 않는 것이 좋습니다.**

### 3.4 파일 삭제·이동

| 명령어 | 설명 |
|---|---|
| `git rm 파일.py` | 파일 삭제 + 삭제 사실 스테이징 |
| `git rm --cached 파일.py` | Git 추적에서만 제외 (파일은 남김, `.gitignore`와 함께 사용) |
| `git rm -r --cached 폴더/` | 폴더를 추적에서만 제외 |
| `git mv 옛이름.py 새이름.py` | 파일 이름 변경·이동 |

---

## 4. 변경 내용·이력 보기

### 4.1 `git diff` — 변경 내용 비교

| 명령어 | 설명 |
|---|---|
| `git diff` | 작업 폴더 vs 스테이징 영역 (아직 add 안 한 변경) |
| `git diff --staged` | 스테이징 영역 vs 마지막 커밋 (커밋될 내용) |
| `git diff HEAD` | 작업 폴더 vs 마지막 커밋 (전체 변경) |
| `git diff main dev` | 두 브랜치 비교 |
| `git diff <커밋1> <커밋2>` | 두 커밋 비교 |
| `git diff --stat` | 파일별 변경 줄 수만 요약 |
| `git diff --name-only` | 변경된 파일 이름만 |

### 4.2 `git log` — 커밋 이력

| 명령어 | 설명 |
|---|---|
| `git log` | 커밋 이력 (`q`로 종료) |
| `git log --oneline` | 한 줄씩 간단하게 |
| `git log --oneline --graph --all` | 모든 브랜치를 그래프로 |
| `git log -5` | 최근 5개만 |
| `git log -p 파일.py` | 특정 파일의 변경 내용과 함께 |
| `git log --stat` | 커밋별 변경 파일 요약 |
| `git log --author="이름"` | 작성자로 필터 |
| `git log --since="2 weeks ago"` | 기간으로 필터 |
| `git log --grep="버그"` | 커밋 메시지로 검색 |
| `git log -S "함수이름"` | 해당 문자열이 추가·삭제된 커밋 검색 |

### 4.3 기타 조회

| 명령어 | 설명 |
|---|---|
| `git show` | 마지막 커밋의 내용 |
| `git show <커밋>` | 특정 커밋의 내용 |
| `git show <커밋>:파일.py` | 특정 커밋 시점의 파일 내용 |
| `git blame 파일.py` | 줄마다 누가 언제 수정했는지 |
| `git reflog` | HEAD가 이동한 모든 기록 (실수로 지운 커밋 복구할 때) |
| `git shortlog -sn` | 작성자별 커밋 수 |

> 💡 커밋은 `a1b2c3d` 같은 해시 앞 7자리로 지정하거나, `HEAD`(현재), `HEAD~1`(1개 전), `HEAD~3`(3개 전)처럼 지정할 수 있습니다.

---

## 5. 브랜치

브랜치를 쓰면 `main`을 건드리지 않고 새 기능을 따로 개발할 수 있습니다.

| 명령어 | 설명 |
|---|---|
| `git branch` | 로컬 브랜치 목록 (`*`가 현재 브랜치) |
| `git branch -a` | 원격 브랜치까지 모두 |
| `git branch -v` | 브랜치별 마지막 커밋 표시 |
| `git branch feature` | 새 브랜치 만들기 (이동은 안 함) |
| `git switch feature` | 브랜치 이동 |
| `git switch -c feature` | 새 브랜치를 만들고 바로 이동 |
| `git switch -` | 직전 브랜치로 돌아가기 |
| `git branch -m 새이름` | 현재 브랜치 이름 변경 |
| `git branch -d feature` | 브랜치 삭제 (병합된 경우만) |
| `git branch -D feature` | 브랜치 강제 삭제 |
| `git branch --merged` | 현재 브랜치에 병합된 브랜치 목록 |

> 💡 예전 방식인 `git checkout feature`, `git checkout -b feature`도 똑같이 동작합니다. 새로 배운다면 `git switch`(브랜치 이동)와 `git restore`(파일 되돌리기)를 쓰는 것이 헷갈리지 않습니다.

---

## 6. 병합과 리베이스

### 6.1 `git merge` — 브랜치 합치기

```powershell
git switch main        # 합쳐 받을 브랜치로 이동
git merge feature      # feature의 변경을 main에 합침
```

| 명령어 | 설명 |
|---|---|
| `git merge feature` | 현재 브랜치에 `feature` 병합 |
| `git merge --no-ff feature` | 항상 병합 커밋을 만들어 이력에 남김 |
| `git merge --squash feature` | `feature`의 커밋들을 하나로 합쳐 스테이징 (이후 `git commit`) |
| `git merge --abort` | 충돌이 났을 때 병합 취소 |

### 6.2 충돌(conflict) 해결

두 브랜치가 같은 부분을 다르게 수정하면 충돌이 납니다. 파일 안에 다음 표시가 생깁니다:

```
<<<<<<< HEAD
현재 브랜치의 내용
=======
합치려는 브랜치의 내용
>>>>>>> feature
```

1. 파일을 열어 원하는 내용만 남기고 `<<<<<<<`, `=======`, `>>>>>>>` 줄을 지웁니다 (VS Code에서는 "Accept Current / Incoming / Both" 버튼 사용 가능).
2. `git add 파일` 로 해결 완료를 표시합니다.
3. `git commit` (merge 중) 또는 `git rebase --continue` (rebase 중)로 마무리합니다.

### 6.3 `git rebase` — 이력을 한 줄로 정리

| 명령어 | 설명 |
|---|---|
| `git rebase main` | 현재 브랜치의 커밋들을 `main` 끝으로 옮겨 붙임 |
| `git rebase --continue` | 충돌 해결 후 계속 |
| `git rebase --abort` | 리베이스 취소, 원래 상태로 |
| `git rebase -i HEAD~3` | 최근 3개 커밋을 합치기·수정·순서 변경 (대화형) |
| `git cherry-pick <커밋>` | 다른 브랜치의 특정 커밋 하나만 가져오기 |

> ⚠️ rebase는 커밋을 새로 만듭니다. **다른 사람과 같이 쓰는 브랜치(이미 push한 커밋)에는 rebase하지 마세요.**

---

## 7. 원격 저장소 (GitHub)

### 7.1 `git remote` — 원격 저장소 연결

| 명령어 | 설명 |
|---|---|
| `git remote -v` | 연결된 원격 저장소 목록 |
| `git remote add origin <URL>` | `origin`이라는 이름으로 원격 저장소 연결 |
| `git remote set-url origin <새 URL>` | 원격 저장소 주소 변경 |
| `git remote rename origin upstream` | 원격 저장소 이름 변경 |
| `git remote remove origin` | 원격 저장소 연결 해제 |
| `git remote show origin` | 원격 저장소 상세 정보 |

### 7.2 `git push` — 올리기

| 명령어 | 설명 |
|---|---|
| `git push -u origin main` | 처음 올릴 때: `main`을 올리고 추적 관계 설정 |
| `git push` | 이후에는 이것만으로 올리기 |
| `git push origin feature` | 특정 브랜치 올리기 |
| `git push -u origin feature` | 새 브랜치를 처음 올리면서 추적 설정 |
| `git push origin --delete feature` | 원격 브랜치 삭제 |
| `git push --tags` | 태그 모두 올리기 |
| `git push --force-with-lease` | 강제로 덮어쓰기 (다른 사람이 그사이 올린 게 있으면 거부, `--force`보다 안전) |

### 7.3 `git fetch` / `git pull` — 내려받기

| 명령어 | 설명 |
|---|---|
| `git fetch` | 원격의 변경 사항을 받아오기만 함 (내 파일은 안 바뀜) |
| `git fetch --prune` | 원격에서 삭제된 브랜치 정보도 정리 |
| `git pull` | fetch + merge (원격 변경을 받아서 내 브랜치에 합침) |
| `git pull --rebase` | fetch + rebase (병합 커밋 없이 이력을 한 줄로) |
| `git pull origin main` | 특정 원격 브랜치를 받아 합치기 |

> 💡 작업 시작 전에 `git pull`, 작업 후에 `git push` 하는 습관을 들이면 충돌이 줄어듭니다.

### 7.4 원격 브랜치 가져와 작업하기

```powershell
git fetch
git switch feature     # origin/feature가 있으면 자동으로 추적 브랜치를 만들어 이동
```

---

## 8. 되돌리기·취소

| 상황 | 명령어 |
|---|---|
| 수정한 파일을 마지막 커밋 상태로 되돌리기 | `git restore 파일.py` |
| 모든 수정을 버리기 | `git restore .` |
| 스테이징만 취소 | `git restore --staged 파일.py` |
| 파일을 특정 커밋 시점으로 되돌리기 | `git restore --source=<커밋> 파일.py` |
| 직전 커밋 취소, 변경은 스테이징 상태로 남김 | `git reset --soft HEAD~1` |
| 직전 커밋 취소, 변경은 작업 폴더에 남김 | `git reset HEAD~1` (`--mixed`, 기본값) |
| 직전 커밋 취소, 변경도 완전히 삭제 | `git reset --hard HEAD~1` ⚠️ |
| 작업 폴더를 원격 상태로 완전히 맞추기 | `git reset --hard origin/main` ⚠️ |
| 이미 push한 커밋을 되돌리기 (되돌리는 커밋을 새로 만듦) | `git revert <커밋>` |
| 추적하지 않는 새 파일 삭제 (미리 보기) | `git clean -n` |
| 추적하지 않는 새 파일·폴더 삭제 | `git clean -fd` ⚠️ |
| 실수로 지운 커밋 찾기 | `git reflog` → `git reset --hard <해시>` |

> ⚠️ `--hard`와 `git clean`은 **되돌릴 수 없이 파일을 지웁니다.** 실행 전 `git status`로 확인하세요.
> 💡 **push 전**이면 `reset`, **push 후**면 `revert`를 쓰는 것이 원칙입니다.

---

## 9. 임시 저장 (`git stash`)

커밋하기엔 이른 작업을 잠시 치워 두고 다른 브랜치로 이동해야 할 때 사용합니다.

| 명령어 | 설명 |
|---|---|
| `git stash` | 현재 변경 사항을 임시 저장하고 작업 폴더를 깨끗하게 |
| `git stash -u` | 추적하지 않는 새 파일까지 함께 저장 |
| `git stash push -m "메모"` | 메모를 붙여 저장 |
| `git stash list` | 임시 저장 목록 |
| `git stash pop` | 가장 최근 것을 꺼내 적용하고 목록에서 삭제 |
| `git stash apply stash@{1}` | 특정 항목을 적용 (목록에는 남김) |
| `git stash show -p stash@{0}` | 저장된 내용 보기 |
| `git stash drop stash@{0}` | 특정 항목 삭제 |
| `git stash clear` | 전부 삭제 |

---

## 10. 태그

릴리스 버전(`v1.0.0` 등)에 이름표를 붙일 때 사용합니다.

| 명령어 | 설명 |
|---|---|
| `git tag` | 태그 목록 |
| `git tag v1.0.0` | 현재 커밋에 태그 |
| `git tag -a v1.0.0 -m "첫 릴리스"` | 메시지가 있는 태그 (권장) |
| `git tag v0.9.0 <커밋>` | 특정 커밋에 태그 |
| `git push origin v1.0.0` | 태그 하나 올리기 |
| `git push --tags` | 태그 모두 올리기 |
| `git tag -d v1.0.0` | 로컬 태그 삭제 |
| `git push origin --delete v1.0.0` | 원격 태그 삭제 |

---

## 11. `.gitignore` — 추적 제외 파일

저장소 최상위에 `.gitignore` 파일을 만들고, Git이 무시할 파일 패턴을 적습니다.
**API 키가 들어 있는 `.env` 파일은 반드시 넣어서 GitHub에 올라가지 않게 하세요.**

```gitignore
# 비밀 정보
.env
*.key

# Python
__pycache__/
*.pyc
.venv/
.ipynb_checkpoints/

# 빌드 결과
dist/
build/

# 에디터·OS
.vscode/
.DS_Store
Thumbs.db

# 예외: 무시 패턴 중에서 이 파일은 포함
!.env.example
```

| 명령어 | 설명 |
|---|---|
| `git check-ignore -v 파일` | 그 파일이 왜 무시되는지 (어느 규칙 때문인지) 확인 |
| `git rm --cached .env` | 이미 커밋된 파일을 추적에서 제외 (`.gitignore`에 추가한 뒤) |
| `git status --ignored` | 무시된 파일까지 표시 |

> ⚠️ `.gitignore`는 **이미 커밋된 파일에는 적용되지 않습니다.** 먼저 `git rm --cached`로 추적을 끊어야 합니다.
> ⚠️ API 키를 실수로 push했다면 커밋을 지워도 이미 노출된 것으로 보고 **키를 즉시 재발급**하세요.
> 💡 언어별 템플릿: <https://github.com/github/gitignore>

---

## 12. GitHub 인증

GitHub는 2021년부터 push할 때 **계정 비밀번호를 받지 않습니다.** 다음 중 하나를 사용합니다.

### 방법 1: Git Credential Manager (Windows 기본, 가장 쉬움)

Git for Windows에 포함되어 있습니다. 처음 `git push`를 하면 브라우저 로그인 창이 뜨고, 로그인하면 이후 자동으로 인증됩니다.

| 명령어 | 설명 |
|---|---|
| `git config --global credential.helper` | 사용 중인 인증 도우미 확인 (`manager`이면 정상) |
| `git credential-manager github list` | 저장된 GitHub 계정 확인 |
| `git credential-manager github logout <아이디>` | 저장된 GitHub 계정 로그아웃 |

### 방법 2: SSH 키

```powershell
ssh-keygen -t ed25519 -C "메일@example.com"   # 키 생성 (엔터 연타 가능)
Get-Content ~/.ssh/id_ed25519.pub              # 공개키 출력 → 복사
# GitHub → Settings → SSH and GPG keys → New SSH key 에 붙여넣기
ssh -T git@github.com                          # 연결 확인
```

이후 원격 주소를 SSH 형식(`git@github.com:user/repo.git`)으로 사용합니다.

```powershell
git remote set-url origin git@github.com:user/repo.git
```

---

## 13. GitHub CLI (`gh`)

터미널에서 GitHub 저장소·PR·이슈를 다룰 수 있는 공식 도구입니다 (`winget install --id GitHub.cli`).

| 명령어 | 설명 |
|---|---|
| `gh auth login` | GitHub 로그인 (Git 인증도 함께 설정됨) |
| `gh auth status` | 로그인 상태 확인 |
| `gh repo create` | GitHub에 새 저장소 만들기 (대화형) |
| `gh repo create my-repo --public --source=. --push` | 현재 폴더로 공개 저장소를 만들고 바로 push |
| `gh repo clone user/repo` | 저장소 내려받기 |
| `gh repo view --web` | 현재 저장소를 브라우저로 열기 |
| `gh pr create` | Pull Request 만들기 |
| `gh pr list` | PR 목록 |
| `gh pr checkout 12` | 12번 PR을 내려받아 확인 |
| `gh pr merge 12` | 12번 PR 병합 |
| `gh issue create` | 이슈 만들기 |
| `gh issue list` | 이슈 목록 |

---

## 14. 자주 쓰는 작업 흐름

### 내 폴더를 GitHub에 처음 올리기

1. GitHub 웹사이트에서 **New repository**로 빈 저장소를 만듭니다 (README 등은 체크하지 않음).
2. 내 폴더에서:

```powershell
cd C:\Users\wins\AI_edu
git init
git add .
git commit -m "첫 커밋"
git branch -M main
git remote add origin https://github.com/<아이디>/<저장소이름>.git
git push -u origin main
```

### 매일 하는 작업

```powershell
git pull                       # 1. 최신 내용 받기
# ... 파일 수정 ...
git status                     # 2. 무엇이 바뀌었는지 확인
git add .                      # 3. 스테이징
git commit -m "기능 설명"       # 4. 커밋
git push                       # 5. 올리기
```

### 기능 브랜치로 작업하고 PR 보내기

```powershell
git switch -c feature/login    # 브랜치 만들고 이동
# ... 작업 + 커밋 ...
git push -u origin feature/login
gh pr create                   # 또는 GitHub 웹에서 "Compare & pull request"
# PR이 병합된 뒤
git switch main
git pull
git branch -d feature/login
```

### 다른 사람 저장소에 기여하기 (Fork)

```powershell
# GitHub에서 Fork 버튼으로 내 계정에 복사한 뒤
git clone https://github.com/<내아이디>/repo.git
cd repo
git remote add upstream https://github.com/<원래주인>/repo.git
git fetch upstream
git merge upstream/main        # 원본의 최신 내용 반영
```

### 커밋 메시지 작성 요령

```
feat: 로그인 기능 추가
fix: 파일 저장 시 한글 깨짐 수정
docs: README 설치 방법 보완
refactor: 데이터 로딩 함수 정리
chore: 패키지 버전 업데이트
```

> 💡 "무엇을 왜 바꿨는지"를 현재형으로 짧게 적으면 나중에 `git log`로 찾기 쉽습니다.

---

## 15. 자주 만나는 오류와 해결

| 오류 메시지 | 원인과 해결 |
|---|---|
| `'git' is not recognized...` | 설치 후 PATH 미반영 → VS Code·터미널을 완전히 다시 시작 |
| `Author identity unknown` | 이름·이메일 미설정 → `git config --global user.name / user.email` |
| `fatal: not a git repository` | 저장소가 아닌 폴더 → `git init` 하거나 올바른 폴더로 `cd` |
| `remote origin already exists` | 이미 연결됨 → `git remote set-url origin <URL>` |
| `! [rejected] ... (fetch first)` | 원격에 내가 없는 커밋이 있음 → `git pull` 후 다시 `git push` |
| `refusing to merge unrelated histories` | GitHub에서 README를 만든 저장소에 처음 push → `git pull origin main --allow-unrelated-histories` |
| `src refspec main does not match any` | 커밋이 하나도 없거나 브랜치 이름이 `master` → 먼저 커밋, `git branch -M main` |
| `Permission denied (publickey)` | SSH 키 미등록 → [12. GitHub 인증](#12-github-인증) 참고 |
| `Support for password authentication was removed` | 비밀번호로 인증 시도 → Credential Manager 로그인 또는 토큰·SSH 사용 |
| `CONFLICT (content): Merge conflict in ...` | 충돌 발생 → [6.2 충돌 해결](#62-충돌conflict-해결) 참고 |
| `LF will be replaced by CRLF` | 줄바꿈 변환 안내 (경고일 뿐, 무시해도 됨) |
| `File ... exceeds GitHub's file size limit of 100.00 MB` | 큰 파일 → `.gitignore`에 추가하거나 Git LFS(`git lfs track "*.bin"`) 사용 |
| `detached HEAD` | 브랜치가 아닌 커밋으로 이동한 상태 → `git switch main` (작업을 남기려면 `git switch -c 새브랜치`) |

---

## 참고 자료

- 공식 문서: <https://git-scm.com/docs>
- Pro Git 책 (한국어 무료): <https://git-scm.com/book/ko/v2>
- GitHub 문서: <https://docs.github.com/ko>
- GitHub CLI: <https://cli.github.com/manual/>
- 명령어 상세: `git help <명령어>`
