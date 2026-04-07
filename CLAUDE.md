# J.dev 기술 블로그

Hugo 정적 사이트 + seotax 테마 기반 기술 블로그.

- 사이트: https://j00j12i11.github.io/
- 테마: seotax (`themes/seotax/`)

## 프로젝트 구조

```
hugo.yaml          # Hugo 설정 (사이트명, 퍼머링크, 메뉴 등)
assets/css/_custom.scss  # 커스텀 스타일 (색상, 폰트, 코드블록)
content/posts/     # 블로그 글 (마크다운)
static/            # 정적 파일 (이미지, favicon 등)
layouts/           # 테마 오버라이드 레이아웃 (필요 시)
```

## 빌드 & 실행

```bash
hugo server -D     # 로컬 개발 서버 (드래프트 포함)
hugo                # 빌드 → public/
```

## 글 작성 규칙

- 경로: `content/posts/{카테고리}/{slug}/index.md`
- 퍼머링크: `/posts/:sections/:slug/`
- front matter 필수 필드: `title`, `date`, `draft`, `categories`, `tags`, `summary`
- 이미지는 글과 같은 디렉토리에 배치 (Page Bundle)

## 디자인 시스템

- 색상: 블루톤 라이트(`#f2f5f9`) / 다크 네이비(`#0a1628`) / 포인트 오렌지(`#f17300`)
- 폰트: Pretendard(본문) + JetBrains Mono(코드)
- 커스텀 스타일은 `assets/css/_custom.scss`에서 관리
- CSS 수정 시 덧붙이지 말고 원인 파악 후 깔끔하게 재작성

## 커밋 컨벤션

- 제목: 영어 Conventional Commits (`feat:`, `fix:`, `style:`, `docs:`, `content:`)
- 본문: 한국어
- `content:` 접두사는 글 추가/수정에 사용
