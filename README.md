# Cursor PR Reviewer 🤖

Cursor IDE와 GitHub API를 활용한 자동화된 Pull Request 코드 리뷰 도구입니다.

## Overview

Cursor PR Reviewer는 AI 기반 코드 분석을 통해 GitHub Pull Request를 자동으로 리뷰하고, 발견된 이슈에 대해 자동으로 코멘트를 게시하는 도구입니다. Cursor IDE의 AI 에이전트가 시니어 소프트웨어 엔지니어처럼 코드를 분석하여 실제 문제, 버그, 보안 이슈, 중요한 코드 품질 문제를 식별합니다.

### 주요 기능

- ✅ **자동 PR Diff 분석**: GitHub CLI를 사용하여 PR의 변경사항을 자동으로 가져옵니다
- 🤖 **AI 기반 코드 리뷰**: Cursor AI가 코드의 실제 문제점을 식별합니다
- 💬 **자동 코멘트 게시**: GitHub API를 통해 특정 라인에 리뷰 코멘트를 자동으로 게시합니다
- 🎯 **실용적인 리뷰**: 스타일이나 포맷팅이 아닌 실제 문제에만 집중합니다
- 🌐 **한국어 지원**: 모든 리뷰 코멘트는 한국어로 작성됩니다

## Prerequisites

다음 도구들이 설치되어 있어야 합니다:

- **GitHub CLI (`gh`)**: [설치 가이드](https://cli.github.com/)
- **Python 3.x**: PR diff 파싱용
- **bash**: 스크립트 실행 환경
- **jq**: JSON 파싱용
- **curl**: GitHub API 호출용
- **Cursor IDE**: AI 기반 코드 분석용

또한 다음 권한이 필요합니다:

- GitHub Personal Access Token (repo 권한 포함)
- 리뷰하려는 저장소에 대한 읽기/쓰기 권한

## Installation & Setup

### 1. GitHub CLI 설치 및 인증

```bash
# GitHub CLI 설치 (Ubuntu/Debian)
sudo apt update
sudo apt install gh

# GitHub CLI 설치 (Fedora/RHEL/CentOS)
sudo dnf install gh

# GitHub CLI 인증
gh auth login
```

### 2. 환경 변수 설정

`cursor-tools/.env` 파일을 생성하고 다음 내용을 추가합니다:

```bash
# GitHub Personal Access Token
GITHUB_TOKEN=your_github_token_here

# 프로젝트 루트 경로
PROJECT_ROOT=/path/to/your/project
```

**GitHub Token 생성 방법:**

1. GitHub Settings > Developer settings > Personal access tokens > Tokens (classic)
2. "Generate new token (classic)" 클릭
3. `repo` 권한 선택
4. 토큰 생성 후 복사하여 `.env` 파일에 추가

### 3. Cursor 룰 적용

`.cursor/rules/github-pr-review.mdc` 파일이 자동으로 Cursor IDE에 적용됩니다. 이 파일은 AI 에이전트가 PR 리뷰를 수행할 때 따라야 할 가이드라인을 포함합니다.

## Usage

### 사용 방법 (정말 간단합니다!)

Cursor IDE에서 채팅을 열고 다음과 같이 요청하면 됩니다:

```
123번 PR 리뷰해줘
```

그러면 AI가 자동으로:

1. ✅ PR diff를 가져오고
2. ✅ 코드를 분석하여 이슈를 찾고
3. ✅ GitHub에 자동으로 코멘트를 게시합니다

### 예시

**기본 리뷰 요청:**

```
PR #456 리뷰 부탁해
```

**특정 저장소의 PR 리뷰:**

```
owner/repo의 PR #789 리뷰해줘
```

**리뷰 완료 후 확인:**
AI가 모든 코멘트를 게시하고 나면, 채팅에서 완료 메시지를 확인할 수 있습니다:

- 이슈가 발견된 경우: "3개의 리뷰 코멘트를 GitHub에 게시했습니다."
- 이슈가 없는 경우: "이슈를 찾지 못했습니다. 승인 코멘트를 게시했습니다."

## How It Works (내부 동작 원리)

사용자가 "123번 PR 리뷰해줘"라고 요청하면, Cursor AI는 다음과 같은 순서로 작업을 수행합니다:

1. **PR Diff 가져오기** → `fetch_pr_diff.py` 실행
2. **코드 분석** → AI가 diff를 분석하여 이슈 식별
3. **코멘트 게시** → 각 이슈마다 `gh-pr-comment.sh` 실행
4. **승인/완료** → 이슈가 없으면 `gh-pr-general-comment.sh`로 승인 코멘트 게시

### 사용되는 스크립트들

Cursor AI가 백그라운드에서 자동으로 실행하는 스크립트들입니다:

### `fetch_pr_diff.py`

GitHub CLI를 사용하여 PR의 diff를 가져오고 파싱합니다.

**AI가 실행하는 명령어:**

```bash
python3 cursor-tools/fetch_pr_diff.py <PR_NUMBER>
```

**기능:**

- 현재 git 저장소의 remote URL에서 owner/repo 정보 추출
- `gh pr diff` 명령으로 PR diff 가져오기
- diff를 AI가 이해하기 쉬운 형식으로 파싱
- 추가된 라인(`+`)과 삭제된 라인(`-`)을 명확히 표시

**출력 형식:**

```
## File: 'src/file.py'

@@ ... @@
__new hunk__
11  unchanged code line
12 +new code line added in the PR
13  unchanged code line
```

### `gh-pr-comment.sh`

특정 파일의 특정 라인에 리뷰 코멘트를 게시합니다.

**AI가 실행하는 명령어:**

```bash
cursor-tools/gh-pr-comment.sh pr review <PR_NUMBER> \
  --comment -b "<리뷰 코멘트>" \
  --path <FILE_PATH> \
  --line <LINE_NUMBER>
```

**파라미터:**

- `<PR_NUMBER>`: Pull Request 번호
- `<리뷰 코멘트>`: 게시할 코멘트 내용
- `<FILE_PATH>`: 코멘트를 달 파일 경로
- `<LINE_NUMBER>`: 코멘트를 달 라인 번호

**동작 방식:**

1. PR의 최신 커밋 ID를 GitHub API로 조회
2. 제공된 정보로 JSON payload 생성
3. GitHub API를 통해 코멘트 게시

### `gh-pr-general-comment.sh`

PR에 일반 코멘트를 게시합니다 (특정 라인이 아닌 전체 PR에 대한 코멘트).

**AI가 실행하는 명령어:**

```bash
cursor-tools/gh-pr-general-comment.sh pr comment <PR_NUMBER> \
  --comment -b "<코멘트 내용>"
```

**파라미터:**

- `<PR_NUMBER>`: Pull Request 번호
- `<코멘트 내용>`: 게시할 코멘트 내용

**사용 예시:**

- 리뷰 완료 알림
- 승인 메시지
- 전반적인 피드백

## Review Guidelines

### 리뷰 기본 원칙

Cursor AI는 다음 원칙에 따라 PR을 리뷰합니다:

- 🎯 **실제 문제에만 집중**: 버그, 보안 이슈, 중요한 코드 품질 문제만 지적
- ❌ **스타일 코멘트 지양**: 코드 포맷팅, 스타일, 사소한 개선사항은 무시
- 🚫 **긍정적 피드백 지양**: 칭찬이나 "좋습니다" 같은 코멘트는 작성하지 않음
- 📝 **실행 가능한 코멘트만**: 구체적이고 실행 가능한 조치만 제안
- 🔍 **새 코드만 리뷰**: PR diff에서 추가된 라인(`+`)만 검토

### 코멘트 작성 규칙

- 모든 리뷰 코멘트는 **한국어**로 작성됩니다
- 마크다운 포맷팅을 사용할 수 있습니다
- 특수 문자나 코드 블록은 사용하지 않습니다
- 파일 끝의 개행(newline) 문제는 무시합니다

### 승인 기준

- **실행 가능한 이슈가 없을 때만 승인**
- 승인 메시지: "이슈가 없습니다. 승인합니다." (간결하게)
- 이슈가 하나라도 있으면 승인하지 않음

## Environment Variables

### `GITHUB_TOKEN`

**필수**: GitHub Personal Access Token

- **용도**: GitHub API 인증
- **필요 권한**: `repo` (전체 저장소 제어)
- **생성 위치**: GitHub Settings > Developer settings > Personal access tokens

### `PROJECT_ROOT`

**필수**: 프로젝트 루트 디렉토리의 절대 경로

- **용도**: 스크립트 실행 시 작업 디렉토리 지정
- **형식**: `/absolute/path/to/project`
- **예시**: `/Users/username/projects/my-repo`

## Troubleshooting

### "Error: GITHUB_TOKEN environment variable is not set"

**원인**: `.env` 파일이 없거나 `GITHUB_TOKEN`이 설정되지 않음

**해결**:

```bash
# cursor-tools/.env 파일 생성
echo "GITHUB_TOKEN=your_token_here" > cursor-tools/.env
echo "PROJECT_ROOT=$(pwd)" >> cursor-tools/.env
```

### "Error fetching PR diff"

**원인**:

- PR 번호가 잘못되었거나
- 저장소 접근 권한이 없거나
- GitHub CLI 인증이 안 됨

**해결**:

```bash
# GitHub CLI 재인증
gh auth login

# 저장소 접근 확인
gh pr view <PR_NUMBER>
```

### "Failed to add review comment"

**원인**:

- 잘못된 파일 경로
- 존재하지 않는 라인 번호
- 이미 머지된 PR

**해결**:

- 파일 경로가 PR에 실제로 존재하는지 확인
- 라인 번호가 diff의 새 코드(`+`)에 해당하는지 확인
- PR이 아직 오픈 상태인지 확인

### "Could not determine repository owner and name"

**원인**: git remote URL이 표준 형식이 아님

**해결**:

```bash
# remote URL 확인
git config --get remote.origin.url

# 표준 형식으로 변경 (HTTPS)
git remote set-url origin https://github.com/owner/repo.git

# 또는 SSH
git remote set-url origin git@github.com:owner/repo.git
```

## Project Structure

```
cursor-pr-reviewer/
├── .cursor/
│   └── rules/
│       └── github-pr-review.mdc    # Cursor AI 리뷰 가이드라인
├── cursor-tools/
│   ├── fetch_pr_diff.py            # PR diff 가져오기 스크립트
│   ├── gh-pr-comment.sh            # 라인별 코멘트 게시 스크립트
│   ├── gh-pr-general-comment.sh    # 일반 코멘트 게시 스크립트
│   └── .env                        # 환경 변수 (gitignore됨)
└── README.md
```

---

**Made with ❤️ using Cursor IDE**
