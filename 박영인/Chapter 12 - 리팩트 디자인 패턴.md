## 고차 컴포넌트
고차 컴포넌트 패턴은 여러 컴포넌트에서 동일한 로직을 재사용할 수 있게 해주는 패턴이다.
고차 컴포넌트는 다른 컴포넌트를 인자로 받아 해당 컴포넌트에 추가 기능을 적용하여 새로운 컴포넌트로 반환한다.

#### 예시
url fetch하고 데이터를 상태에 업데이트 하는 로직을 공통으로 사용할 수 있게 되었다.
```js
import React, { useEffect, useState } from "react";

export default function withLoader(Element, url) {
  return (props) => {
    const [data, setData] = useState(null);

    useEffect(() => {
      async function getData() {
        const res = await fetch(url);
        const data = await res.json();
        setData(data);
      }

      getData();
    }, []);

    if (!data) {
      return <div>Loading...</div>;
    }

    return <Element {...props} data={data} />;
  };
}
```

### 고차 컴포넌트 조합하기
여러 고차 컴포넌트를 조합하여 사용해서, 여러 기능을 붙일 수도 있다.

#### 예시
```js
export default withHover(
  withLoader(DogImages, "https://dog.ceo/api/breed/labrador/images/random/6")
);
```


### 사용하기 좋은 상황
- 여러 컴포넌트에 동일한 동작/기능을 적용해야할 때
- 추가된 커스텀 로직 없이도 컴포넌트가 독립적으로 작동할 수 있을 때
  ㄴ 기존 컴포넌트도 순수하게 유지하며, 외부에서 기능을 확장할 수 있다.


### 장점
코드를 DRY(Don’t Repeat Yourself)하게 유지하여, 버그 발생 위험을 줄일 수 있다.
효과적으로 관심사를 분리할 수 있다.

### 단점
고차 컴포넌트가 대상 컴포넌트에 props를 주입할 때, 이미 대상 컴포넌트가 동일한 이름의 prop을 가지고 있다면 충돌이 발생할 수 있다.

#### 예시
```js
// 이슈 발생: 내부 style값이 props.style에 덮어씌워진다.
function withStyles(Component) {
  return props => {
    const style = { padding: '0.2rem', margin: '1rem' };
    return <Component style={style} {...props} />
  }
}

const Button = () => <button style={{ color: 'red' }}>Click me!</button>
const StyledButton = withStyles(Button);


// 해결: prop을 병합하여 해결한다.
function withStyles(Component) {
  return props => {
    const style = {
      padding: '0.2rem',
      margin: '1rem',
      ...props.style
    };

    return <Component style={style} {...props} />
  }
}

```

여러 HOC를 함께 쓰면, 어떤 HOC가 어떤 props를 넘기는지 파악하기 어려워질 수 있다(Wrapper Hell). 이로 인해 디버깅이나 확장 작업이 복잡해질 수 있다.


## 렌더링 Props 패턴
렌더링에 사용할 함수를 props로 전달하는 패턴이다.
즉, 컴포넌트의 UI를 외부에서 정의할 수 있게 해주는 방식이다.

여기서 컴포넌트 자체는 렌더링 prop 외에 다른 UI는 렌더링 하지 않는다.
즉, 로직만 있고 UI는 위임하는 방식이다.

#### 예시
```js
const Title = (props) => (
  <>
    {props.renderFirstComponent()}
    {props.renderSecondComponent()}
    {props.renderThirdComponent()}
  </>
);

<Title
  renderFirstComponent={() => <h1>First render prop!</h1>}
  renderSecondComponent={() => <h2>Second render prop!</h2>}
  renderThirdComponent={() => <h3>Third render prop!</h3>}
/>
```


### 상태 끌어올리기
형제 컴포넌트 간에 상태가 공유되어야 하는 경우, 상태를 가장 가까운 공통 조상 컴포넌트로 끌어올리곤 한다.
이것이 상태 끌어올리기 이다.

하지만 이 경우 상태 변경 시 데이터를 사용하지 않는 자식 컴포넌트까지 모두 리렌더링될 수 있어, 성능에 악영향을 줄 수 있다.
👉 이를 렌더링 Props 패턴을 사용하여 해결할 수 있다.

```js
function Input(props) {
  const [value, setValue] = useState("");

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Temp in °C"
      />
      {props.render(value)}
    </>
  );
}

// Input 컴포넌트의 value 값을 App 컴포넌트 내에서 다룰 수 있게된다.
export default function App() {
  return (
    <div className="App">
      <h1>Temperature Converter</h1>
      <Input
        render={(value) => (
          <>
            <Kelvin value={value} />
            <Fahrenheit value={value} />
          </>
        )}
      />
    </div>
  );
}
```


### 컴포넌트의 자식으로 함수 전달하기
children prop를 통해서도 렌더링 Props 패턴을 사용할 수 있다.

### 장점
- 여러 컴포넌트 사이에서 로직과 데이터를 쉽게 공유할 수 있다.
- 컴포넌트의 재사용성을 높일 수 있다.
- 고차 컴포넌트에서 발생할 수 있는 이름 충돌 문제를 해결할 수 있다.

### 단점
- 대부분 hooks로 대체 가능하다.
- 렌더링 Prop에는 라이프사이클 관련 메서드를 추가할 수 없어, 데이터를 변경할 필요 없는 경우에만 사용할 수 있다.
  ㄴ 근데 이거 render prop을 컴포넌트로 따로 만들면 가능한거 아닌가..? 정확히 어떤건지 이해가 잘 안된다..


## 리액트 Hooks 패턴
React 16.8부터 도입된 Hooks는 기존의 **클래스 컴포넌트의 한계와 복잡성**을 해결하기 위해 등장했다.
### 클래스 컴포넌트의 문제점
#### 구조 변경의 필요성
클래스 컴포넌트 간에 코드를 공유하려면 고차 컴포넌트(HOC)나 렌더링 Props 패턴을 사용해야 했다.
하지만 이런 구조는 컴포넌트가 깊게 중첩되어 Wrapper Hell을 유발하였고, 
이로인해 데이터 흐름을 추적하기 어렵고 디버깅이 어려워졌다.

#### 복잡성 증가
클래스 컴포넌트에 로직이 많아질수록 컴포넌트 자체가 비대해지고 
내부 로직이 응집력 없이 흩어져 구조화되지 않는 문제가 생겼다.
이로인해 디버깅이 어려워지고, 라이프사이클 메서드 간 중복 코드가 발생했다.

### Hooks
클래스 컴포넌트의 일반적인 문제를 해결하기 위해 Hooks를 도입했다:

- 함수형 컴포넌트에서도 상태 추가 가능
- componentDidMount 같은 라이프사이클 제어 가능 (useEffect)
- 여러 컴포넌트 간 공통 로직(Custom Hook) 재사용 가능

### 장점
- 라이프사이클 메서드로 코드를 나누지 않고, 관심사/기능 단위로 코드를 구성할 수 있다.
  ㄴ 코드가 더 간결하고 깔끔해지며, 전체 코드 길이도 줄어든다.
- 상태 관련 로직을 단순한 자바스크립트 함수로 추출할 수 있게 되었으며, 이를 재사용할 수 있게 한다.
  ㄴ 이전에는 고차 컴포넌트, 렌터링 Props 같은 복잡한 방법을 써야했다.

### 단점
- 사용 규칙 준수가 필요하다.
  ㄴ Hook은 컴포넌트 최상단에서만 호출해야 하며, 순서가 보장되어야 한다.

## 정적 가져오기
import를 사용해서 모듈을 정적으로 가져올 수 있다.

정적으로 가져온 모듈은 초기 번들에 포함된다.
하지만 모든 컴포넌트를 처음부터 다 가져오면 초기 번들 크기가 커져 로딩이 지연될 수 있다.
👉 필요한 시점에 모듈을 가져오는 방법으로 개선 가능!

## 동적 가져오기
사용자와 상호작용이 발생하는 등 필요한 시점에 모듈을 동적으로 가져오는 방식을 사용하면,
해당 모듈은 초기 번들에서 제외된다.
👉 앱 로딩 속도가 개선된다.

리액트의 Suspense와 lazy를 통해서 구현 가능
```js
import React, { Suspense, lazy } from "react";

// 동적으로 불러올 컴포넌트를 lazy로 감싸기
const EmojiPicker = lazy(() => import("./EmojiPicker"));

function App() {
  const [open, setOpen] = useState(false);

  return (
    <Suspense fallback={<p>Loading...</p>}>
      <button onClick={() => setOpen(true)}>이모지 열기</button>
      {open && <EmojiPicker />}
    </Suspense>
  );
}
```
- EmojiPicker가 렌더 시도 될때 모듈을 불러온다.
- 컴포넌트가 로드되기 전까지 fallback UI를 보여준다.
### 로더블 컴포넌트
@loadable/component
SSR 환경에서 Suspense와 유사한 기능을 제공해주는 라이브러리이다.
근데.. 리액트 18버전 부터는 Suspense도 SSR 환경을 지원한다!!

### 상호작용 / 화면에 보이는 순간 가져오기
사용자가 클릭 같은 상호작용을 하거나, 스크롤 해서 화면에 컴포넌트가 나타났을 때 동적으로 모듈을 가져오곤 한다.

## 코드 스플리팅
여러 route와 컴포넌트로 구성된 복잡한 애플리케이션은 적절한 시기에 정적/동적 import가 가능하도록 코드를 최적으로 분할하고 번들링해야한다.

### 경로 기반 분할
특정 URL 경로에 도달할 때 필요한 컴포넌트를 **지연 로딩**하는 방식이다
사용자가 특정 페이지에 진입해야만 해당 페이지용 JS 번들이 네트워크 요청된다,
따라서 첫 진입 시 불필요한 번들을 제외하여 **초기 렌더링 성능 개선**할 수 있다.

```js
const App = lazy(() => import(/* webpackChunkName: "home" */ "./App"));
const Overview = lazy(() => import(/* webpackChunkName: "overview" */ "./Overview"));
const Settings = lazy(() => import(/* webpackChunkName: "settings" */ "./Settings"));

<Suspense fallback={<div>Loading...</div>}>
  <Switch>
    <Route exact path="/" component={App} />
    <Route path="/overview" component={Overview} />
    <Route path="/settings" component={Settings} />
  </Switch>
</Suspense>
```

### 번들 분할
 최신 웹앱에서는 Webpack 같은 번들러를 이용해 전체 소스 코드를 하나의 파일이 아닌 여러 개의 작은 번들로 나눈다. 
 
 이렇게 하면 사용자가 웹사이트를 방문할 때 실제로 필요한 기능에 해당하는 번들만 로드할 수 있어 초기 로딩 속도를 크게 줄일 수 있다.
성능 측면에서는 FCP(First Contentful Paint), LCP(Largest Contentful Paint), TTI(Time to Interactive) 같은 지표를 개선하기 위해 번들 크기를 줄이는 것이 효과적이다. 

👉 초기 로딩 시 꼭 필요한 코드만 메인 번들에 포함하고, 나머지 코드는 필요할 때 동적으로 불러오는 방식이 권장된다.

## PRPL 패턴
PRPL(Push Render Pre-cache Lazy-load) 패턴은 느린 네트워크나 저사양 기기에서도 웹 애플리케이션이 빠르고 안정적으로 동작하도록 돕는 **로딩 최적화 전략**이다.

PRPL은 다음 네 가지 핵심 원칙으로 구성된다:
1. **Push**: 중요한 리소스를 서버에서 먼저 클라이언트로 푸시(서버 푸시)하여 HTTP 요청 수를 줄이고 로딩 속도를 단축한다.
2. **Render**: 사용자 경험을 개선하기 위해 가장 먼저 보여줄 경로를 가능한 빨리 렌더링한다.
3. **Pre-cache**: 자주 방문하는 경로의 리소스를 백그라운드에서 미리 캐싱해 다음 접근 시 빠르게 사용할 수 있도록 한다.
4. **Lazy-load**: 자주 사용하지 않는 리소스는 필요할 때 지연 로드한다.

| **항목**           | **HTTP/1.1**                                                                 | **HTTP/2**                                                                           |
| ---------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **요청 방식**        | 하나의 TCP 연결에서는 **한 번에 하나의 요청만 처리**할 수 있음.여러 요청을 병렬로 처리하려면 TCP 연결을 여러 개 열어야 함. | 하나의 TCP 연결에서 **여러 요청을 동시에 처리(Multiplexing)** 가능. 각 요청은 스트림(Stream)으로 분리되어 독립적으로 처리됨. |
| **연결 수**         | 브라우저는 성능을 위해 같은 도메인에 대해 **최대 6개의 TCP 연결만 허용**. 그 이상은 대기 상태.                  | **1개의 TCP 연결만으로 수십~수백 개 요청 처리 가능**. 연결 수 병목 제거.                                      |
| **헤더 전송**        | HTTP 헤더는 **텍스트 기반**이며, 요청마다 **전체 헤더를 반복 전송**해야 함. 쿠키나 에이전트 정보 등이 중복됨.        | HTTP/2는 **헤더를 HPACK 알고리즘으로 압축**하여 중복 제거. 네트워크 대역폭 절약.                                |
| **서버 푸시**        | 클라이언트가 요청한 리소스만 응답함. 예: /index.html 요청 시, CSS/JS는 클라이언트가 별도 요청해야 함.          | 서버가 클라이언트 요청 전에 필요한 리소스를 **미리 전송 가능** (<link rel="preload">과 함께 사용 가능).              |
| **HOL Blocking** | 하나의 연결에서 앞 요청이 끝나야 다음 요청 가능. → 앞 요청이 느리면 뒷 요청도 지연됨.                          | 요청마다 스트림 ID가 있어 병렬 처리됨. 앞 요청 지연이 **다른 요청에 영향 없음** (애플리케이션 레벨에서 해결됨).                 |
| **프로토콜 형식**      | 요청/응답이 **텍스트 기반**(예: GET /index.html). 디버깅은 쉬우나 성능은 낮음.                      | **바이너리 프레임 기반**. 더 빠르고 효율적인 데이터 전송 및 처리 가능.                                          |
| **보안 (TLS)**     | HTTPS는 선택사항. HTTP/1.1은 **암호화 없이도 동작** 가능.                                    | TLS가 강력히 권장됨. HTTP/3부터는 **암호화(TLS) 필수**.                                             |

기존 HTTP/1.1은 한 번에 여러 요청을 처리하지 못해 요청이 지연되는 **HOL(Head-of-Line) Blocking** 문제가 있었다. 
반면, **HTTP/2**는 하나의 TCP 연결로 **동시 다발적인 요청/응답 전송이 가능한 스트리밍 구조**를 제공하며, 이를 통해 PRPL의 핵심 전략인 Server Push, Preload 등이 효과적으로 동작할 수 있다.


## 로딩 우선순위
웹 성능을 높이기 위해, 초기에 꼭 필요한 리소스를 먼저 불러오는 것이 좋다.
`<link rel="preload">` 를 사용하여 이런 기능을 구현할 수 있다.

`<link rel="preload">` 는 **브라우저가 중요한 리소스를 먼저 요청하도록 지시**하는 태그이다.
브라우저가 늦게 요청할 수도 있는 중요한 리소스를 **미리 요청**하게 한다.

하지만 무분별한 preload는 다른 주요 리소스의 로드를 지연시켜 오히려 FCP, LCP를 지연시킬 수 있으니 우선순위를 잘 고려해야한다.
👉 트레이드 오프가 있으니 유의해야한다!
 
## 리스트 가상화
리스트 가상화는 대규모 데이터 리스트의 렌더링 성능을 향상시켜주는 기술이다.
전체 목록을 모두 렌더링하는 대신 현재 화면에 보이는 부분만 동적으로 렌더링 한다.

react-virtualized, react-window 같은 라이브러리를 써서 구현 가능하다.

### 웹 플랫폼의 발전
최신 브라우저는 css content-visivility 속성을 지원한다.
`content-visivility: auto` 로 설정하면 화면 밖 콘텐츠의 렌더링과 페인팅을 지연시킬 수 있다.

