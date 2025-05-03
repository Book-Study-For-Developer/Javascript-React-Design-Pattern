새로운 패턴은 채택되기 까지 여러 차례에 걸친 심사를 받아야 한다. 이때 **세 가지 법칙을 충족**해 디자인 패턴으로서 인정받게 된다.

## 프로토 패턴이란?

새로운 패턴이 등장하면 기본적으로 **철저한 조사와 검증을 거치기 전까지는 패턴으로 간주하면 안 된다.**

새로운 패턴이 등장하고 실제로 적용했을 때 구조를 시각적으로 표현할 수 있어야 한다.

> [!NOTE]
>
> 🤔 여기서 헷갈렸던 점
>
> 여기서 proto-pattern 이라고 해서 구글링했을 때는 prototype pattern만 주구장창 나왔다.
> 근데 여기서 말하는 **프로토 패턴과 프로토타입 패턴은 서로 다른 것**
>
> 프로토 패턴은 말그대로 아직 정착되지 않은 패턴을 의미하는 것이었다. (타이틀은 프로토 패턴인데 왜 설명을 안해주는 것인가..)

### 패턴성 검증

프로토 패턴이란 아직 ‘패턴성’ 검증을 모두 통과하지 않은 미숙한 패턴을 의미

새롭게 등장한 패턴에 대해서 간단한 설명을 덧붙일 때 코드 조각들을 패틀릿(patlet)이라고 부른다.

좋은 패턴으로 간주할 수 있는 조건

- 원리나 방법만 담고 있지 않고 **특정 문제를 해결**할 수 있어야 한다.
- 문제 해결 기법은 기본 원칙에서 도출되는 것이 좋다. 최고의 디자인 패턴들은 **해결책은 간접적으로 제공**된다.
- 디자인 패턴은 **설명에 쓰인 그대로 동작**해야 하며 추측에 의존해 동작하면 안 된다.
- 패턴의 공식 설명은 **코드와의 관계를 나타내는 심층 구조와 메커니즘을 서술**해야 한다.

> [!NOTE]
> 🤔 무슨 말인지 와닿지 않아 내가 이해한 식으로 다시 쓰기
>
> 원문 살펴보기: https://bcanotes4u171863936.wordpress.com/patterns/
>
> - 이론상으로만 완벽하지 않고, 실제로 해결이 가능한 패턴이어야 한다.
> - “은탄환은 없다” 와 같은 말 같이 디자인 패턴이 모든걸 해결해주진 않는다.
> - 디자인 패턴의 적용하였는데, 사람마다 다르게 동작하면 안 되고 모두가 동일한 패턴의 기대 동작이 이뤄져야 한다.
> - It describes a relationship: patterns do not just describe modules but describe deeper structure and mechanism.
>   - 이게 제일 이해 안갔는데, 원문을 보고 그나마 이해한 해석은 **프로토 패턴을 실제로 코드로 구현했을 때에 구조적으로 설명이 가능해야 한다**

## 세 가지 법칙

좋은 패턴이 되기 위해서 세 가지 질문에 답할 수 있어야 한다.

1. **목적 적합성:** 좋은 패턴은 어떻게 판단하나요?
2. **유용성:** 좋은 패턴이라고 할 수 있는 이유가 무엇인가요?
3. **적용 가능성:** 넓은 적용 범위를 가지고 있어 패턴이 될 가치가 있나요?

<hr />

> [!NOTE]
>
> 🤔 최근에 내가 쓴 DX 해소에 넣은 컴포넌트를 패턴으로 할 수 있을까? 해서 질문에 답해본다 ㅋㅋㅋ

```tsx
type PromiseValues = readonly (() => Promise<unknown>)[] | [];

type PromiseAllAwaitedReturnType<T extends PromiseValues> = {
  -readonly [K in keyof T]: Awaited<ReturnType<T[K]>>;
};

type Props<T extends PromiseValues> = {
  fetchFunctions: T;
  children: (results: PromiseAllAwaitedReturnType<T>) => JSX.Element;
};

export const FetchBoundary = async <T extends PromiseValues>({
  fetchFunctions,
  children,
}: Props<T>) => {
  try {
    const results = (await Promise.all(
      fetchFunctions.map((fetchFunction) => fetchFunction())
    )) as PromiseAllAwaitedReturnType<T>;

    return children(results);
  } catch (error: unknown) {
    throw error;
  }
};
```

1. 비지니스 로직을 가진 컴포넌트와 비동기 요청을 진행하는 컴포넌트간의 선이 생겼습니다.
2. 서버 환경에서 사용되는 API 요청 부분과 사용될 컴포넌트 책임을 분리할 수 있게 되어졌습니다.
3. API fetching 라이브러리, 프레임워크와 관계없이 비동기 함수를 사용하는 모든 곳에 대해서 모두 적용이 가능합니다.
