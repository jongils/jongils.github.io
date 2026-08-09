# jongils.github.io

[jongils](https://github.com/jongils)의 개인 GitHub Pages 사이트입니다. 별도의 빌드 도구나 프레임워크 없이 순수 HTML/CSS/JavaScript로만 만들어졌고, `main` 브랜치에 푸시되는 즉시 [https://jongils.github.io](https://jongils.github.io)에 배포됩니다.

## 구성

```
.
├── index.html                        # 메인 랜딩 페이지
├── links.html                        # 링크 매니저(북마크 관리) 페이지
├── data/
│   ├── links.json                    # 링크 매니저가 사용하는 북마크 데이터
│   └── repos.json                    # GitHub repo 타일에 쓰이는 데이터 (자동 생성)
├── .github/workflows/
│   └── update-repos.yml              # repo 목록을 주기적으로 갱신하는 Actions 워크플로우
└── LICENSE                           # Apache License 2.0
```

## 페이지 소개

### `index.html` — 메인 페이지
- 터미널 스타일 UI로 자기소개
- **Link Manager**: 상단에 별도 섹션으로 배치된 배너 카드, 클릭 시 `links.html`로 이동
- **Projects**: `data/repos.json`의 `publicRepos`를 읽어와 공개 GitHub repo를 타일(이름/설명/언어/★/최근 업데이트일)로 자동 렌더링
- **Secret Panel**: 페이지 하단의 `[root]` 트리거를 눌러 비밀번호를 입력하면, `data/repos.json`의 `privateRepos`를 같은 형태의 타일로 보여줌
  - ⚠️ 이 비밀번호 보호는 브라우저에서 SHA-256 해시를 비교하는 클라이언트 사이드 검증일 뿐, 실질적인 접근 제어가 아닙니다. `data/repos.json`은 공개 정적 파일이라 private repo의 이름·설명 등 메타데이터는 이미 누구나 열람 가능한 상태로 배포됩니다.

### `links.html` — 링크 매니저
- 저장된 북마크를 검색/카테고리별로 탐색
- `[admin]` 트리거로 비밀번호 인증 후 관리자 패널에서 북마크 추가/수정/삭제 가능
- 관리자 패널에 입력한 GitHub Personal Access Token(브라우저 `localStorage`에만 저장)으로 GitHub Contents API를 직접 호출해 `data/links.json`을 커밋

## 자동화 — Repo 타일 갱신

`.github/workflows/update-repos.yml`이 매일 1회(UTC 00:00) 실행되며, 필요 시 **Actions 탭에서 수동 실행(`workflow_dispatch`)**도 가능합니다.

1. `REPOS_SCAN_TOKEN` 시크릿(Repository secrets에 등록된 GitHub PAT)으로 `GET /user/repos`를 호출해 계정 소유의 모든 repo(공개+비공개, fork 포함, archived 제외)를 조회
2. `private` 여부로 `publicRepos` / `privateRepos`로 분리하고 각각 최신 업데이트순 정렬
3. `data/repos.json`을 재생성해 변경이 있을 때만 `main`에 커밋
4. 이 커밋이 GitHub Pages 자동 배포(`pages-build-deployment`)를 다시 트리거해 사이트에 최신 목록이 반영됨

워크플로우가 정상 동작하려면 저장소 **Settings → Secrets and variables → Actions**에 `REPOS_SCAN_TOKEN`(Classic PAT `repo` scope, 또는 Fine-grained PAT `All repositories` + `Metadata: Read-only`)이 등록되어 있어야 합니다.

## 로컬에서 미리보기

빌드 과정이 없으므로 정적 파일 서버만 있으면 됩니다.

```bash
python3 -m http.server 8000
# http://localhost:8000/index.html 접속
```

`data/repos.json`을 원하는 테스트 데이터로 바꿔보면 Projects/Secret Panel 타일 렌더링을 확인할 수 있습니다.

## 라이선스

[Apache License 2.0](LICENSE)
