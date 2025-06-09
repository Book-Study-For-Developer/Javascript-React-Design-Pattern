# 리액트 디자인 패턴

## 12.2 고차 컴포넌트
- 다른 컴포넌트를 인자로 받아 새로운 컴포넌트를 반환하는 컴포넌트
- 여러 컴포넌트에서 동일한 로직을 재사용하는 방법
- 필요한 로직을 한 곳에 유지하면서 동시에 여러 컴포넌트에 동일한 로직 제공 가능

### 사용 예시
- 로그인한 사용자만 볼 수 있는 컴포넌트
```js
import React from 'react';

const withAuth = (WrappedComponent) => {
  return function AuthComponent(props) {
    const isLoggedIn = localStorage.getItem('token'); // 예시용
    if (!isLoggedIn) {
      return <div>로그인이 필요합니다.</div>;
    }
    return <WrappedComponent {...props} />;
  };
};

export default withAuth;
```
```js
function MyPage() {
  return <div>마이페이지 콘텐츠</div>;
}

import withAuth from './withAuth';
export default withAuth(MyPage);
```

- 로딩 스피너 처리 HOC
```js
import React from 'react';

const withLoading = (WrappedComponent) => {
  return function LoadingComponent({ isLoading, ...rest }) {
    if (isLoading) {
      return <div>로딩 중...</div>;
    }
    return <WrappedComponent {...rest} />;
  };
};

export default withLoading;
```
```js
function UserList({ users }: { users: string[] }) {
  return <ul>{users.map((u) => <li key={u}>{u}</li>)}</ul>;
}

import withLoading from './withLoading';
export default withLoading(UserList);
```

### 12.2.3 단점
- 고차 컴포넌트가 대상 컴포넌트에 전달하는 prop의 이름은 충돌을 일으킬 수 있음 -> prop의 이름을 변경하거나 병합하는 방식을 사용해야 함
- 여러 고차 컴포넌트를 조합하여 사용하면 어떤 고차 컴포넌트가 어떤 prop을 제공하는지 파악하기 어려울 수 있음

<br />

## 12.3 렌더링 Props 패턴
- 렌더링 prop은 JSX 요소를 반환하는 함수 값을 가지는 컴포넌트의 prop이고, 컴포넌트는 렌더링 prop외에는 아무것도 렌더링하지 않음
- 한 컴포넌트를 여러 번 재사용하면서 매번 다른 값을 렌더링 prop에 전달할 수 있음
- 공통 로직을 재사용하면서도 UI는 호출한 쪽에서 자유롭게 정의하도록 할 수 있음
- 로직과 UI를 분리하여 재사용성 높일 수 있음

### 사용 예시
- 버튼 클릭 수 세기
```ts
// 로직 담당
type CountProps = {
  render: (count: number, increment: () => void) => React.ReactNode;
};

function Counter({ render }: CountProps) {
  const [count, setCount] = React.useState(0);
  const increment = () => setCount((c) => c + 1);

  return <>{render(count, increment)}</>;
}
```
```ts
// UI 담당
function App() {
  return (
    <Counter
      render={(count, increment) => (
        <div>
          <p>현재 카운트: {count}</p>
          <button onClick={increment}>+1</button>
        </div>
      )}
    />
  );
}
```

<br />

### 12.3.1 상태 끌어올리기
- 입력 컴포넌트가 자신의 상태를 다른 컴포넌트와 공유하려면, 상태를 필요로 하는 컴포넌트와 가장 가까운 공통 조상 컴포넌트로 끌어올려야 함
- 많은 자식 컴포넌트를 처리하는 큰 규모의 애플리케이션에서는 상태 변경 시 데이터를 사용하지 않는 자식 컴포넌트까지 모두 리렌더링될 수 있음
- 렌더링 Props 패턴을 사용하여 문제 해결 가능

```js
function Input(props) {
  const [value, setValue] = usestate('');

  return (
    <>
      <input
        type='text'
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder='Temp in C'
      />
      {props.render(value)}
    </>
  )
}

export default function App() {
  return (
    <div className='App'>
      <h1>Temperature Converter</h1>
      <Input
        render={(value) => (
          <div>
            <Kelvin value={value} />
            <Fahrenheit value={value} />
          </>
        )}
      />
    </div>
  )
}
```

<br />

### 12.3.2 컴포넌트의 자식으로 함수 전달하기
- 렌더링 prop을 명시적으로 전달하는 대신, Input 컴포넌트의 자식으로 함수 전달하기
- 렌더링 prop의 이름에 구애받지 않을 수 있음

```js
function Input(props) {
  const [value, setValue] = useState('');

  return (
    <>
      <input 
        type='text'
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder='Temp in C'
      />
      {props.children(value)}
    </>
  )
}

export default function App() {
  return (
    <div className='App'>
      <h1>Temperature Converter</h1>
      <Input>
        {(value) => (
          <>
            <Kelvin value={value} />
            <Fahrenheit value={value} />
          </>
        )}
      </Input>
    </div>
  )
}
```

<br />

### 12.3.3 장점
- 고차 컴포넌트 패턴에서 발생할 수 있는 이름 충돌 문제 해결
- props를 자동으로 병합하지 않고, 부모 컴포넌트에서 제공하는 값 그대로 자식 컴포넌트로 전달
- 요소에 전달해야 할 props는 모두 렌더링 prop의 인자 목록에 명확하게 나타남 -> 어떤 props가 어디에서 오는지 정확하게 파악 가능
- 렌더링 prop을 통해 애플리케이션 로직과 렌더링 컴포넌트 분리 가능

<br />

## 12.4 리액트 Hooks 패턴
Hooks를 사용하면 클래스 컴포넌트를 사용하지 않고도 상태와 라이프사이클 메서드를 활용할 수 있음

<br />

### 12.4.2 구조 변경의 필요성
- 컴포넌트가 클수록 구조 변경이 까다로워지고, 깊게 중첩된 컴포넌트 간 코드를 공유하기 위해서는 레퍼 헬에 빠질 수도 있음
- 레퍼 헬은 애플리케이션 내 데이터 흐름을 파악하기 어렵게 만듦

<br />

### 12.5.3 Hook 관련 추가 정보
- useState: 상태 업데이트와 조작 가능
- useEffect: 함수형 컴포넌트의 주요 라이프사이클 이벤트 중간에 코드 실행 가능, 부수 효과 방지
- useContext: React.createContext로 만들 수 있는 context 객체를 인자로 받아 해당 context의 현재 상태에 접근할 수 있게 함
- useReducer: setState의 대안, 여러 깊은 트리를 가진 복잡한 상태 로직이나 변경 이후의 상태가 이전 상태에 따라 달라지는 경우 유용

<br />

### 12.5.4 Hook의 장점
- 코드를 라이프사이클별로 나누지 않고, 관심사 및 기능별로 그룹화 가능
- 복잡한 컴포넌트 단순화
- 상태 관련 로직 재사용 가능
- UI에서 분리된 로직 공유 가능

<br />

### 12.5.5 Hook vs class
- Hook은 복잡한 계층 구조를 피하고 코드를 명확하게 만듦
- 클래스에서 고차 컴포넌트나 렌더링 Props를 사용하면 여러 계층에 걸쳐 애플리케이션 구조를 파악해야 함
- Hook은 리액트 컴포넌트 전체에 일관성 부여함

<br />

## 12.6 정적 가져오기
- 각 모듈은 자바스크립트 엔진이 해당 모듈을 import하는 코드에 도달하는 즉시 실행됨
- 웹팩은 모듈을을 초기 번들에 포함시킴
- 컴포넌트가 사용자 화면에 내용을 렌더링하려면 먼저 모든 모듈을 로드하고 파싱해야 함

<br />

## 12.7 동적 가져오기
- 초기 페이지 로드 시에 사용되지 않는 컴포넌트를 동적으로 가져올 수 있음
- 리액트에서 제공하는 Suspense 컴포넌트는 동적으로 로드되어야 할 컴포넌트를 감쌈
- 모듈의 가져오기를 일시적으로 중단시킴으로써 컴포넌트가 더 빠르게 내용을 렌더링하도록 함
- Suspense 컴포넌트는 일시 중단된 컴포넌트가 로딩되는 동안에 대신 렌더링할 컴포넌트를 받는 fallback prop 사용

<br />

### 12.7.1 로더블 컴포넌트
SSR 환경은 Suspense를 지원하지 않았지만, 리액트 18부터는 Suspense를 사용할 수 있게 됨

<br />

### 12.7.2 상호작용 시 가져오기
- 사용자의 상호작용을 통해 컴포넌트 가져오기 실행
- 예시: 버튼 클릭, 드롭다운 오픈

<br />

### 12.7.3 화면에 보이는 순간 가져오기
- 컴포넌트가 사용자에게 보일 때 동적 가져오기를 실행하는 것
- IntersectionObserver API, react-loadable-visibility, react-lazyload를 사용하여 컴포넌트가 현재 화면에 표시되는지 확인 가능
  - IntersectionObserver API: 브라우저 내장 API로, DOM 요소가 뷰포트 안에 들어왔는지 감지
  - react-lazyload: 스크롤 위치에 따라 컴포넌트가 화면에 들어오면 렌더링하는 React 전용 Lazy Loading 컴포넌트
  - react-loadable-visibility: react-loadable을 확장한 라이브러리로, 컴포넌트가 화면에 보일 때만 비동기 로딩을 트리거

  | 항목             | IntersectionObserver API | react-lazyload      | react-loadable-visibility            |
  | -------------- | ------------------------ | ------------------- | ------------------------------------ |
  | 구현 방식          | 브라우저 내장 API 사용           | 스크롤/위치 기반           | IntersectionObserver 사용 + 비동기 import |
  | 비동기 컴포넌트 로딩 지원 | ❌ 직접 구현해야 함              | ❌ 렌더링만 제어           | ✅ 화면 보일 때만 `import()`                |
  | 제어 범위          | 매우 유연함 (직접 제어 가능)        | 간단한 설정 위주           | 중간 수준, 간편하게 쓰기 좋음                    |
  | 실제 사용 예        | 직접 구현할 때                 | 간단한 LazyRender 필요 시 | SSR 고려된 고급 Lazy Loading 시            |
  | SSR 지원 여부      | 직접 구현해야 가능               | ❌ SSR 대응 어려움        | ✅ SSR도 지원 가능                         |

<br />

## 12.8 코드 스플리팅
여러 경로와 컴포넌트로 구성된 복잡한 애플리케이션에서는 적절한 시기에 정적 및 동적 임포트가 가능하도록 코드를 최적으로 스플리팅하고 분할해야 함

### 12.8.1 경로 기반 분할
- 특정 경로에 필요한 리소스 요청 가능
- Suspense, loadable-components나 react-router같은 라이브러리 함께 사용하면 현재 경로에 따라 컴포넌트 동적 로드 가능

```js
const App = lazy(() => import(/* webpackChunkName: 'home' */ './App'));
const Overview = lazy(() => import(/* webpackChunkName: 'overview' */ './Overview'));
const Settings = lazy(() => import(/* webpackChunkName: 'settings' */ './Settings'));

render(
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path='/'>
          <App />
        </Route>
        <Route path='/overview'>
          <Overview />
        </Route>
        <Route path='/settings'>
          <Settings />
        </Route>
      </Switch>
    </Suspense>
  </Router>,
  document.getElementById('root');
)
```

<br />

### 12.8.2 번들 분할
- 불필요한 코드를 포함하는 거대한 번들 하나를 요청하는 대신, 번들을 여러 개의 작은 번들로 분할하는 방법을 사용할 수 있음
- 번들 크기를 결정할 때 고려해야 할 지표
  - FCP: 첫 번째 콘텐츠가 사용자 화면에 표시되는 시간
  - LCP: 가장 큰 콘텐츠가 화면에 렌더링되는 시간
  - TTI: 모든 콘텐츠가 화면에 표시되고 인터랙티브해지는 데 걸리는 시간

<br /> 

## 12.9 PRPL 패턴
- 푸시: 중요한 리소스를 효율적으로 푸시하여 서버 왕복 횟수를 최소화하고 로딩 시간 단축
- 렌더링: 사용자 경험을 개선하기 위해 초기 경로를 최대한 빠르게 렌더링
- 사전 캐싱: 자주 방문하는 경로의 에셋을 백그라운드에서 미리 캐싱하여 서버 요청 횟수를 줄임
- 지연 로딩: 자주 요청되지 않는 경로나 에셋은 지연 로딩

<br /> 

### HTTP/1.1
- 요청과 응답에 줄바꿈 문자로 구분되는 일반 텍스트 프로토콜 사용
- 클라이언트와 서버 간 최대 6개의 TCP 연결만 가능
- 동일한 TCP 연결을 통해 새로운 요청을 보내려면 이전 요청이 완료되어야 했음 -> HOL Blocking

<br /> 

### HTTP/2
- 요청과 응답을 더 작은 프레임으로 분할
- 양방향 스트림 사용 -> 단일 TCP 연결을 통해 여러 개의 양방향 스트림을 만들어 클라이언트와 서버 간에 여러 개의 요청 및 응답 프레임을 동시에 전달할 수 있음
- 이전에 보낸 요청이 완료되기 전 동일한 TCP 연결을 통해 여러 요청을 보낼 수 있게 함으로써 HOL Blocking 문제 해결
- 서버가 리소스를 푸시하여 자동으로 추가 리소스 전송 가능

<br /> 

### PRPL 패턴 적용
- 요청하는 번들이 해당 시점에 필요한 최소한의 리소스만 포함하고, 브라우저에서 리소스를 캐싱할 수 있도록 해야 함
- 주로 애플리케이션의 셸을 주요 진입점으로 사용, 애플리케이션 셸은 애플리케이션 로직의 대부분을 포함하며 여러 경로에서 공유되는 최소한의 파일
- PRPL 패턴은 초기 접근 경로의 화면이 사용자 기기에 표시되기 전에 다른 리소스가 요청되거나 렌더링되지 않도록 보장

<br />

## 12.10 로딩 우선순위
- 필요하다고 예상되는 특정 리소스를 우선적으로 요청하도록 설정
- Preload는 브라우저가 늦게 요청할 수도 있는 중요한 리소스를 더 일찍 요청할 수 있도록 함
- preload는 TTI 또는 FID같은 지표 최적화 시 유리하지만, FCP 또는 LCP에 필요한 리소스 로딩 지연을 조심해야 함
- 자바스크립트 자체의 로딩을 최적화하려면 defer를 사용하는 것이 도움이 될 수 있음

<br />

### 12.10.1 SPA의 Preload
- 웹팩의 번들링 과정에 영향을 줄 수 있는 특수한 주석을 추가해 모듈을 미리 로드해야 함을 알릴 수 있음
- 사용자의 기기 및 인터넷 연결 환경에 따라 초기 로딩 시간이 크게 증가할 수 있기 때문에, 초기 렌더링 후 약 1초 이내에 표시되어야 하는 리소스만 선별하여 로드하는 것이 좋음

```js
const EmojiPicker = import(/* webpackPreload: true */ './EmojiPicker');
```
- 미리 로드된 EmojiPicker는 초기 번들과 병렬로 로드됨
- 브라우저가 인터넷 연결 상태와 대역폭을 고려하여 어떤 리소스를 미리 가져올지 결정하는 prefetch와 달리, Preload된 리소스는 어떤 상황에서든 미리 로드됨

<br />

### 12.10.2 Preload + async 기법
- 브라우저가 스크립트를 높은 우선순위로 다운로드하면서도, 스크립트를 기다리는 동안 파싱이 멈추지 않게 할 수 있음

| 장점                | 설명                                                             |
| ----------------- | -------------------------------------------------------------- |
| ⚡ 빠른 다운로드         | `preload`로 먼저 다운로드하므로 `script` 태그에서 시작하는 것보다 **빠르게 리소스 확보** 가능 |
| 🚀 HTML 파싱 블로킹 없음 | `async` 덕분에 **HTML 파싱을 막지 않음**, 초기 렌더링이 지연되지 않음                |
| ⏱ 실행 타이밍 제어 가능    | 다운로드는 `preload`, 실행은 `async`로 나누므로 **다운로드-실행 시점 분리** 가능        |
| 📈 LCP 개선에 도움     | 중요한 스크립트를 더 빨리 불러와서 중요한 요소가 더 빨리 보이게 함 (예: 폴드 위 인터랙션)          |

| 단점            | 설명                                                       |
| ------------- | -------------------------------------------------------- |
| 😵 의존성 관리 어려움 | `async`는 **실행 순서를 보장하지 않음**, 의존성이 있는 스크립트 간 충돌 위험 있음     |
| 🧠 브라우저 지원 고려 | 오래된 브라우저는 `preload`를 제대로 지원하지 않음 (IE 등)                  |
| 👀 코드 복잡도 증가  | preload + async 조합은 **간단한 상황보다 복잡한 시나리오에 적합**, 관리 복잡도 증가 |
| 🧪 캐싱 문제 주의   | preload로 받은 스크립트가 실제 실행 안 되면 낭비가 될 수 있음 (예: 조건부 로딩 실패)   |

<br />

## 12.11 리스트 가상화
- 대규모 데이터 리스트의 렌더링 성능 향상시키는 기술
- 전체 목록을 모두 렌더링하는 대신 현재 화면에 보이는 행만 동적으로 렌더링

<br />

### 12.11.1 윈도잉/가상화의 작동 방식
- react-virtualized에서 윈도잉은 아래와 같은 방식으로 작동
- 상대적인 위치를 가진 작은 컨테이너 DOM 요소를 윈도우로 사용
- 스크롤을 위한 큰 DOM 요소 가짐
- 컨테이너 내부에 자식 요소를 절대 위치로 배치하고, top, left, width, height 등의 스타일 설정
- 한 번에 수천 개의 목록 요소를 렌더링하여 초기 렌더링 속도 저하 또는 스크롤 성능 저하를 유발하는 대신, 가상화는 사용자에게 보이는 항목만 렌더링하는 데 집중함

<br />

### 12.11.2 react-window의 List 컴포넌트
- 윈도잉된 리스트의 아이템 렌더링 (화면에 보이는 행)
- 내부적으로 Grid 컴포넌트 사용하여 행을 렌더링하고, prop 전달

<br />

### 12.11.3 Grid 컴포넌트
- 수직 및 수평 방향으로 가상화된 표 형태의 데이터 렌더링
- 수평/수직의 현재 스크롤 위치에 따라 필요한 Grid 셀만 렌더링

<br />

### 가상화 라이브러리 비교
| 라이브러리                                       | 주요 특징                          | 사용 난이도 | 장점                                   | 단점                         | 적합한 상황                                           |
| ------------------------------------------- | ------------------------------ | ------ | ------------------------------------ | -------------------------- | ------------------------------------------------ |
| **react-window**                            | 리스트/그리드 가상화 전용 경량 라이브러리        | ★☆☆    | 가볍고 빠름<br>API가 간단함                   | 커스터마이징이 제한적                | 고정된 높이/너비의 단순 리스트나 그리드, 빠른 성능이 필요할 때             |
| **react-virtualized**                       | 다양한 유형 지원 (리스트, 그리드, 테이블 등)    | ★★★    | 고기능, 다양한 케이스 대응                      | 덩치 큼 (번들 사이즈 ↑)<br>러닝커브 있음 | 테이블, AutoSizer, Cell 병합 등 고급 UI 구현이 필요한 대규모 프로젝트 |
| **react-virtuoso**                          | 자동 높이 조절, 무한 스크롤 등 고급 기능 기본 제공 | ★★☆    | 빠른 성능<br>높은 추상화<br>무한 스크롤 지원         | 비교적 새로운 라이브러리              | 아이템 높이가 유동적이거나, 빠르게 고급 리스트 기능을 붙이고 싶을 때          |
| **TanStack Virtual**<br>(ex. react-virtual) | TanStack 제작진이 만든 훅 기반 라이브러리    | ★★☆    | 유연함, 커스터마이징 용이<br>기존 TanStack과 연동 용이 | UI 직접 구성 필요<br>러닝커브 있음     | 사용자 정의가 많은 UI, 테이블, 그리드, 커스텀 리스트 구현이 필요할 때       |

| 상황                   | 추천 라이브러리            |
| -------------------- | ------------------- |
| 단순 리스트               | `react-window`      |
| 다양한 UI (그리드, 테이블 등)  | `react-virtualized` |
| 빠르게 기능 붙이고 싶을 때      | `react-virtuoso`    |
| 완전한 커스터마이징, Hooks 선호 | `TanStack Virtual`  |

<br />

### 12.11.4 웹 플랫폼의 발전
- content-visibility:auto 설정하면 화면 밖 콘텐츠의 렌더링과 페인팅을 필요한 시점까지 지연시킬 수 있음

