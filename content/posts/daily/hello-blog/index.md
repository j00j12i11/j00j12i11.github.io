---
title: "Hugo + GitHub Pages 블로그 개설 회고"
date: 2026-04-07
draft: false
categories: ["daily"]
tags: ["블로그", "Hugo", "GitHub Pages", "GitHub Actions", "회고"]
summary: "Hugo + GitHub Pages로 기술 블로그를 만드는 전 과정을 정리했다. 환경 세팅부터 배포, 커스터마이징까지."
---

## 블로그를 시작하는 이유

원래도 노션을 이용해 정리를 해왔지만, 개인적인 공간이다 보니 점점 대강 쓰고 넘어가게 된 것 같다.   
아예 퍼블릭한 공간에 자산으로 남을 수 있도록 블로그를 작성해보자는 생각이 들었다. 누군가가 볼 수 있다는 생각으로 신경 써서 가꾸고자 한다.

## 왜 Hugo인가

### Hugo

티스토리나 벨로그 같이 좋은 플랫폼들도 있지만 웬지 개발자답게 블로그를 운영하고 싶다는 생각이 들었다. 그래서 GitHub 기반의 블로그 방식에 대해 찾게 되었다.  
정적 사이트 생성기(SSG)를 찾다 보니 Jekyll, Gatsby, Hugo 등이 있었다. Hugo를 선택한 이유는.. 

- **빌드 속도가 빠르다** — Go 기반이라 수백 개의 페이지도 수 초 내에 빌드된다
- **마크다운 기반** — 별도의 에디터 없이 `.md` 파일로 글을 작성한다
- **단일 바이너리** — Node.js 의존성 없이 설치가 간단하다
- **커스터마이징 자유도가 높다** — 레이아웃, 스타일을 원하는 대로 수정할 수 있다

### seotax 테마

Hugo 테마 중 [seotax](https://github.com/minyeamer/hugo-seotax)를 선택했다. 사이드바 구성이 깔끔하고, 카테고리/태그 분류 구조가 잘 잡혀 있고, 검색 기능(Fuse.js 기반)도 내장되어 있어서 기술 블로그에 딱 맞았다. 다크 모드, 반응형 디자인, SEO 최적화까지 기본으로 지원한다.

### GitHub Pages + GitHub Actions

GitHub Pages는 GitHub에서 무료로 호스팅을 해준다. 추가로 GitHub Actions로 자동화를 적용하면 로컬에서 `git push`를 날리는 것만으로도 자동 배포가 가능하다.

## 환경 세팅

### Hugo 설치

macOS라서 Homebrew를 사용하였다.

```bash
brew install hugo
hugo version  # 설치 확인
```

### 프로젝트 생성 & 테마 적용

```bash
hugo new site blog
cd blog
git init
```

테마 적용은 보통 `git submodule`을 많이 쓰지만, 나는 직접 `git clone`으로 가져왔다.

```bash
git clone https://github.com/minyeamer/hugo-seotax.git themes/seotax
```

submodule 방식은 테마를 별도 레포지토리로 관리해서 업데이트가 편하다는 장점이 있다.  
하지만 나는 테마를 내 입맛대로 수정할 생각이었기 때문에 clone으로 가져와서 직접 관리하는 쪽을 택했다. 테마 원본에 의존하지 않고 자유롭게 건드릴 수 있다.

### hugo.yaml 설정

Hugo의 설정 파일에서 주요 항목을 설정한다.

```yaml
baseURL: "https://{username}.github.io/"
title: "블로그 이름"
theme: "seotax"
defaultContentLanguage: "ko"

# 퍼머링크 구조: /posts/카테고리/슬러그/
permalinks:
  posts: "/posts/:sections/:slug/"

# 마크다운 렌더링 설정
markup:
  goldmark:
    renderer:
      unsafe: true  # HTML 태그 허용
  highlight:
    noClasses: false
    codeFences: true
```

- `baseURL`
    - GitHub Pages 도메인.
    - `{username}.github.io` 형식으로 설정한다
- `permalinks`
  - URL 구조 정의. 
  - `:sections`은 카테고리 폴더명, `:slug`는 글 폴더명이 된다
- `unsafe: true`
  - 마크다운 안에서 raw HTML을 사용할 수 있게 허용한다. 
  - 이 글의 [색상 팔레트](#색상-팔레트)처럼 HTML을 직접 넣을 때 필요하다

### 로컬 서버 확인

```bash
hugo server -D  # -D 옵션: draft 글도 포함해서 미리보기
```

`http://localhost:1313`에서 실시간으로 확인할 수 있다. 파일을 수정하면 자동으로 새로고침된다.

## GitHub Pages 배포

### 레포지토리 생성

GitHub에서 `{username}.github.io` 이름으로 레포지토리를 생성한다.  
그 다음 Settings → Pages에서 Source를 **GitHub Actions**로 설정한다.

### GitHub Actions 워크플로우

`.github/workflows/deploy.yml` 파일을 생성한다.

```yaml
name: Deploy Hugo to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: '0.160.0'
          extended: true

      - run: hugo --minify

      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

각 부분이 무슨 역할을 하는지 하나씩 살펴보자.

#### on (트리거)

> ```yaml
> on:
>   push:
>     branches: [main]
>   workflow_dispatch:
> ```
>
> `main` 브랜치에 push하면 워크플로우가 자동으로 실행된다.  
> `workflow_dispatch`는 GitHub 웹에서 수동으로 실행 버튼을 누를 수 있게 해주는 옵션이다.

#### permissions (권한)

> ```yaml
> permissions:
>   contents: read
>   pages: write
>   id-token: write
> ```
>
> 워크플로우가 어떤 작업을 할 수 있는지 권한을 지정한다.  
> - `contents: read`: 레포지토리 코드 읽기
> - `pages: write`:  GitHub Pages 배포
> - `id-token: write`: 배포 인증 토큰 발급

#### concurrency (동시 실행 방지)

> ```yaml
> concurrency:
>   group: pages
>   cancel-in-progress: false
> ```
>
> 짧은 시간 안에 여러 번 push하면 배포가 동시에 실행될 수 있으므로...  
>  `concurrency`를 설정하여 이전 배포가 끝날 때까지 다음 배포가 대기한다.

#### build 작업

> ```yaml
> jobs:
>   build:
>     runs-on: ubuntu-latest
> ```
>
> GitHub Actions는 이렇게 클라우드 서버를 무료로 빌려주는데 그 서버에서 `runs-on: ubuntu-latest` 우분투로 실행한다.  
> 아마 도커 이미지 지정하는 것과 비슷한 거 아닐까?

빌드 과정은 4단계로 진행된다:

1. **checkout**: 레포지토리 코드를 가상 머신에 내려받는다.
2. **Hugo 설치**: `extended` 버전을 설치한다.  
extended는 SCSS 컴파일을 지원하는 버전으로, 커스텀 스타일을 사용하려면 필요하다.
3. **빌드**: `hugo --minify`로 사이트를 빌드한다.  
`--minify`는 HTML, CSS, JS를 압축해서 파일 크기를 줄여준다.
4. **아티팩트 업로드**: 빌드 결과물(`public/` 폴더)을 **아티팩트**로 업로드한다.  
아티팩트는 GitHub Actions에서 작업 간에 파일을 전달하기 위한 임시 저장소라고 보면 된다.  
build 작업에서 만든 파일을 deploy 작업에서 쓸 수 있게 넘겨주는 역할이다.

#### deploy 작업

> ```yaml
>   deploy:
>     needs: build
> ```
>
> `needs: build`는 build 작업이 완료된 후에야 실행된다는 의미다.   
> build에서 올린 아티팩트를 받아서 GitHub Pages에 최종 배포한다.

---

이게 전부다. 이제 글을 작성하고 `git push`하면 자동으로 사이트에 반영된다.

## 커스터마이징

### 색상 팔레트

SSAFY에서 알게 된 분이 [Coolors](https://coolors.co)라는 색상 팔레트 생성 사이트를 추천해줬다. (항상 꿀팁 전수받는 중..) 스페이스바를 누르면 랜덤으로 팔레트를 생성해주는데 거기서 **블루 톤 + 포인트 오렌지** 조합을 골랐다.

<div style="display:flex; border-radius:12px; overflow:hidden; font-family:monospace; text-align:center; margin:1.5rem 0;">
  <div style="flex:1; background:#054A91; color:#fff; padding:2rem 0.5rem 1rem;">
    <div style="font-size:0.85rem; margin-top:auto;">#054A91</div>
    <div style="font-size:0.75rem; opacity:0.8;">Steel Azure</div>
  </div>
  <div style="flex:1; background:#3E7CB1; color:#fff; padding:2rem 0.5rem 1rem;">
    <div style="font-size:0.85rem;">#3E7CB1</div>
    <div style="font-size:0.75rem; opacity:0.8;">Steel Blue</div>
  </div>
  <div style="flex:1; background:#81A4CD; color:#fff; padding:2rem 0.5rem 1rem;">
    <div style="font-size:0.85rem;">#81A4CD</div>
    <div style="font-size:0.75rem; opacity:0.8;">Wisteria Blue</div>
  </div>
  <div style="flex:1; background:#DBE4EE; color:#333; padding:2rem 0.5rem 1rem;">
    <div style="font-size:0.85rem;">#DBE4EE</div>
    <div style="font-size:0.75rem; opacity:0.6;">Alice Blue</div>
  </div>
  <div style="flex:1; background:#F17300; color:#fff; padding:2rem 0.5rem 1rem;">
    <div style="font-size:0.85rem;">#F17300</div>
    <div style="font-size:0.75rem; opacity:0.8;">Harvest Orange</div>
  </div>
</div>
<p style="text-align:center; font-size:0.8rem; color:#888; margin-top:-0.5rem;">Coolors에서 뽑은 이 블로그의 색상 팔레트</p>

이 팔레트를 기반으로 블로그의 배경색, 텍스트 색상, 포인트 색상을 `_custom.scss`에서 설정했다. 라이트 모드 배경은 Alice Blue 계열, 다크 모드는 네이비, 링크 호버와 강조 요소에는 Harvest Orange를 적용했다.

### 폰트

본문에는 **Pretendard**, 코드 블록에는 **JetBrains Mono**를 적용했다. 둘 다 개발자 사이에서 가독성이 좋다고 평가받는 폰트다. (솔직히 알못이라 지금 잘 적용된 건지 사실 잘 모르겠다. 일단 눈에 거슬리지 않으면 된 거 아닐까.)

### 블로그 헤더 커스터마이징

이 블로그의 메인 페이지 헤더에는 보르조이 강아지 캐릭터가 들어가 있다. 이 과정이 가장 많은 삽질을 했던 부분이다.

#### 1단계: AI로 이미지 생성

최근에 인스타 짤로 본 보르조이의 긴 목이 계속 생각이 나서 그냥 보르조이로 블로그를 꾸미게 되었다. 근데 인터넷에서 사진을 가져다 쓰자니 저작권이 신경 쓰이고 직접 그리자니 물론 능력이 없었다.   
결국 젬미니와 지피티에게 부탁하기로 했다.  

Google Gemini와 ChatGPT(DALL-E) 둘 다 열심히 영어로 프롬프트를 넣어가면서 마음에 드는 결과가 나올 때까지 돌렸다. "minimal flat style borzoi illustration, transparent background, ..." 같은 식으로. 

그런데 Gemini한테 투명 배경 PNG를 달라고 했더니, 마치 PNG 이미지 마냥 체크무늬를 배경에 그려넣은 이미지를 줬다... 어디서 이런 꼼수를 알아온 건지. 그래도 프로필에 사용한 보르조이를 잘 그려줘서 넘어가기로 했다.  

투명 배경은 ChatGPT(DALL-E)를 통해 목이 긴 보르조이의 옆모습 이미지를 얻을 수 있었다.

#### 2단계: 이미지 분할

헤더 디자인상 강아지의 몸통과 얼굴을 사이트 제목 양쪽에 배치해야 했다. 처음에는 몸통과 얼굴을 따로 생성해봤는데, 역시나 프롬프트 짜기도 어렵고, 분리된 두 개의 이미지를 이어지게 그리는 건 AI가 잘 못하는 것 같았다. (GPT에게 친구비를 안 주고 있어서 더한 부탁을 할 수 없었음..)

해결 방법: **한 장의 긴 보르조이 이미지를 생성한 뒤, Python(PIL)으로 두 조각으로 크롭**했다.

```python
from PIL import Image

img = Image.open("new_long_zoi_original.png")
# 몸통 부분과 얼굴 부분을 각각 잘라서 저장
body = img.crop((0, 0, body_width, img.height))
face = img.crop((face_x, 0, img.width, img.height))
```

#### 3단계: CSS 삽질기

**시도 1 — CSS pseudo-element 방식**

처음에는 `::before`와 `::after`로 배경 이미지를 넣으려 했다. 하지만 처음엔 배경이 불투명이어서 이미지가 박스처럼 보이는 문제가 있었고 `overflow: hidden` 때문에 이미지가 잘리기도 했다.

**시도 2 — img 태그 + Flexbox (최종)**

결국 `<img>` 태그를 직접 사용하고 Flexbox로 배치하는 방식으로 바꿨다.

```html
<div class="longzoi-header">
  <img class="zoi-img zoi-body" src="/zoi-body.png" />
  <div class="longzoi-text">
    <h1>J.dev</h1>
  </div>
  <img class="zoi-img zoi-face" src="/zoi-face.png" />
</div>
```

```scss
.longzoi-header {
  display: flex;
  align-items: flex-end;

  img.zoi-body {
    height: 140px;
    width: auto;
  }

  img.zoi-face {
    height: 56px;
    width: auto;
    align-self: flex-start;
  }

  .longzoi-text {
    flex: 1;
    text-align: center;
  }
}
```

#### 4단계: 테마 CSS 충돌 해결

이미지 높이를 계속 변경시키면서 맞추고자 했는데 적용이 계속 안 됐다. 원인은 테마의 CSS에 있는 `.markdown img { height: auto }` 규칙 때문으로 이게 커스텀 스타일을 덮어쓰고 있었다.

해결: `img.zoi-body`처럼 더 구체적인(specificity가 높은) 선택자를 사용해서 테마 스타일보다 우선하게 만들었다.

#### 5단계: 반응형 대응

데스크톱에서 잘 보이던 레이아웃이 모바일에서는 이미지가 너무 커서 깨졌다. 미디어 쿼리를 추가해서 해결했다.

```scss
@media screen and (max-width: 768px) {
  .longzoi-header {
    img.zoi-body { height: 80px; }
    img.zoi-face { height: 32px; }
    .longzoi-text h1 { font-size: 1.5rem; }
  }
}
```

#### 6단계: 이미지 최적화

원본 프로필 이미지가 **6.9MB**(2048x2048)였어서 페이지 새로고침할 때 버벅이는 느낌이 있었다. 400x400으로 리사이즈해서 **142KB**로 줄였더니 깔끔하게 로드된다.

> 사실 지금 이미지를 두 조각으로 자른 부분에서 목이 잘린 게 좀 어색하다. 다음에 자연스럽게 이어지도록 다시 다듬을 예정이다.

## 마무리

블로그 만들라고 매번 생각만 하고 시도해보지 못했는데 LLM과 AI의 발전이 나를 이끌어주었다. 특히 클로드가 1대1 코칭으로 붙어서 잘 도와주고 있다. ~~클로드는 신이야! 클로드는 신이야? 클로드는 신이야!~~

앞으로 이 블로그에 백엔드 개발, DevOps, CS 기초, 프로젝트 회고 같은 글을 꾸준히 써볼 생각이다. 알고리즘도 열심히 풀고 있긴 한데, 이건 어떻게 써야 할지 고민이 된다. (내 보잘 것 없는 풀이를 올려도 되는 걸까?)   
아무튼 최소 ***~~2주~~*** **1달에 한번** 쓰는 것을 목표로 해보자.
