## 클라이언트 사이드 렌더링 (CSR)

모든 UI가 클라이언트에서 생성되며, 처음 요청시 전체 웹 애플리케이션을 모두 로드하게 된다. CSR은 페이지 새로고침 없이 SPA를 구축할 수 있게 한다.

단점

- 이미지 표시, 이벤트 처리 등 페이지의 복잡성이 올라가면 번들 용량이 증가하여 FCP, TTI를 증가시키게 된다.
- 또한 크롤러가 CSR로 만든 페이지는 색인하지 못해 SEO에 영향을 미칠 수 있다.

## 서버 사이드 렌더링 (SSR)

사용자 쿠키 정보나 요청 데이터를 기반으로 하는 개인 맞춤형 데이터를 포함하는 페이지에 적합하다.

데이터 연결 및 fetching작업이 서버에서 처리된다. 따라서 클라이언트 사이드 레넏링 코드가 필요하지 않아 자바스크립트 코드가 클라이언트에 전송할 필요가 없다.

핵심 원칙은 서버에서 렌더링하고 클라이언트에서 다시 하이드레이션하는데 필요한 자바스크립트를 함꼐 제공한다.

### ISR, On-demand ISR

| **항목**       | **ISR (Incremental Static Regeneration)**                               | **On-Demand ISR**                                                        |
| -------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 🔧 개념        | 정적 페이지를 빌드 후 일정 시간마다 백그라운드에서 재생성               | 특정 이벤트 발생 시 수동으로 정적 페이지를 재생성                        |
| 🕓 트리거 시점 | 사용자가 페이지 요청 후, revalidate 시간이 지나면 백그라운드에서 재생성 | API 요청 등으로 **명시적으로 트리거**할 때만 재생성                      |
| ⚙️ 설정 방법   | getStaticProps 내 revalidate 옵션 사용                                  | /api/revalidate와 같은 API 라우트를 만들어 사용                          |
| 📄 적용 대상   | 주기적으로 갱신되는 데이터 (뉴스, 블로그 등)                            | 수정 빈도 낮지만 갱신 타이밍을 정확히 제어해야 하는 페이지 (CMS 연동 등) |
| 🚀 장점        | 자동화된 주기적 갱신                                                    | 필요할 때만 재생성 → 낭비 없는 빌드                                      |
| 🧠 단점        | 정확한 갱신 타이밍 통제가 어려움                                        | 트리거 API 따로 구현해야 함, 인증 필요                                   |
| 🔒 보안        | 없음 (공개 접근)                                                        | API 경로에 보안 토큰 등을 사용해야 안전함                                |

### ISR

```tsx
// app/posts/[slug]/page.tsx

export const revalidate = 60; // 60초마다 ISR

export default async function PostPage({ params }) {
  const post = await getPost(params.slug);
  return <PostDetail post={post} />;
}
```

- 사용자가 페이지에 접근하면 정적 페이지가 보여짐
- 60초가 지나면, 다음 요청 시 **백그라운드에서 페이지를 새로 생성**
- 이후 요청부터는 새로 생성된 페이지가 사용됨

### On-Demand ISR

```tsx
// pages/api/revalidate.ts

export default async function handler(req, res) {
  const { secret, path } = req.query;

  if (secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: "Invalid token" });
  }

  try {
    await res.revalidate(path); // 예: /posts/my-article
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send("Error revalidating");
  }
}
```

```tsx
export const dynamic = "force-static"; // 정적 생성
export const revalidate = false; // 자동 ISR 비활성화 → 오직 수동 재생성만 허용

export default async function PostPage({ params }) {
  const post = await getPost(params.slug);
  return <PostDetail post={post} />;
}
```

- Next 문서 ⇒ https://nextjs.org/docs/14/app/building-your-application/caching#time-based-revalidation

### Streaming SSR

반면 **Streaming SSR**은 **HTML을 점진적으로 생성하며, 준비된 부분부터 클라이언트에 전송**합니다.

책에서는 16,17 버전의 api를 설명(`renderToNodeStream`)하기에 18버전으로 썼습니다.

Next는 Reac의 `renderToPipeableStream`를 사용합니다.

```tsx
// app/page.tsx
import { Suspense } from "react";
import Comments from "./Comments";

export default function Page() {
  return (
    <div>
      <h1>Post</h1>
      <Suspense fallback={<p>Loading comments...</p>}>
        <Comments />
      </Suspense>
    </div>
  );
}
```

```tsx
import express from "express";
import React from "react";
import { renderToPipeableStream } from "react-dom/server";
import App from "./App";

const app = express();

app.get("/", (req, res) => {
  const ABORT_DELAY = 5000; // fallback timeout

  let didError = false;

  const { pipe, abort } = renderToPipeableStream(<App />, {
    onShellReady() {
      // Initial HTML shell is ready to stream
      res.statusCode = didError ? 500 : 200;
      res.setHeader("Content-type", "text/html");
      pipe(res); // Start streaming
    },
    onError(err) {
      didError = true;
      console.error("Render error:", err);
    },
    onShellError(err) {
      res.statusCode = 500;
      res.send("<h1>Something went wrong</h1>");
    },
  });

  // Abort streaming if it takes too long
  setTimeout(abort, ABORT_DELAY);
});

app.listen(3000, () => console.log("Server running on http://localhost:3000"));
```

| **옵션**     | **설명**                                                           |
| ------------ | ------------------------------------------------------------------ |
| onShellReady | 최소한의 HTML Shell이 준비됐을 때 호출 (→ 브라우저에 즉시 전송)    |
| onAllReady   | 모든 컴포넌트 준비 완료 후 호출 (전체 렌더 완료 후 보내고 싶을 때) |
| onShellError | Shell 생성 중 에러 발생 시 호출                                    |
| onError      | 렌더 중 에러가 발생할 때마다 호출                                  |
| abort()      | 렌더링 중단 함수 (예: 타임아웃 처리 시)                            |

## 엣지 SSR

SSR을 전통적인 중앙 서버(Region Server)가 아닌, 사용자의 지리적으로 가까운 Edge 서버에서 수행하는 것

단점 (https://nextjs.org/docs/app/api-reference/edge)

- Next에서 엣지 SSR을 하기 위해서는 엣지 런타임(Vercel 등)을 활용해야 한다.
- Node.js API 사용 불가
- Edge Runtime은 모든 Node.js API를 지원하지 않습니다. 일부 패키지는 예상대로 작동하지 않을 수 있습니다.
- Edge Runtime은 ISR(증분적 정적 재생성)을 지원하지 않습니다.
- 두 런타임 모두 배포 어댑터에 따라 [스트리밍을 지원할 수 있습니다.](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming)

예시로 사용자별로 지역 특화 리스트 페이지를 구축한다 할 때, 리스트 컴포넌트만 서버 사이드에서 렌더링하고 나머지는 엣지 사이드에서 렌더링하도록 선택할 수 있다.

## Progressive Hydration(점진적 하이드레이션)

https://patterns-dev-kr.github.io/rendering-patterns/progressive-hydration/

여기 설명이 너무 잘되어있네요!

### 아일랜드 아키텍처

SSG의 진화버전⇒ 현재는 Astro가 이를 가장 본격적으로 적용하였다.

```tsx
[ 전체 페이지 (정적인 HTML) ]
 ├─ Header         ← static
 ├─ Article list   ← static
 ├─ ✨ LikeButton  ← "섬" → 클라이언트 JS로 하이드레이션됨
 ├─ ✨ CommentBox  ← "섬" → 클라이언트 JS로 하이드레이션됨
 └─ Footer         ← static
```

점진적 하이드레이션과 다른점은?

- 점진적 하이드레이션은 페이지의 **하이드레이션 구조가 top-down방식**
- 각 컴포넌트가 자체적으로 하이드레이션 스크립트를 가지고 있으며, 독립적으로 비동기 실행된다.
  - 즉 다른 컴포넌트에 영향을 주지 않아 성능이 좋음

장점

- 성능: 클라이언트에 전송되는 자바스크립트 코드의 양을 줄인다. 전체 요소를 하이드레이션에 필요한 스크립트보다 코드의 양이 적음
- SEO: 모든 정적 콘텐츠가 서버에서 렌더링되기 때문에 유리
- 중요 콘텐츠 우선순위: 블로그, 뉴스 기사 핵심 콘텐츠를 즉시 볼 수 있다.

단점:

- 아일랜드 아키텍처는 아직 초기 단계에 있음, 몇 안되는 프레임워크를 쓰거나 직접 개발해야한다..

🤔 그럼 Next는 island 아키텍처를 선택하지 않은 이유?

- https://www.reddit.com/r/nextjs/comments/17j8a30/nextjs_rsc_and_island_architeture/
- https://www.heise.de/hintergrund/How-Next-js-Empowers-Developers-Through-Its-Island-Architecture-9787650.html

선택하지 않은 것은 아니고 현재 RSC를 쓰면서 아일랜드 아키텍처의 핵심 개념들과 “거의 유사”하다고 한다.

필요한 부분만 클라이언트 컴포넌트로 만들면 서버 컴포넌트는 서버에서 렌더링되기 때문에 결과적으로 비슷한 효과를 내는 것

약간의 차이라면, 결국 리액트로 만들기 때문에 **하이드레이션을 하기 위한 js는 결국 오고가야 하는 것**입니다.

관련 예시) https://patterns-dev-kr.github.io/rendering-patterns/the-island-architecture/

### 리액트 서버 컴포넌트

제 블로그에,, 정리했었으니 링크로 대체해버리기 ㅋㅋㅋㅋ

https://velog.io/@haryan248/next-with-react-query
