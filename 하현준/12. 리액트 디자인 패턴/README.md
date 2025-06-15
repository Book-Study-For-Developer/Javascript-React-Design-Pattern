## 고차 컴포넌트

여러 컴포넌트에 동일한 로직이 있을 경우 사용할 수 있는 패턴

```tsx
function withLoader(Element, url) {
  return function WithLoader(props) {
    const [data, setData] = React.useState(null);

    React.useEffect(() => {
      async function fetchData() {
        const response = await fetch(url);
        const data = await response.json();
        setData(data);
      }

      fetchData();
    }, []);

    if (!data) {
      return <div>Loading...</div>;
    }

    return <Element {...props} data={data} />;
  };
}

const WrappedDogImages = withLoader(
  DogImages,
  "https://dog.ceo/api/breed/labrador/images/random/6"
);
```

- withLoader HOC를 사용하여 로딩이 필요한 컴포넌트에는 조합하여 사용할 수 있는 것이다.
  - 🤔 지금은 이를 Suspense 컴포넌트가 해결하죠?
- 고차 컴포넌트는 중첩해서 사용할 수 있습니다. 그렇게 되면 DogImages 컴포넌트는 **다른 역할을 몰라**도 됩니다.
  - 🤔 Next에서 SSR 사용할때는 중첩해서 사용하게 되면 더 불편했던 경험이 있습니다. HOC로 하게되면 결국 직렬 요청이 이뤄져 장점을 누리지 못하기 때문입니다.

장점

- 재사용 가능한 로직을 모아 관리할 수 있다.
- 코드를 DRY하게 유지하고 관심사를 분리할 수 있다.

단점

- 고차 컴포넌트가 대상 컴포넌트에 전달하는 prop의 이름은 충돌을 일으킬 수 있다.
  - 같은 props를 사용하여 전달받은 컴포넌트에 덮어씌울 수 있는 것

## 렌더링 Props 패턴

컴포넌트는 렌더링 prop이외에 어떠한 것도 렌더링하지 않는 것이다.

```tsx
<Title render={() => <h1>I am render prop!</h1>} />

<Title
 renderFirst={() => <h1>I am render prop1!</h1>}
 renderSecond={() => <h1>I am render prop1!</h1>}
/>
```

- 렌더링 prop을 받는 컴포넌트에서 가져온 데이터를 렌더링 prop으로 전달된 요소에 전달할 수 있다.
  - `FetchBoundary` 도 렌더링 Props 패턴
  - Suspensive 라이브러리도 렌더링 Props 패턴

⇒ 여기서 이제 상태 끌어올리기를 사용할 수 있다.

```tsx
// App component
function App() {
  return (
    <div className="App">
      <h1>Dog Images</h1>
      <DogImages
        render={(images, loading) => (
          <div>
            {loading ? (
              <div>Loading...</div>
            ) : (
              <div className="container">
                {images.map((image, index) => (
                  <img
                    src={image}
                    alt={`Dog ${index + 1}`}
                    key={index}
                    className="image"
                  />
                ))}
              </div>
            )}
          </div>
        )}
      />
    </div>
  );
}
```

DogImages 내부에 존재했던 상태(image, loading)을 부모 컴포넌트에서 바로 사용이 가능해진 것이다.

- 이는 코드를 읽을 때 가독성을 좋게 할 뿐더러, 상태를 관리할 때 복잡성을 낮춰준다.
- children을 통해서도 렌더링 Props 패턴 적용이 가능하다.

장점

- props를 명시적으로 전달함으로 써 고차 컴포넌트에서 발생했던 암시적인 props 문제 해결이 가능하다.

단점

- 렌더링 Prop에는 라이프사이클 관련 메서드(`useEffect`)를 추가할 수 없기에 받은 데이터를 변경할 필요가 없는 렌더링에만 사용이 가능하다.

## 리액트 Hooks 패턴

클래스 컴포넌트 시절의 특징

- 생성자 함수 내의 상태
- 컴포넌트와 라이프사이클에 따른 효과를 처리하기 위해 componentDidMount, componentWillDidUnmount와 같은 라이프사이클 메소드
- 추가적인 로직을 구현하기 위한 커스텀 메서드

### 구조 변경의 필요성

컴포넌트가 클수록 이러한 구조 변경이 더 까다로워질 뿐만 아니라, 깊게 중첩된 컴포넌트 간에 코드를 공유하기 위해 여러겹의 컴포넌트를 사용하면 “래퍼 헬”이 발생한다.

### 훅에 대해서

다들 useState, useEffect, 커스텀 훅은 아실테니

유용한 커스텀 훅 라이브러리 ⇒ https://github.com/toss/react-simplikit

장점

- 코드를 라이프사이클 별로 나누지 않고, 관심사 및 기능별로 그룹화가 가능하다.
- 전체의 코드 길이도 줄일 수 있게 해준다.

## 정적 가져오기

import 문을 통해서 모듈을 정적으로 가져올 수 있다.

각 모듈은 자바스크립트 엔진이 해당 모듈을 import하는 코드에 도달하는 즉시 실행된다.

컴포넌트를 정적으로 가져오게 되면 웹팩은 초기 번들에 포함시킨다.

- 초기 번들 용량이 커지면 첫 로딩시간이 길어질 수 있기 떄문에 번들 용량을 줄여야 한다.

## 동적 가져오기

Dynamic Import를 통해 컴포넌트를 가져오면 초기 번들에 포함되지 않아 번들 크기가 작아진다.

그렇기 때문에 초기 로딩 속도가 빨라져서 사용자 경험을 높일 수 있다.

🤔 Next에서 Streaming SSR을 활용할 때에도 API요청이 있는 부분을 Suspense로 감싸면 서버 환경에서 전체를 기다릴 필요없이 부분별로 Streaming을 하기 때문에 초기 로딩속도측면에서 더 향상 시킬 수 있다.

- https://react.dev/reference/react/lazy

```tsx
const MarkdownPreview = lazy(() => import("./MarkdownPreview.js"));
```

![image](https://github.com/user-attachments/assets/5bfceb33-7696-4c67-8438-80de15c7fc92)

### 로더블 컴포넌트

- 책에서는 Suspsense가 SSR 환경에서 지원하지 않을때여서 loadable-components 라이브러리를 사용한 것 같다.
- https://github.com/gregberge/loadable-components

즉, Suspsense와 동일한 기능을 하지만, SSR 환경에서 지원이 가능한 라이브러리인 것이다.

### 상호작용 시 가져오기

사용자의 상호작용을 통해 컴포넌트를 가져오는 방법

### 화면에 보이는 순간 가져오기

- 화면 하단에 위치한 컴포넌트의 경우에는 초기에 보여줄 필요가 없다.
- IntersectionObserver API를 사용하여 화면에 보일 때 가져오는 방식으로 구현이 가능하다.

```tsx
// hooks/useInView.ts
import { useEffect, useRef, useState } from "react";

export function useInView<T extends Element>(
  options?: IntersectionObserverInit
) {
  const ref = useRef<T | null>(null);
  const [isInView, setIsInView] = useState(false);

  useEffect(() => {
    if (!ref.current) return;

    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setIsInView(true);
        observer.disconnect(); // 최초 1회만 감지 후 종료
      }
    }, options);

    observer.observe(ref.current);

    return () => observer.disconnect();
  }, [ref.current]);

  return { ref, isInView };
}
```

```tsx
// components/LazyLoadWrapper.tsx
import React from "react";
import { useInView } from "../hooks/useInView";

type LazyLoadWrapperProps = {
  children: React.ReactNode;
  fallback?: React.ReactNode;
};

export const LazyLoadWrapper = ({
  children,
  fallback,
}: LazyLoadWrapperProps) => {
  const { ref, isInView } = useInView<HTMLDivElement>();

  return (
    <div ref={ref}>
      {isInView
        ? children
        : fallback ?? <div style={{ height: 200 }}>Loading...</div>}
    </div>
  );
};

// pages/index.tsx or App.tsx
import React from "react";
import { LazyLoadWrapper } from "./components/LazyLoadWrapper";

const HeavyComponent = () => {
  return (
    <div style={{ height: 300, background: "lightgreen" }}>
      🌟 Heavy Component Loaded!
    </div>
  );
};

export default function App() {
  return (
    <div>
      <div style={{ height: "100vh", background: "#ccc" }}>
        스크롤을 내려보세요
      </div>

      <LazyLoadWrapper>
        <HeavyComponent />
      </LazyLoadWrapper>

      <div style={{ height: "100vh" }} />
    </div>
  );
}
```

## 코드 스플리팅

### 경로 기반 분할, 번들 분할

- 라우팅에 필요한 리소스만 요청하여 동적으로 로드하는 방법
  - 다만, 이렇게 하게 될 경우 페이지 이동 시에 리소스 요청이 이뤄지기 때문에 오히려 시간이 더 걸릴 수있다.
  - Next에서도 Link태그를 쓸 경우 링크태그에 마우스가 올라가지면 다음 페이지의 리소스들을 미리 요청한다. (https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#2-prefetching)
- 하나의 큰 번들을 사용하는 대신 작은 번들 여러개로 나눠서 사용하는 것이 더 효과적
- 사용자 화면에 표시되는 시간(FCP), 가장 큰 컨텐츠가 화면에 렌더링 되는 시간(LCP), 사용자가 컨텐츠와 인터릭티프 할때까지 걸린 시간 (TTI)를 신경써야 한다.

🤔 당연한 말들만 너무 길게 써놔서,, 패스

## PRPL 패턴

- **푸시**: 중요한 리소스를 효율적으로 푸시하여 서버 왕복 횟수를 최소화하고 로딩시간을 단축합니다.
- **렌더링**: 사용자 경험을 개선하기 위해 초기 경로를 최대한 빠르게 렌더링합니다.
- **사전 캐싱**: 자주 방문하는 경로의 에셋을 백그라운드에서 미리 캐싱하여 서버 요청 횟수를 줄이고 더 나은 오프라인 경험을 제공합니다.
- **지연 로딩**: 자주 요청되지 않는 경로나 에셋은 지연로딩 합니다.

**HTTP/1.1 vs HTTP/2.0**

| **항목**                    | **HTTP/1.1**                                      | **HTTP/2.0**                                         |
| --------------------------- | ------------------------------------------------- | ---------------------------------------------------- |
| **전송 방식**               | 텍스트 기반 (ASCII)                               | 바이너리 기반                                        |
| **다중 요청 처리**          | 하나의 연결에서 **동시 요청 불가 (HOL blocking)** | 하나의 연결로 **동시 요청 처리 가능 (Multiplexing)** |
| **헤더 압축**               | 없음                                              | 있음 (HPACK 사용으로 헤더 크기 감소)                 |
| **서버 푸시 (Server Push)** | 지원하지 않음                                     | 지원 (서버가 클라이언트에 리소스 미리 전송 가능)     |
| **요청 우선순위**           | 없음                                              | 있음 (리소스 우선순위 지정 가능)                     |
| **연결 개수**               | 브라우저마다 도메인당 6개 정도로 제한             | 단일 연결로 충분 (연결 수 감소)                      |
| **속도 및 성능**            | 느림 (다중 연결 필요, 지연 높음)                  | 빠름 (헤더 압축, 다중화로 성능 향상)                 |
| **서포트 여부**             | 거의 모든 시스템에서 기본 지원                    | 최신 브라우저 및 서버에서 지원 (점점 보편화 중)      |
| **보안 연결 (HTTPS)**       | 선택사항 (HTTP 또는 HTTPS)                        | 대부분 구현에서 HTTPS 필수로 사용                    |

→ HTTP/2.0 사용하자

서버 푸시란?

Nodejs 환경에 클라이언트의 요청없이 미리 리소스를 요청하는 것이다.

```tsx
import http2 from "http2";
import fs from "fs";
import express from "express";

const app = express();

app.get("/", (req, res) => {
  const stream = res.stream;

  // 서버 푸시로 CSS 미리 전송
  stream.pushStream({ ":path": "/style.css" }, (err, pushStream) => {
    if (err) return;
    pushStream.respondWithFile("public/style.css", {
      "content-type": "text/css",
    });
  });

  res.sendFile("public/index.html");
});

http2
  .createSecureServer(
    {
      key: fs.readFileSync("./server.key"),
      cert: fs.readFileSync("./server.crt"),
    },
    app
  )
  .listen(8443);
```

- 서버는 요청하지 않은 리소스를 미리 받아와 성능 향상을 이루는 방법
- 다만 브라우저에서 한번 더 요청하여 중복으로 요청이 갈 수 있다.

## 로딩 우선순위

브라우저가 늦게 요청할 수도 있는 중요한 리소스를 더 일찍 요청할 수 있도로 `<link rel=”preload”>` 를 사용하는 것이다.

- 당연히 트레이드오프가 존재한다. 상호작용에 필요한 리소스가 preload로 인해 늦어질 수 도 있다.

### SPA의 preload

웹펙의 번들 옵션중에 주석을 통해 청크파일에 preload를 붙일 수 있도록 하는 옵션이 있다.

https://webpack.kr/guides/code-splitting/#prefetchingpreloading-modules

```tsx
import(/* webpackPrefetch: true */ "./path/to/LoginModal.js");
```

- 프리로드 청크는 부모 청크와 병렬로 로드를 시작합니다. 프리페치 청크는 부모 청크가 로드 완료된 후에 로드를 시작합니다.
- 프리로드 청크는 중간 우선순위를 가지며 즉시 다운로드됩니다. 프리페치 청크는 브라우저가 유휴 상태일 때 다운로드 됩니다.
- 프리로드 청크는 부모 청크에서 즉시 요청 되어야 합니다. 프리페치 청크는 나중에 언제라도 사용할 수 있습니다.
- 지원하는 브라우저에 차이가 있습니다.

### Chrome 95+ preload

원문 ⇒ https://www.patterns.dev/vanilla/preload/

- Putting it in HTTP headers will jump ahead of everything else
- Generally, preloads will load in the order the parser gets to them for anything >= Medium so be careful putting preloads at the beginning of the HTML.
- Font preloads are probably best towards the end of the head or beginning of the body
- Import preloads should be done after the script tag that needs the import (so the actual script gets loaded/parsed first)
- Image preloads will have a low priority and should be ordered relative to async scripts and other low/lowest priority tags

- HTTP 헤더에 넣으면 다른 모든 것보다 앞서 나갈 것입니다.
- 일반적으로 preload는 파서가 우선순위가 Medium 이상에 대해 받는 순서대로 로드되므로 HTML의 시작 부분에 preload를 넣는 데 주의하세요.
  - 따라서 **preload를 HTML 맨 앞에 두면 너무 일찍 로드**되어 **중요한 다른 리소스를 방해할 수 있으니 주의하세요.**
- 폰트 preload는 아마도 머리글의 끝이나 본문의 시작 부분에 가장 적합합니다.
- import preload는 가져오기가 필요한 스크립트 태그 뒤에 수행되어야 합니다(따라서 실제 스크립트가 먼저 로드/구문 분석됨)
- 이미지 preload는 낮은 우선순위를 가지며 비동기 스크립트 및 기타 낮은/가장 낮은 우선순위 태그와 관련하여 정렬되어야 합니다.

구글 크롬의 preload 에 대한 비용 ⇒ https://docs.google.com/document/d/1ZEi-XXhpajrnq8oqs5SiW-CXR3jMc20jWIzN5QRy1QA/edit?tab=t.0

### 🤔 구글 크롬의 preload 관련 내용 정리

<link rel="preload">는 **정교하게 사용하면 성능 개선 효과가 있지만**,

잘못 사용하면 오히려 **유지보수 비용과 성능 저하를 유발하는 **“발사기(footgun)”**가 될 수 있음. 특히 **Chrome의 버그로 인해 preload 리소스가 우선순위 큐를 점프**하는 문제 때문에 preload는 **매우 조심해서 써야 함\*\*.

**Preload가 이론적으로 유용한 경우**

1. **늦게 발견되는 중요한 리소스**
   - 예: CSS를 파싱한 후에야 알 수 있는 폰트, JS 실행 후에야 알 수 있는 이미지
2. **메인 스레드가 바빠서 HTML 파싱 지연**
   - 예: 초기 동기 JS가 CSS/JS 발견을 늦춤
3. **외부 리소스로 인해 초기 렌더링 지연**
   - 예: CSS/폰트가 외부에서 로딩되어 RTT(round trip)가 추가됨

**⚠️ 왜 문제가 되는가?**

**1.사용이 너무 어려움**

- preload는 **어떤 리소스가 중요한지 정확히 알고**,**그 순서를 완전히 수동으로 관리**해야 효과적임.
  - 예: LCP 이미지, CSS, 폰트 등을 모두 **preload 순서로 정리**해야 함

**2. 네트워크/메인스레드는 한정된 자원**

- preload한 리소스가 다른 중요한 리소스를 **가로막을 수 있음**

**3. 크롬의 preload 큐 점핑 버그**

- preload된 리소스가 **선행 리소스를 밀어내며 먼저 요청**됨
  → 결과적으로 **브라우저의 자연 로딩 순서를 깨뜨림**

**🧨 주의해야 할 preload 사례**

**❌ JS preload**

- 서버 렌더링 페이지에서 preload된 JS는 **CSS, 폰트, 이미지보다 먼저 다운로드됨**
- **대역폭을 차지**하고 **다른 리소스를 늦게 로드**하게 만듦

**❌ 폰트 preload**

- 폰트는 **용량이 크고 변형(variant)이 많아** bandwidth를 많이 소모함
- **렌더 블로킹 여부나 지연 로딩 여부를 정확히 알기 전까지는 preload 비추천**

**✅ 언제 preload가 유용한가?**

| **상황**   | **설명**                                                         |
| ---------- | ---------------------------------------------------------------- |
| 외부 CSS   | FCP 개선을 위해 preload 유용                                     |
| LCP 이미지 | UX 개선에는 유효하지만 LCP metric에는 아직 완벽히 반영 안됨      |
| 폰트       | 조건부로만 유용: CSS로부터 늦게 발견되고, 렌더블로킹일 때만 고려 |

**🛠️ 대체 전략**

- ✅ **critical CSS 인라인**
- ✅ **font CSS 인라인**
- ✅ **preconnect** 활용
- ✅ **LCP 이미지에 blur placeholder 사용** (UX 관점에서 도움)

**📌 결론**

- preload는 **매우 신중하게**, **명확한 목적과 정확한 순서 이해가 있는 경우에만 사용**
- 그렇지 않으면 **브라우저가 이미 잘 하고 있는 리소스 로딩 최적화 흐름을 깨뜨리는 꼴**
- 특히 크롬의 **preload 큐 점프 버그**로 인해 많은 경우 오히려 해가 될 수 있음

---

## 리스트 가상화

### 윈도잉/가상화의 작동 방식

- 상대적인 위치를 가진 작은 컨테이너 요소를 윈도우로 사용한다.
- 스크롤을 위한 큰 DOM 요소를 가진다.
- 컨테이너 내부에 자식 요소를 절대 위치로 배치시키고 top, left, width, height 등의 스타일을 설정
- 한번에 수천 개의 목록 요소를 렌더링하여 초기 렌더링 속도 저하 또는 스크롤 성능 저하를 유발하는 대신, 가상화는 사용자에게 보이는 항목만 렌더링 하는데 집중

⇒ 라이브러리들 종류: [react-window](https://www.npmjs.com/package/react-window), [react-virtualized](https://www.npmjs.com/package/react-virtualized), [tanstack-virtual](https://github.com/TanStack/virtual) (추천⭐️)

### 웹 플랫폼의 발전

css의 content-visibility 속성을 사용하면 화면 밖 컨텐츠의 렌더링과 페인팅에 필요한 시점까지 지연할 수 있다.

```tsx
.card {
  content-visibility: auto;
  contain-intrinsic-size: 500px; /* 필수는 아니지만 레이아웃 점프 방지용 */
}
```

- content-visibility: auto는 렌더링을 건너뛰기 때문에 실제 크기를 모름
- 대신 이 속성으로 **가상의 레이아웃 높이/너비**를 지정해 미리 공간을 확보함

🤔 이 속성을 통해 성능 최적화가 되는군요.. 처음알았습니다.

⇒ 관련 내용: https://web.dev/articles/content-visibility?hl=ko
