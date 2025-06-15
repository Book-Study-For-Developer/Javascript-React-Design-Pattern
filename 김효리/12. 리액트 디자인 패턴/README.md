리액트는 익숙하기 때문에 몰랐던 사실들만 모아서 정리!

## 12.2 고차 컴포넌트 (Higher-Order Component, HOC)

**정의**: 고차 컴포넌트는 컴포넌트를 입력으로 받아 **새로운 기능을 부여한 컴포넌트를 반환하는 함수**입니다.

```tsx
function withLoadingSpinner(Component) {
  return function WithLoadingSpinner({ isLoading, ...props }) {
    if (isLoading) return <div>Loading...</div>;
    return <Component {...props} />;
  };
}

// 사용 예시
const UserProfileWithSpinner = withLoadingSpinner(UserProfile);
```

### 🔍 용도

- 인증 검사
- 로딩 처리
- 로깅, 퍼포먼스 측정 등

### ⚠️ 단점

- 어떤 HOC가 어떤 props를 주입하는지 **코드 외부에서는 추적이 어렵다**.
- 여러 HOC가 중첩되면 **디버깅과 유지보수가 어려워진다** (Wrapper Hell).
- React DevTools에서는 원래의 컴포넌트 이름 대신 HOC의 이름이 뜰 수 있어 추적이 까다롭다.

### ✅ 보완 방법

- `displayName`을 수동으로 설정해 디버깅 용이성 확보
- 가능한 HOC를 단일 책임으로 나눔

---

## 12.3 렌더 Props 패턴 (Render Props)

**정의**: props로 함수를 전달하여 자식 컴포넌트의 렌더링을 제어하는 패턴

```tsx
function Input({ render }) {
  const [value, setValue] = useState('');

  return (
    <>
      <input
        type='text'
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder='Temp in °C'
      />
      {render(value)}
    </>
  );
}

function App() {
  return (
    <div>
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

### 🎯 목적

- **상태 재사용**: 상태 관리 로직을 재사용하되, UI는 각기 다르게 렌더링
- **유연성**: 전달받은 함수가 자식 UI를 정의하므로, **컴포넌트 간 조합이 자유로움**

### ➕ `children`으로도 대체 가능

```tsx
<Input>
  {(value) => (
    <>
      <Kelvin value={value} />
      <Fahrenheit value={value} />
    </>
  )}
</Input>
```

---

## 12.8 코드 스플리팅 (Code Splitting)

**정의**: 앱을 여러 개의 작은 번들로 나누어 **필요할 때마다 로드하도록 만드는 기법**

```tsx
const LazyComponent = React.lazy(() => import('./MyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

### 🎯 왜 중요한가?

- 큰 번들은 초기 로딩 시간을 증가시켜 **사용자 경험(FCP, LCP, TTI)을 저해**
- 페이지 첫 진입에서 **불필요한 리소스를 지연 로드**하여 체감 성능을 높임

### 📈 지표 설명

- **FCP (First Contentful Paint)**: 처음으로 텍스트/이미지 등 의미 있는 콘텐츠가 렌더링된 시점
- **LCP (Largest Contentful Paint)**: 가장 큰 콘텐츠가 화면에 완전히 표시되는 시간 (페이지 로딩 성능의 핵심 지표)
- **TTI (Time to Interactive)**: 사용자가 실제로 상호작용할 수 있는 시점

---

## 12.9 PRPL 패턴

**정의**: 고성능 웹 앱을 위한 성능 최적화 전략

| 단계          | 설명                                                 |
| ------------- | ---------------------------------------------------- |
| **Push**      | 중요한 리소스를 HTTP/2 서버 푸시로 클라이언트에 전달 |
| **Render**    | 초기 경로를 빠르게 렌더링                            |
| **Pre-cache** | 자주 방문하는 경로의 에셋을 백그라운드에서 미리 캐싱 |
| **Lazy-load** | 나머지 리소스를 나중에 비동기로 로딩                 |

### 🔗 HTTP/1.1 vs HTTP/2

| 항목      | HTTP/1.1          | HTTP/2                                             |
| --------- | ----------------- | -------------------------------------------------- |
| 요청 처리 | 순차적            | 병렬                                               |
| 성능      | HOL Blocking 존재 | 다중 스트림으로 해결                               |
| 서버 푸시 | 없음              | 있음 (서버가 클라이언트에 리소스를 미리 전달 가능) |

### 💡 HOL Blocking이란?

> Head-of-Line Blocking: 앞의 요청이 완료되기 전까지 뒤의 요청들이 대기하는 문제.
> HTTP/2는 이 문제를 **멀티플렉싱(Multiplexing)** 으로 해결하여 **한 커넥션에 여러 요청을 병렬 처리**할 수 있게 합니다.

### 🎯 PRPL 패턴의 목적

- 사용자에게 **첫 화면을 빠르게 표시** (초기 경로 집중)
- **서비스 워커**와 `preload`를 조합하여 **불필요한 리소스 요청 최소화**
- 다음 방문에서 빠른 응답을 위해 **사전 캐싱된 리소스를 재사용**

---

## 12.10 로딩 우선순위

리소스 로딩의 우선순위를 조절하여 **핵심 경로의 렌더링을 방해하지 않도록 최적화**합니다.

```tsx
<link rel='preload' href='/main.js' as='script' />
```

### 💡 preload의 주의점

- 중요한 JS 리소스를 빠르게 가져올 수 있으나,
- **FCP, LCP에 필요한 리소스를 늦게 불러오게 될 경우 UX가 저하**될 수 있음
- 상호작용에 필요한 리소스(`TTI`, `FID`)와 시각적 요소(`FCP`, `LCP`) 간 밸런스가 중요

---

### 12.10.1 SPA에서의 Preload

```tsx
const EmojiPicker = import(/* webpackPreload: true */ './EmojiPicker');
```

- 해당 모듈은 **메인 번들에는 포함되지 않지만**, 브라우저가 **백그라운드에서 병렬로 가져오도록 힌트**를 줌
- 초기 렌더 이후 1초 이내에 사용자에게 보여질 요소에만 적용하는 것이 이상적

---

## 🔚 마무리

React 앱을 설계할 때는 단순히 기능을 구현하는 것을 넘어서, **사용자 경험과 유지보수성, 성능을 함께 고려하는 구조적 사고**가 중요합니다.

- HOC, Render Props는 **코드 재사용성 패턴**
- PRPL, Preload는 **성능 최적화 전략**
- 이 모든 개념은 서로 연결되며, 팀 규모가 커질수록 **패턴의 일관성**이 중요한 요소로 작용합니다.
