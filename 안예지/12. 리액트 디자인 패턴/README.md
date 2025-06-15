## 리액트의 기본 개념 

### 1. JSX (JavaScript XML) <a name="jsx"></a>

#### JSX란?
- JSX는 JavaScript 코드 안에서 HTML과 유사한 문법을 작성할 수 있게 해주는 문법 확장입니다. 선언형 UI를 작성할 수 있어 가독성과 유지보수성이 크게 향상됩니다.
- React에서 JSX는 UI 구조를 명확하게 표현할 수 있는 수단입니다. 실제로는 Babel이 JSX를 React.createElement로 변환하여 가상 DOM 트리를 생성합니다.

#### 규칙
- 모든 태그는 반드시 닫아야 합니다. (`<br />`, `<img />` 등)
- 속성은 camelCase로 작성해야 합니다. (`className`, `onClick` 등)
- JavaScript 표현식은 중괄호 `{}`로 감쌉니다.
- 반드시 하나의 루트 요소가 필요합니다. 여러 요소를 반환하려면 `<></>`(Fragment)를 사용합니다.

#### 예시
```jsx
const element = <h1 className="title">Hello, JSX!</h1>;
// 변환 후
const element = React.createElement('h1', { className: 'title' }, 'Hello, JSX!');
```

#### 특징
- JSX 내부에서는 삼항 연산자, 논리 연산자(`&&`) 등을 활용하여 조건부 렌더링을 구현할 수 있습니다.
- JSX는 JavaScript의 모든 기능을 사용할 수 있으므로, 동적 렌더링, 반복 렌더링(map), 중첩 구조 등 다양한 UI 패턴을 쉽게 구현할 수 있습니다.
- Babel이 JSX를 어떻게 변환하는지 이해하면, 렌더링 최적화나 디버깅에 도움이 됩니다. 실제로 Babel playground([바로가기](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=Q&forceAllTransforms=false&modules=false&shippedProposals=false&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.27.4&externalPlugins=&assumptions=%7B%7D))에서 JSX 코드를 입력하면, 실제로 어떤 JavaScript 코드로 변환되는지 바로 확인할 수 있습니다.

예를 들어,
```tsx
const element = <h1 className="title">Hello, JSX!</h1>;
```
와 같이 입력하면, Babel은 다음과 같이 변환합니다.
```js
import { jsx as _jsx } from "react/jsx-runtime";
const element = /*#__PURE__*/_jsx("h1", {
  className: "title",
  children: "Hello, JSX!"
});
```
이 과정은 다음과 같은 단계로 이루어집니다.
- **JSX 파싱**: Babel의 @babel/parser가 JSX를 AST(Abstract Syntax Tree)로 파싱합니다.
- **함수 호출 치환**: JSX 엘리먼트는 내부적으로 jsx(type, props) 또는 jsxs() 함수 호출로 변환되며, 이는 react/jsx-runtime에서 가져온 함수로 기존의 React.createElement를 대체합니다.
- **속성 처리**: JSX의 속성(className, children 등)은 객체 리터럴로 변환되어 함수의 인자로 전달됩니다.
- **주석 및 최적화**: `/*#__PURE__*/` 주석은 트리쉐이킹(tree-shaking)을 돕기 위한 힌트로, 번들러에게 해당 표현식이 부작용이 없음을 알려줍니다.



### 2. Component

#### 개념
- 컴포넌트는 UI의 최소 단위이며, 독립적이고 재사용 가능한 구조입니다. React 애플리케이션은 컴포넌트의 트리 구조로 전체 UI를 구성합니다.
- 컴포넌트는 입력(props)과 내부 상태(state)를 받아 UI를 출력하는 순수 함수적 개념에 가깝습니다.

#### 종류
- 함수형 컴포넌트가 React 16.8 이후 표준입니다. Hooks를 통해 상태와 생명주기 기능을 사용할 수 있습니다.

#### 예시
```jsx
function Welcome({ name }) {
  return <h1>안녕하세요, {name}님!</h1>;
}
```


#### 특징
- 컴포넌트 합성을 통해 여러 컴포넌트를 조합하여 복잡한 UI를 구현할 수 있습니다. 예를 들어, 레이아웃 컴포넌트와 컨텐츠 컴포넌트를 분리하여 재사용성을 높입니다.
- 컴포넌트의 불변성을 유지하는 것이 중요합니다. props나 state를 직접 변경하지 않고, 항상 새로운 객체로 갱신해야 예측 가능한 동작을 보장할 수 있습니다.
- 컴포넌트의 책임을 명확히 분리하고, 단일 책임 원칙(SRP)을 지키는 것이 유지보수에 유리합니다.
- Storybook, Figma 등 디자인 시스템 도구와 연동하여 컴포넌트 기반 UI 시스템을 구축하면 협업과 유지보수가 쉬워집니다.
- Atomic Design 패턴을 적용하면, atoms → molecules → organisms → templates → pages 순으로 UI를 체계적으로 설계할 수 있습니다.



### 3. Props

#### 개념
- Props는 부모 컴포넌트가 자식 컴포넌트에게 전달하는 읽기 전용 데이터입니다. React는 단방향 데이터 흐름을 따르기 때문에, 데이터는 항상 부모에서 자식으로만 전달됩니다.
- Props는 컴포넌트의 재사용성과 확장성을 높이는 핵심 수단입니다.

#### 예시
```jsx
function Greeting({ message }) {
  return <p>{message}</p>;
}
```

#### 특징
- PropTypes 또는 TypeScript를 활용하여 props의 타입을 명확히 지정하면, 런타임 오류를 예방할 수 있습니다.
- 함수형 컴포넌트에서는 매개변수 기본값을 활용해 defaultProps를 대체할 수 있습니다.
- props로 함수를 전달할 때는 useCallback 등으로 메모이제이션하여 불필요한 렌더링을 방지할 수 있습니다.
- props drilling(깊은 전달)이 심해지면 Context API나 전역 상태 관리 도구의 도입을 고려해야 합니다.
- 자식 컴포넌트가 부모의 상태를 변경해야 할 때는, 부모가 콜백 함수를 props로 내려주고 자식이 이를 호출하는 방식으로 처리합니다.
- 여러 props를 전달할 때는 객체 구조 분해 할당을 활용하면 코드가 간결해집니다.


### 4. State

#### 개념
- State는 컴포넌트 내부에서 관리되는 데이터입니다. 값이 변경될 때마다 해당 컴포넌트와 하위 컴포넌트가 자동으로 리렌더링됩니다.
- State는 UI의 동적인 부분을 관리하는 데 필수적입니다.
- useState는 가장 기본적인 상태 관리 Hook입니다.
- useReducer는 복잡한 상태 로직이나 여러 값이 연관된 경우에 적합합니다.

#### 예시
```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>현재 카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}
```

#### 주요 상태 관리 도구

| 도구         | 설명                           |
|--------------|-------------------------------|
| Context API  | 전역 상태 공유에 적합하며, 구조가 단순합니다. |
| Redux        | 구조화된 전역 상태 관리에 적합하며, 대규모 애플리케이션에 주로 사용됩니다. |
| Zustand      | 가볍고 빠르며, 사용법이 간단합니다. |
| Recoil       | atom 기반의 상태 관리로, React 철학에 가깝습니다. |
| Jotai        | 미니멀한 전역 상태 관리 도구입니다. |

#### 특징
- State는 직접 변경하지 않고, 반드시 setState(함수형은 setX)로 갱신해야 합니다. 직접 변경 시 React의 상태 추적이 깨져 예기치 않은 동작이 발생할 수 있습니다.
- Derived State(파생 상태)는 props로부터 계산되는 state를 의미합니다. 불필요한 파생 상태는 최소화하여 데이터 일관성을 유지해야 합니다.
- 상태가 여러 컴포넌트에 걸쳐 필요하다면, 상태 끌어올리기(lifting state up)나 전역 상태 관리 도구를 활용해야 합니다.
- useState의 초기값 계산이 복잡하다면, 함수형 초기값 패턴을 사용하여 성능을 최적화할 수 있습니다.


### 5. CSR vs SSR

| 방식 | 설명 | 장점 | 단점 |
|------|------|------|------|
| CSR (Client-Side Rendering) | JS가 브라우저에서 HTML을 렌더링합니다. | 상호작용성이 우수합니다. | 초기 로딩이 느리고, SEO에 불리합니다. |
| SSR (Server-Side Rendering) | 서버에서 미리 렌더링된 HTML을 전달합니다. | 빠른 초기 로딩과 SEO에 유리합니다. | 서버 부하가 크고, 구현이 복잡해집니다. |

#### 프레임워크별 비교

| 프레임워크 | 렌더링 방식 | 특징 |
|------------|------------|------|
| React      | CSR        | 기본 방식입니다. |
| Next.js    | SSR/SSG/ISR| 유연하며, SEO 최적화에 강점이 있습니다. |
| SvelteKit  | SSR/CSR    | 번들 크기가 작고 빠릅니다. |
| Astro      | Static     | 최소한의 JS로 매우 빠른 성능을 제공합니다. |

#### 특징
- CSR은 SPA(Single Page Application)에서 주로 사용되며, 클라이언트에서 모든 라우팅과 렌더링이 이루어집니다.
- SSR은 서버에서 HTML을 미리 생성하여 전달하므로, 검색 엔진 최적화(SEO)와 첫 화면 로딩 속도에 유리합니다.
- Next.js는 SSR, SSG(Static Site Generation), ISR(Incremental Static Regeneration) 등 다양한 렌더링 방식을 지원합니다.
- 하이브리드 렌더링 전략을 통해 페이지별로 CSR/SSR/SSG를 선택적으로 적용할 수 있습니다.


### 6. Hydration

#### 개념
- 하이드레이션은 SSR로 생성된 HTML에 클라이언트의 JavaScript를 연결하여, 상호작용이 가능하게 만드는 과정입니다.
- 서버에서 전달된 정적 HTML에 React가 이벤트 핸들러를 바인딩하여 동적인 SPA로 전환됩니다.

#### 흐름
1. 서버에서 HTML과 초기 상태를 전달합니다.
2. 클라이언트에서 JS 번들을 로드합니다.
3. React가 기존 DOM에 이벤트 핸들러를 바인딩합니다.

#### 예시 (Next.js)
```tsx
export default function Home({ data }) {
  return <div>{data.message}</div>;
}
export async function getServerSideProps() {
  return { props: { data: { message: '서버에서 렌더링된 메시지입니다' } } };
}
```

#### 특징
- Hydration mismatch는 서버와 클라이언트의 렌더링 결과가 다를 때 발생하는 경고입니다. 상태 동기화에 각별히 주의해야 합니다.
- 하이드레이션 과정에서 클라이언트와 서버의 데이터가 불일치하면, React가 경고를 출력하고 일부 UI가 비정상적으로 동작할 수 있습니다.
- Next.js, Remix 등 SSR 프레임워크에서는 하이드레이션 최적화와 관련된 다양한 옵션을 제공합니다.


### 7. Isomorphic Component (동형 컴포넌트)

#### 개념
- 동형 컴포넌트는 동일한 React 컴포넌트가 서버와 클라이언트에서 모두 실행될 수 있는 구조를 의미합니다.
- 코드의 재사용성과 유지보수성을 극대화할 수 있습니다.

#### 장점
- 첫 화면 렌더링 속도가 매우 빠르며(TTFB), SEO에 최적화되어 있습니다.
- 서버와 클라이언트에서 동일한 코드를 사용하므로, 중복 구현을 줄일 수 있습니다.

#### 특징
- 환경별 분기 처리가 필요할 때는 `typeof window !== 'undefined'`와 같은 조건문을 사용하여 클라이언트/서버 코드를 구분합니다.
- 동형 컴포넌트는 SSR, SSG, CSR 등 다양한 렌더링 방식과 조합하여 사용할 수 있습니다.
- 서버 전용 코드(예: 파일 시스템 접근)와 클라이언트 전용 코드(예: 브라우저 API 사용)는 분리하여 관리해야 합니다.


### 8. React 디자인 패턴

#### 고차 컴포넌트 (HOC)
```jsx
function withLogger(Wrapped) {
  return function(props) {
    console.log('렌더링:', props);
    return <Wrapped {...props} />;
  };
}
```
- 고차 컴포넌트는 공통 로직을 추출하여 여러 컴포넌트에 재사용할 수 있는 패턴입니다. Wrapper Hell에 주의해야 합니다.

#### Render Props
```jsx
function MouseTracker({ render }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  return (
    <div onMouseMove={e => setPos({ x: e.clientX, y: e.clientY })}>
      {render(pos)}
    </div>
  );
}
```
- Render Props 패턴은 동적인 UI를 구현할 때 유용하지만, 코드 가독성이 저하될 수 있습니다.

#### Compound Components (컴파운드 컴포넌트)
```jsx
function Tabs({ children, defaultTab }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ id, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  const isActive = activeTab === id;
  
  return (
    <button 
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

function TabPanels({ children }) {
  return <div className="tab-panels">{children}</div>;
}

function TabPanel({ id, children }) {
  const { activeTab } = useContext(TabsContext);
  return activeTab === id ? <div className="tab-panel">{children}</div> : null;
}

// 사용 예시
<Tabs defaultTab="tab1">
  <TabList>
    <Tab id="tab1">첫 번째 탭</Tab>
    <Tab id="tab2">두 번째 탭</Tab>
  </TabList>
  <TabPanels>
    <TabPanel id="tab1">첫 번째 탭 내용</TabPanel>
    <TabPanel id="tab2">두 번째 탭 내용</TabPanel>
  </TabPanels>
</Tabs>
```

#### 개념
- 컴파운드 컴포넌트 패턴은 여러 컴포넌트가 함께 작동하여 하나의 기능을 구현하는 패턴입니다.
- 내부적으로 Context API를 활용하여 상태를 공유하며, 사용자에게는 선언적이고 직관적인 API를 제공합니다.

#### 장점
- 높은 재사용성과 유연성을 제공합니다.
- 각 컴포넌트의 역할이 명확하게 분리되어 있습니다.
- 사용자가 원하는 구조로 자유롭게 조합할 수 있습니다.

#### 특징
- Context API를 활용하여 부모-자식 간 깊은 prop drilling을 방지합니다.
- React.Children API를 사용하여 children을 동적으로 조작할 수도 있습니다.
- Tabs, Accordion, Modal, Dropdown 등 복합적인 UI 컴포넌트에서 주로 사용됩니다.
- 컴포넌트 간의 암묵적 의존성이 생기므로, 타입스크립트 환경에서는 타입 안전성에 특별히 주의해야 합니다.

#### 추가 디자인 패턴 특징
- Custom Hooks는 로직 재사용의 표준으로, 복잡한 상태 관리나 비동기 로직을 분리할 때 유용합니다.
- 컨텍스트(Context)와 결합하여 복잡한 상태 공유 패턴을 구현할 수 있습니다.


#### 상태 끌어올리기 (Lifting State Up)

#### 개념
- 여러 자식 컴포넌트가 동일한 state를 공유해야 할 때, 공통 부모 컴포넌트로 상태를 끌어올려 관리합니다.
- 상태를 끌어올리면 데이터 일관성과 동기화가 쉬워집니다.

#### 예시
```jsx
function Parent() {
  const [value, setValue] = useState('');
  return (
    <>
      <Input value={value} onChange={setValue} />
      <Preview value={value} />
    </>
  );
}
```

#### 특징
- Prop drilling(상태 전달이 깊어지는 현상)이 심해지면 Context API나 전역 상태 관리 도구의 도입을 고려해야 합니다.
- 상태를 끌어올릴 때는, 각 컴포넌트의 책임과 역할을 명확히 분리하는 것이 중요합니다.
- 불필요하게 모든 상태를 상위로 끌어올리면 컴포넌트 트리가 복잡해질 수 있으므로, 필요한 범위 내에서만 적용해야 합니다.


### Hooks (React 16.8+)

#### 대표 Hook

| Hook        | 설명                |
|-------------|---------------------|
| useState    | 컴포넌트의 상태를 선언하고 관리합니다. |
| useEffect   | 부수효과(사이드 이펙트)를 관리합니다. |
| useContext  | 전역 상태를 공유합니다. |
| useReducer  | 복잡한 상태를 처리합니다. |
| useMemo     | 연산 결과를 메모이제이션하여 최적화합니다. |
| useCallback | 함수를 메모이제이션하여 불필요한 렌더링을 방지합니다. |

### 부록 - useEffect와 부수효과(Side Effects)
---

#### 부수효과란?
- 부수효과는 컴포넌트의 렌더링 결과와 직접 관련이 없는 외부와의 상호작용을 의미합니다. 예를 들어, API 요청, 타이머, 이벤트 리스너 등록/해제, 외부 라이브러리 초기화, WebSocket 구독 등이 있습니다.

#### 대표 예시
```jsx
// 마운트 시 API 호출
useEffect(() => {
  fetch('/api/data')
    .then(res => res.json())
    .then(data => setData(data));
}, []);
```
```jsx
// 이벤트 등록과 정리
useEffect(() => {
  const handleResize = () => console.log('윈도우 크기 변경됨');
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```
```jsx
// 디바운싱 구현
useEffect(() => {
  const timer = setTimeout(() => {
    fetch(`/api/search?q=${query}`);
  }, 500);
  return () => clearTimeout(timer);
}, [query]);
```

#### 주의사항
- 의존성 배열을 생략하면 무한 루프가 발생할 수 있으니 주의해야 합니다.
- useEffect에 async 함수를 직접 전달하지 않고, 내부에서 별도의 async 함수를 정의해 사용해야 합니다.
- clean-up 함수를 반드시 작성하여 메모리 누수를 방지해야 합니다.

#### 특징
- Custom Hook을 만들어 여러 컴포넌트에서 공통적으로 사용하는 로직을 추출할 수 있습니다.
- useLayoutEffect는 DOM 변경 직후 동기적으로 실행되어야 하는 작업에 사용합니다.
- useRef를 활용하면 렌더링과 무관한 값을 저장하거나, DOM 요소에 직접 접근할 수 있습니다.


### 9. 성능 최적화 기법

#### 코드 스플리팅 & 동적 import
React 애플리케이션이 커질수록 초기 번들 크기가 커져 로딩 속도가 느려질 수 있습니다. 코드 스플리팅은 필요한 시점에 필요한 코드만 로드하여 초기 로딩 속도를 개선하는 전략입니다.

```js
// 정적 import
import Component from './Component';
// 동적 import (Next.js 기준)
const Component = dynamic(() => import('./Component'));
```

- React.lazy와 Suspense를 활용하면 컴포넌트 단위로 코드 스플리팅이 가능합니다.
- Next.js의 dynamic import는 SSR 환경에서도 코드 스플리팅을 지원합니다.
- 코드 스플리팅 시 fallback UI(로딩 스피너 등)를 신경 써야 하며, Suspense의 범위 설정에 따라 UX가 달라질 수 있습니다.
- 너무 세분화된 코드 스플리팅은 오히려 네트워크 요청이 많아져 역효과가 날 수 있으니, 주요 라우트/대형 컴포넌트 단위로 적용하는 것이 일반적입니다.

#### 코드 스플리팅 예시
```jsx
const LazyComponent = React.lazy(() => import('./Component'));

<Suspense fallback={<div>로딩 중...</div>}>
  <LazyComponent />
</Suspense>
```

#### React.memo, useMemo, useCallback
- **React.memo**: props가 변경되지 않으면 컴포넌트의 리렌더링을 방지하는 고차 컴포넌트입니다. 값이 자주 바뀌지 않는 하위 컴포넌트에 적용하면 불필요한 렌더링을 줄일 수 있습니다.
  - 단, props로 전달되는 객체/함수/배열이 매번 새로 생성되면 효과가 없습니다. useMemo/useCallback과 함께 사용해야 진가를 발휘합니다.
  - 지나친 memoization은 오히려 메모리 사용량과 복잡도를 높일 수 있으니, 렌더링 병목이 실제로 발생하는 부분에만 적용하는 것이 좋습니다.

- **useMemo**: 연산 비용이 큰 값을 메모이제이션하여, 의존성 배열이 바뀔 때만 재계산합니다. 리스트 필터링, 정렬, 복잡한 계산 등에서 유용합니다.
  - useMemo(() => expensiveFn(a, b), [a, b])
  - 의존성 배열을 잘못 지정하면 값이 갱신되지 않거나, 불필요하게 재계산될 수 있으니 주의해야 합니다.

- **useCallback**: 함수를 메모이제이션하여, 의존성 배열이 바뀔 때만 새 함수를 생성합니다. 자식 컴포넌트에 콜백을 props로 넘길 때, 불필요한 리렌더링을 방지할 수 있습니다.
  - useCallback(() => fn(a), [a])
  - useCallback과 useMemo는 지나치게 남용하면 코드가 복잡해질 수 있으니, 실제 성능 병목이 있는 곳에만 적용합니다.

#### 리스트 가상화 (Virtualization)
- 대량의 데이터(수백~수천 개)를 렌더링할 때, 모든 DOM을 한 번에 그리면 브라우저 성능이 급격히 저하됩니다.
- react-window, react-virtualized 등 라이브러리를 사용하면, 실제로 화면에 보이는 영역만 DOM으로 렌더링하고, 스크롤에 따라 필요한 부분만 동적으로 추가/제거합니다.
- 가상화는 스크롤 위치, 셀 크기, 동적 높이 등 다양한 옵션을 지원하며, 대형 테이블/리스트/갤러리 등에서 필수적입니다.
- 가상화 적용 시, key 관리, 동적 높이, 스크롤 동기화, 셀 focus 등 세부 구현에 주의해야 합니다.

#### PRPL 패턴
| 단계      | 설명                           |
|-----------|-------------------------------|
| Push      | HTML, CSS, 핵심 JS를 미리 전송합니다.  |
| Render    | 빠르게 첫 화면을 그립니다.         |
| Pre-cache | 다음 경로의 리소스를 캐시합니다.      |
| Lazy-load | 필요한 시점에 나머지 리소스를 로드합니다. |

- PRPL 패턴은 Google에서 제안한 웹 성능 최적화 전략으로, React SPA/SSR 환경 모두에 적용할 수 있습니다.
- Push: 서버에서 핵심 리소스를 HTTP/2 Push 등으로 미리 전송
- Render: 최소한의 JS/CSS로 첫 화면을 빠르게 렌더링
- Pre-cache: Service Worker 등으로 다음 페이지 리소스 미리 캐싱
- Lazy-load: 실제로 필요한 시점에만 나머지 리소스 로드
- Next.js, CRA 등에서는 prefetch, preload, dynamic import 등으로 PRPL 패턴을 구현할 수 있습니다.