# lsw0604.log 프로젝트 명세

Seongwon Lee의 개인 블로그. AstroPaper 테마(v6.1.0) 기반 Astro 정적 사이트이며 GitHub Pages로 배포한다.

- 사이트: https://lsw0604.github.io/
- 저장소: https://github.com/lsw0604/lsw0604.github.io (public, 기본 브랜치 `main`)
- 스택: Astro 7, Tailwind CSS 4, TypeScript, pnpm, Node.js >= 22.12, Pagefind(검색)
- 사이트 언어 `ko`, 시간대 `Asia/Seoul`

## 디렉터리 구조

| 경로 | 내용 |
|---|---|
| `astro-paper.config.ts` | 사이트 정보·기능·소셜/공유 링크. **설정 변경은 이 파일에서** |
| `astro.config.ts` | Astro 설정. i18n `locales: ["ko"]`, `defaultLocale: "ko"` |
| `src/config.ts` | 위 설정에 기본값을 채운 내부용 모듈. 직접 수정하지 않는다 |
| `src/content.config.ts` | 글(`posts`)·페이지(`pages`) frontmatter 스키마 |
| `src/content/posts/` | 블로그 글 (`.md`, `.mdx`) |
| `src/content/pages/about.md` | 소개 페이지 본문 |
| `src/i18n/lang/ko.ts` | UI 한국어 문구 (메뉴, 버튼, 안내문) |
| `src/pages/index.astro` | 홈. 상단 인사말·소개 문단이 하드코딩돼 있다 |
| `src/pages/og.png.ts`, `src/pages/posts/[...slug]/index.png.ts` | OG 이미지 생성 (satori) |
| `src/styles/theme.css` | 색상 테마 |
| `src/assets/icons/socials/` | 소셜/공유 아이콘. config의 `name`은 여기 SVG 파일명과 일치해야 한다 |
| `public/` | 그대로 복사되는 정적 파일 (favicon 등) |
| `.github/workflows/deploy.yml` | `main` 푸시 시 빌드 후 GitHub Pages 배포 |
| `.github/workflows/ci.yml` | PR에서 lint → format:check → build 검사 |

## 글 작성 규칙

`src/content/posts/<slug>.md` 파일을 만든다. 파일명(영문 kebab-case 권장)이 URL이 된다: `hello-world.md` → `/posts/hello-world/`.

```md
---
title: "글 제목"                         # 필수
description: "목록·SEO에 쓰이는 요약"      # 필수
pubDatetime: 2026-10-22T09:00:00+09:00   # 필수. 반드시 +09:00 오프셋 포함
modDatetime: 2026-10-23T10:00:00+09:00   # 선택. 수정 시각
tags:                                    # 선택. 생략 시 ["others"]
  - astro
featured: false                          # 선택. true면 홈 '추천 글'에 표시
draft: false                             # 선택. true면 어디에도 노출 안 됨
---
```

- `ogImage`(선택): 공유 미리보기 이미지. 생략하면 자동 생성된다. 지정할 때는 이미지 경로를 넣고, 빈 문자열은 넣지 않는다.

- 폴더로 묶으면 URL에 폴더명이 들어간다: `posts/dev/foo.md` → `/posts/dev/foo/`. 폴더·파일명이 `_`로 시작하면 URL에서 빠지거나(폴더) 글에서 제외된다(파일).
- 한글 태그 가능: `일상` → `/tags/일상/`.
- 이미지: `src/assets/images/`에 두고 상대 경로로 참조한다 (`![설명](../../assets/images/foo.png)`). 빌드 시 최적화된다.
- **예약 글 주의**: `pubDatetime`이 미래면 프로덕션 빌드에서 제외된다. 정적 사이트라 시간이 지나도 자동 공개되지 않고, 그 이후 **다시 배포해야** 노출된다 (Actions 탭에서 `Deploy to GitHub Pages` 수동 실행 가능).
- 개발 서버(`pnpm dev`)에서는 `draft`가 아닌 글은 예약 글도 보인다.

## 배포 파이프라인

`git push origin main` → GitHub Actions `deploy.yml` (GitHub 가상 머신에서 실행):

1. Build 잡: checkout → pnpm 11.3.0 → Node.js 24 (pnpm 캐시) → `pnpm install --frozen-lockfile` → `pnpm run build` → `dist/` 업로드
2. Deploy 잡 (Build 성공 시에만): `actions/deploy-pages`로 GitHub Pages 배포

- `pnpm run build` = `astro check && astro build && pagefind --site dist && cp -r dist/pagefind public/`
- 빌드 실패 시 배포되지 않고 사이트는 직전 버전 유지. 소요 약 1분.
- `main` 직접 푸시에는 lint/format 검사가 없다 (의도된 설계: 글 작성 흐름에서 불필요). 코드 변경 시에는 로컬에서 직접 돌린다.
- 상태 확인: `gh run list -R lsw0604/lsw0604.github.io --limit 3`

## 작업별 검증

| 작업 | 실행할 명령 |
|---|---|
| 글 추가·수정 | `pnpm build` (frontmatter 스키마·이미지 경로 검사) |
| 코드·설정 변경 | `pnpm lint && pnpm format:check && pnpm build` |
| 화면 확인 | `astro dev --background` 후 http://localhost:4321 , 끝나면 `astro dev stop` |

- `format:check`는 `.md` 글도 검사한다(`.mdx` 제외). 실패하면 `pnpm format`으로 고친다.
- 로컬 pnpm 버전(12.x)과 CI(11.3.0)가 다르다. `pnpm install` 후 `pnpm-lock.yaml`이 바뀌면 의도한 변경인지 확인한다.

## 알려진 함정

- **OG 이미지 한글 깨짐**: OG 이미지 폰트(Google Sans Code)에 한글이 없어 한글 제목이 □로 렌더링된다. 사용자가 나중에 직접 처리하기로 했으므로 요청 없이 고치지 않는다. 고칠 때는 `astro.config.ts`의 `fonts`에 한글 폰트(예: Noto Sans KR)를 추가하고 두 OG 라우트의 satori `fonts` 배열에 함께 넣는다.
- **GitHub Pages 배포 방식**: 저장소 Pages 설정은 `build_type: workflow`(GitHub Actions)여야 한다. legacy(브랜치 배포)로 바뀌면 소스가 그대로 올라가 사이트가 깨진다. 확인: `gh api repos/lsw0604/lsw0604.github.io/pages --jq .build_type`
- **UI 문구 언어**: `src/i18n/index.ts`는 해당 locale 파일이 없으면 영어로 폴백한다. 문구를 바꿀 땐 `ko.ts`를 수정하고 `en.ts`는 건드리지 않는다. 새 키를 추가하면 `src/i18n/types.ts`와 `en.ts`, `ko.ts` 모두에 넣어야 타입 검사를 통과한다.
- **사이트 URL·경로**: `site.url`이 사용자 페이지 루트(`/`)라서 Astro `base` 설정이 없다. 저장소 이름을 바꾸면 URL과 `base`, `editPost.url`을 함께 바꿔야 한다.
- **기본 OG 이미지**: `public/default-og.jpg`가 있으면 그 파일을, 없으면 자동 생성된 `/og.png`를 쓴다 (현재 없음).
- `package.json`은 끝 줄바꿈이 있어야 `format:check`를 통과한다.

## 하지 말 것

- `dist/`, `public/pagefind/`, `.omc/`는 커밋하지 않는다 (`.omc/`는 Claude 플러그인 로컬 상태).
- `LICENSE`의 원작자(Sat Naing) 저작권 표기를 지우지 않는다 (MIT 조건).
- 사용자의 이메일 등 개인정보를 사이트 설정·글에 임의로 넣지 않는다.
- 요청 없이 `main`에 푸시하거나 GitHub 저장소 설정을 바꾸지 않는다.

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)
