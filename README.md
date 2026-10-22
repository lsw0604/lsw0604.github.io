# lsw0604.log

Seongwon Lee의 개인 블로그입니다. [AstroPaper](https://github.com/satnaing/astro-paper) 테마를 기반으로 만들었어요.

- 배포 주소: https://lsw0604.github.io/

## 개발

```bash
pnpm install
pnpm dev       # 로컬 개발 서버 (http://localhost:4321)
pnpm build     # 타입 체크 + 빌드 + Pagefind 검색 인덱스 생성
pnpm preview   # 빌드 결과 미리보기
```

## 글 쓰기

`src/content/posts/`에 Markdown(`.md`) 또는 MDX(`.mdx`) 파일을 추가합니다.

```md
---
title: "글 제목"
description: "글 요약"
pubDatetime: 2026-10-22T09:00:00+09:00
tags:
  - 태그
draft: false
---

본문
```

- 파일명이나 폴더명이 `_`로 시작하면 글 목록에서 제외돼요.
- `featured: true`를 넣으면 홈의 추천 글에 표시돼요.

## 설정

사이트 정보·기능 설정은 [`astro-paper.config.ts`](astro-paper.config.ts)에서, UI 한국어 문구는 [`src/i18n/lang/ko.ts`](src/i18n/lang/ko.ts)에서 수정합니다.

## License

테마 원본은 MIT 라이선스를 따릅니다. 자세한 내용은 [LICENSE](LICENSE)를 참고하세요.
