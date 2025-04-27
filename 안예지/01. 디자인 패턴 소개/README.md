# 디자인 패턴 소개

디자인 패턴에 대한 지식은 요구사항 속에서 반복되는 주제를 파악해 이에 적절한 솔루션으로 이끌어 준다.
이전에 비슷한 문제를 마주하여 최적화된 방법으로 풀이해 낸 사람들의 경험에 기댈 수도 있다.
유지보수가 쉬운 코드를 작성하거나 리팩터링할 수 있는 길을 열어준다는 점에서 굉장히 중요하다.

**디자인 패턴은 어딘가에 얽매이거나, 어느 한 언어에 국한되지 않는다는 점이 중요하다.**

## 1.1 디자인 패턴의 역사

GoF(Gang of Four)의 디자인 패턴은 1994년 출판된 "Design Patterns: Elements of Reusable Object-Oriented Software" 책에서 시작되었다.
이리히 감마(Erich Gamma), 리처드 헬름(Richard Helm), 랄프 존슨(Ralph Johnson), 존 블리시디스(John Vlissides)로 구성된 4명의 저자들이 객체지향 설계에서 자주 발생하는 문제들에 대한 23가지 디자인 패턴을 정리했다.

## 1.2 패턴이란 무엇인가

디자인 패턴의 주요 특징

- 오랜 시간 동안 개발자들에 의해 **검증된 효과적인 접근 방식**이다.
- 사용자의 요구에 맞춰 **쉽게 재사용**할 수 있다.
- 정해진 구조와 공통 표현을 사용하여 알아보기 쉽다.
- 정해진 패턴을 사용하면 보다 구조적이고 체계적인 코드를 자연스럽게 작성할 수 있어 **개발 과정에서 생길 수 있는 큰 문제를 방지**할 수 있다.
- 특정 언어, 특정 애플리케이션의 형태와 상관없이 **종합적인 해결책을 제시**한다.
- 일반화된 함수를 사용하여 **반복되는 작업을 줄이고 코드의 양을 줄일** 수 있다.
- **공통된 어휘를 사용**하기 때문에 의사소통이 원활해진다.
- 새로운 디자인 패턴이 생성되기도 하고, 기존 패턴을 개선하는 방법으로 새롭게 생성되기도 해서 선순환 구조로 더 견고해질 수 있다.

## 1.3 패턴 활용 사례

### Provider 패턴

**개념**: 앱 내의 여러 컴포넌트들이 중간 컴포넌트를 거치지 않고 데이터를 공유할 수 있게 해주는 패턴
**해결하는 문제**: Props Drilling (데이터를 사용하지 않는 중간 컴포넌트들에 props를 전달해야 하는 문제)
**구현 방법**: React의 Context API, Vue의 Provide/Inject API 등을 활용

#### 다양한 프레임워크에서의 Provider 패턴 구현

**1. React - Context API**
```javascript
const DataContext = React.createContext();

function App() {
  const data = { user: 'John', theme: 'dark' };
  return (
    <DataContext.Provider value={data}>
      <MainComponent />
    </DataContext.Provider>
  );
}

function DeepNestedComponent() {
  const data = React.useContext(DataContext);
  return <div>{data.user}</div>;
}
```

**2. Vue.js - Provide/Inject API**
```javascript
// 상위 컴포넌트
export default {
  provide() {
    return {
      theme: this.theme
    }
  },
  data() {
    return {
      theme: 'dark'
    }
  }
}

// 깊은 하위 컴포넌트
export default {
  inject: ['theme'],
  created() {
    console.log(this.theme)
  }
}
```

**장점**
- 데이터를 중간 컴포넌트를 거치지 않고 직접 전달 가능
- 컴포넌트 간 결합도 감소
- 코드 재사용성 및 유지보수성 향상

**단점**
- 모든 프레임워크에서 과도한 사용 시 애플리케이션 구조 파악이 어려울 수 있음
- 데이터 흐름 추적이 복잡해질 수 있음
- 불필요한 렌더링/업데이트가 발생할 수 있음

**성능 개선 방법**
1. 상태 분리: 자주 변경되는 데이터와 정적인 데이터 분리
2. 메모이제이션: 각 프레임워크에서 제공하는 성능 최적화 기법 활용
   - React: memo, useMemo, useCallback
   - Vue: computed 속성, memoization 라이브러리
3. 상태 관리 라이브러리 활용:
   - React: Redux, MobX, Recoil
   - Vue: Vuex, Pinia

Provider 패턴은 이처럼 다양한 프레임워크에서 구현되며, 컴포넌트 간 결합도를 낮추고 데이터 공유를 효율적으로 만드는 데 기여합니다.
