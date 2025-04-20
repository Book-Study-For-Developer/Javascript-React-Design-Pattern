## 애플리케이션 분리의 중요성

**확장 가능한 모듈형 자바스크립트를 사용**하여 **의존성을 낮추어** 유지보수를 용이하게 만든다. 모듈 시스템이 필요해지자 AMD, CommonJs 와 같은 모듈이 등장하게 되었다.

최근에 가장 자주 사용되는 ECMAScript 모듈이라는 표준이 발표되었다.

## 모듈 가져오기와 내보내기

모듈을 사용하여 각 기능에 맞는 독립적인 단위로 코드를 분리할 수 있게 되었다. import/export 키워드를 통해 모듈을 가져오고 내보낼 수 있다.

```tsx
export const baker = {
  bake(item) {
    console.log(`Woo! I just baked ${item}`);
  },
};

export const pastryChef = {
  make(item) {
    console.log(`Woo! I just made a ${item}`);
  },
};

export default { baker, pastryChef };
```

```tsx
import { baker } from "./staff.mjs";

export const cakeFactory = {
  oven: {
    makeCupcake(toppings) {
      baker.bake("cupcake", toppings);
    },
    makeMuffin(mSize) {
      baker.bake("muffin", mSize);
    },
  },
};
```

```tsx
import { cakeFactory } from "./cakeFactory_2.mjs";
cakeFactory.oven.makeCupcake("sprinkles");
cakeFactory.oven.makeMuffin("large");
```

위 예시는 흔히 우리가 개발할때 쓰이는 import/export 구문들이다. 따로 설명은 적지 않았다.

```tsx
<script type="module" src="main.mjs"></script>
<script nomodule src="fallback.js"></script>
```

nomodule 속성을 통해 모듈이 아님을 나타내줄 수 있고, **모듈이 제대로 동작하지 않는 브라우저에서도 동작**해주도록 한다.

### 모듈 객체

```tsx
import * as staff from "./staff.mjs";

export const cakeFactory = {
  oven: {
    makeCupcake(toppings) {
      staff.baker.bake("cupcake", toppings);
    },
    makeMuffin(mSize) {
      staff.pastryChef.make("muffin", mSize);
    },
  },
};
```

`‘*’` 를 사용하여 모듈을 객체 하나로 관리할 수 있게 된다.

🤔 개인적으로 이런 형태보다 export로 이름을 명시해서 가져오는걸 선호합니다.

### 정적/동적으로 모듈 가져오기

정적으로 모듈을 가져오게 될 경우 초기 페이지 로드 시에 미리 js들을 모두 다운로드 해야 하기 때문에 성능에 문제가 발생할 수 있다. 이를 동적으로 바꾸면 **필요한 시점에만 로드하기 때문에 로딩 속도에 있어서 장점**을 가져갈 수 있다.

```tsx
  <script>
    document.addEventListener("DOMContentLoaded", () => {
      const form = document.getElementById("cakeForm");

      form.addEventListener("submit", e => {
        e.preventDefault();
        import("./modules/cakeFactory.mjs")
          .then((module) => {
            // Do something with the module.
            module.cakeFactory.oven.makeCupcake("sprinkles");
            module.cakeFactory.oven.makeMuffin("large");
          })
          .catch((error) => {
            console.error("Error importing module:", error);
          });
      });
    });
  </script>
```

html 스크립트 태그 내에서 돔이 로드되었을 때 `Form` 태그에서 submit을 진행할 때 동적으로 모듈을 불러와 함수를 실행할 수 있다. (이를 await 키워드 사용해서도 가능하다.)

### 사용자 상호작용에 따라 가져오기

```tsx
<script type="module">
  const btn = document.querySelector("button");

  btn.addEventListener("click", async (e) => {
    e.preventDefault();
    try {
      const sortBy = await import(
        "https://cdn.jsdelivr.net/npm/lodash-es@4.17.21/sortBy.js"
      );
      sortInput(sortBy.default); // use the imported dependency
    } catch (err) {
      console.log(err);
    }
  });

  function sortInput(sortBy) {
    // Your sorting logic goes here, e.g.:
    const input = [5, 3, 1, 4, 2];
    const sortedInput = sortBy(input);
    console.log(sortedInput);
  }
</script>

```

사용자가 button을 클릭할때 모듈을 불러와서 함수를 실행할 수 있다.

🤔 js로만 구현하더라도 이런 동적은 Import를 쓰진 않았을 듯 하다.

이유는,, 사용자 경험이 별로 좋지 않을 것 같기 때문

### 모듈을 사용하면 생기는 이점

- 한 번만 실행된다.
  - 모듈 스크립트는 한 번만 실행된다. 가장 내부에 위치한 모듈이 먼저 평가되고 여기에 의존중인 모듈에 접근할 수 있다.
- 자동으로 지연 로드된다.
  - 모듈은 자동으로 지연되어 로드된다.
- 유지보수와 재사용이 쉽다.
  - 독립적으로 실행될 수 있는 코드 조각으로 관리된다.
- 네임스페이스를 제공한다.
  - 모듈과 관련된 변수, 상수가 개별 공간에 생성되어 전역 네임스페이스를 오염시키지 않는다.
- 사용하지 않는 코드를 제거한다.
  - 모듈을 통해 코드를 가져오면 번들러에 의해 트리쉐이킹 된다.

🤔 모듈 스크립트 이점 다시 알아보기

ESM에서는 defer 가 기본으로 적용된다.

```jsx
<script type="module" src="main.mjs"></script>
<script nomodule defer src="fallback.js"></script>
```

type=”module”로 지정한 스크립트는 한 번만 평가된다.

```tsx
<script src="classic.js"></script>
<script src="classic.js"></script>
<!-- 2번 실행 -->

<script type="module" src="module.mjs"></script>
<script type="module" src="module.mjs"></script>
<script type="module">import './module.mjs';</script>
<!-- 1번만 실행 -->
```

commonjs로 작성되면 트리 쉐이킹이 동작하지 않는다.

```jsx
// utils.js (CommonJS)
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = {
  add,
  subtract,
};

// main.js
const { add } = require("./utils");

console.log(add(2, 3));
```

add만 사용중이지만 subtract 함수도 같이 불러와진다.

### 클래스 문법

클래스 문법이 생겼고, Private, Public, static 키워드 사용이 가능해졌다.

> [!NOTE]
> 🤔 대부분 아는 문법이라 생각되어 따로 정리하지 않았습니다.

<hr />

다만 react-error-boundary가 여전히 class문법을 사용중이기 때문에 이를 좀 더 살펴보겠습니다.

https://github.com/bvaughn/react-error-boundary/blob/master/src/ErrorBoundary.ts

```tsx
export class ErrorBoundary extends Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);

    this.resetErrorBoundary = this.resetErrorBoundary.bind(this);
    this.state = initialState;
  }

  static getDerivedStateFromError(error: Error) {
    return { didCatch: true, error };
  }

  // 에러 상태 초기화
  resetErrorBoundary(...args: any[]) {
    const { error } = this.state;

    if (error !== null) {
      this.props.onReset?.({
        args,
        reason: "imperative-api",
      });

      this.setState(initialState);
    }
  }

  // 에러 발생시 catch
  componentDidCatch(error: Error, info: ErrorInfo) {
    this.props.onError?.(error, info);
  }

  componentDidUpdate(
    prevProps: ErrorBoundaryProps,
    prevState: ErrorBoundaryState
  ) {
    const { didCatch } = this.state;
    const { resetKeys } = this.props;

    // There's an edge case where if the thing that triggered the error happens to *also* be in the resetKeys array,
    // we'd end up resetting the error boundary immediately.
    // This would likely trigger a second error to be thrown.
    // So we make sure that we don't check the resetKeys on the first call of cDU after the error is set.

    // 에러 발생한 원인이 resetKeys 배열안에 있을 수 있는 엣지 케이스가 존재할 수 있다.
    // 따라서 즉시 에러바운더리가 리셋될 수 있는데, 이때 또 에러가 발생할 수 있다. (무한 루프 or 연속 에러가 가능성이 있음)
    // 그래서 첫번째 componentDidUpdate 호출에서는 resetKeys를 검사하지 않도록 처리한다.
    if (
      didCatch &&
      prevState.error !== null &&
      hasArrayChanged(prevProps.resetKeys, resetKeys)
    ) {
      this.props.onReset?.({
        next: resetKeys,
        prev: prevProps.resetKeys,
        reason: "keys",
      });

      this.setState(initialState);
    }
  }

  // 에러 Fallback 컴포넌트를 랜더링하는 함수
  render() {
    const { children, fallbackRender, FallbackComponent, fallback } =
      this.props;
    const { didCatch, error } = this.state;

    let childToRender = children;

    if (didCatch) {
      const props: FallbackProps = {
        error,
        resetErrorBoundary: this.resetErrorBoundary,
      };

      if (typeof fallbackRender === "function") {
        childToRender = fallbackRender(props);
      } else if (FallbackComponent) {
        childToRender = createElement(FallbackComponent, props);
      } else if (fallback !== undefined) {
        childToRender = fallback;
      } else {
        if (isDevelopment) {
          console.error(
            "react-error-boundary requires either a fallback, fallbackRender, or FallbackComponent prop"
          );
        }

        throw error;
      }
    }

    return createElement(
      ErrorBoundaryContext.Provider,
      {
        value: {
          didCatch,
          error,
          resetErrorBoundary: this.resetErrorBoundary,
        },
      },
      childToRender
    );
  }
}
```

react-error-boundary는 왜 여전히 class문법을 사용중인가?

- [`getSnapshotBeforeUpdate`](https://react.dev/reference/react/Component#getsnapshotbeforeupdate): 가장 마지막으로 렌더링 된 결과가 DOM 등에 반영되었을 때 호출
- [`getDerivedStateFromError`](https://react.dev/reference/react/Component#static-getderivedstatefromerror): 하위의 자손 컴포넌트에서 오류가 발생했을 때 render 단계에서 호출
- [`componentDidCatch`](https://react.dev/reference/react/Component#componentdidcatch): 하위의 자손 컴포넌트에서 오류가 발생했을 때 commit 단계에서 호출

위 세가지 라이프 사이클을 Hooks에서 지원하지 않기 때문이다.

react-error-boundary 라이브러리에서는 `getDerivedStateFromError`와 `componentDidCatch` 라이프사이클을 사용중이기 때문에 여전히 **class 문법을 사용중**이다.
