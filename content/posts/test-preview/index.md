---
title: "디자인 테스트용 임시 글"
date: 2026-04-07
draft: true
categories: ["테스트"]
tags: ["디자인", "프리뷰", "마크다운"]
description: "색상, 폰트, 레이아웃 확인용 임시 글입니다."
---

## 제목 스타일 (H2)

### 소제목 (H3)

#### 더 작은 제목 (H4)

---

## 링크 & 텍스트

일반 텍스트입니다. **굵은 텍스트**와 *기울임 텍스트*, 그리고 ~~취소선~~도 있습니다.

[일반 링크 - GitHub](https://github.com)는 이렇게 보이고, [또 다른 링크 - Hugo 공식문서](https://gohugo.io/)는 이렇게 보입니다. [방문한 적 있을 법한 링크 - Google](https://google.com)도 확인해보세요.

> 인용문은 이렇게 보입니다. 누군가의 말을 인용하거나 중요한 내용을 강조할 때 사용합니다.

---

## 목록

### 순서 없는 목록
- 첫 번째 항목
- 두 번째 항목
  - 중첩 항목 A
  - 중첩 항목 B
- 세 번째 항목

### 순서 있는 목록
1. Spring Boot로 API 개발
2. Docker로 컨테이너화
3. Kubernetes에 배포
4. ArgoCD로 GitOps 구성

---

## 코드

인라인 코드: `docker compose up -d` 이렇게 보입니다.

```java
@RestController
@RequestMapping("/api/v1/posts")
public class PostController {

    private final PostService postService;

    // 생성자 주입
    public PostController(PostService postService) {
        this.postService = postService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<PostResponse> getPost(@PathVariable Long id) {
        return ResponseEntity.ok(postService.findById(id));
    }
}
```

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
```

```bash
# 배포 스크립트
kubectl apply -f deployment.yaml
kubectl rollout status deployment/my-app
```

---

## 테이블

| 기술 | 용도 | 숙련도 |
|------|------|--------|
| Java / Spring Boot | 백엔드 API | ★★★★☆ |
| Docker / K8s | 컨테이너 & 오케스트레이션 | ★★★☆☆ |
| ArgoCD | GitOps CD | ★★★☆☆ |
| PostgreSQL | 데이터베이스 | ★★★☆☆ |

---

## 이미지 (플레이스홀더)

이미지가 들어갈 자리입니다. 실제 글에서는 아키텍처 다이어그램 등이 들어갑니다.

---

## 체크리스트

- [x] Hugo 설치
- [x] 테마 적용
- [x] 색상 커스터마이징
- [ ] 첫 글 작성
- [ ] GitHub Actions 배포

---

## 알림 / Hint 박스

{{< hint info >}}
**참고**: 이건 info 스타일 힌트 박스입니다.
{{< /hint >}}

{{< hint warning >}}
**주의**: 이건 warning 스타일 힌트 박스입니다.
{{< /hint >}}

{{< hint danger >}}
**위험**: 이건 danger 스타일 힌트 박스입니다.
{{< /hint >}}

---

## 각주

이것은 각주가 있는 문장입니다[^1]. 또 다른 각주도 있습니다[^2].

[^1]: 첫 번째 각주 내용
[^2]: 두 번째 각주 내용
