# J.dev

> 기술 블로그(노력형)

**Hugo** + **GitHub Pages**로 운영하는 기술 블로그입니다.

🔗 **https://j00j12i11.github.io**

---

## Tech Stack

| 구분 | 기술 |
|------|------|
| SSG | [Hugo](https://gohugo.io) |
| Theme | [seotax](https://github.com/minyeamer/hugo-seotax) |
| Hosting | GitHub Pages |
| CI/CD | GitHub Actions |
| Style | SCSS (Pretendard + JetBrains Mono) |

## Project Structure

```
content/posts/     # 블로그 글 (마크다운)
assets/css/        # 커스텀 스타일
static/            # 정적 파일 (이미지, favicon)
layouts/           # 테마 오버라이드 레이아웃
themes/seotax/     # Hugo 테마
.github/workflows/ # 자동 배포 워크플로우
```

## Development

```bash
# 로컬 개발 서버 (드래프트 포함)
hugo server -D

# 빌드
hugo
```

## Deploy

`main` 브랜치에 push하면 GitHub Actions가 자동으로 빌드 & 배포합니다.
