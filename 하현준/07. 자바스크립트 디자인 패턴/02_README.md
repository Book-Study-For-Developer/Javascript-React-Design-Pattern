# 행위 패턴

객체 간의 의사소통을 돕는 패턴이고, 서로 다른 객체 간의 의사소통 방식을 개선하고 간소화하는 것을 목적으로 함

## 관찰자 패턴

한 객체가 변경 될 때 객체들이 변경되었음을 알릴 수 있게 해주는 패턴이며, 변경된 객체는 누가 자신을 구독하는지 알 필요 없이 알림을 보낼 수 있다.

구성요소

**주체**

- 관찰자 리스트를 관리하고, 추가와 삭제가 가능하게 한다.

**관찰자**

- 주체의 상태 변화 알림을 감지하는 update 인터페이스를 제공한다.

**구체적 주체**

- 상태 변화에 대한 알림을 모든 관찰자에게 전달하고 구체적 관찰자의 상태를 저장한다.

**구체적 관찰자**

- 구체적 주체의 참조를 저장하고, 관찰자의 update 인터페이스를 구현하여 주체의 상태 변화와 관찰자의 상태변화가 일치할 수 있도록 한다.

기본 관찰자 패턴 코드 ⇒ https://github.com/addyosmani/learning-jsdp/blob/main/ch07/observer.html

### 관찰차 패턴 vs 발행/구독 패턴

관찰자 패턴

⇒ React-Query도 여기에 속함

- **주체(Subject)** 가 직접 **관찰자(Observer)** 들을 관리한다.
- 관찰자는 주체에 **등록**(subscribe)되고, 주체가 **직접 알림(notify)** 을 준다.
- **1:다 관계** — 하나의 주체가 여러 관찰자를 알린다.

발행/구독 패턴

- **발행자(Publisher)** 와 **구독자(Subscriber)** 가 **서로를 모른다**.
- 중간에 **이벤트 브로커(Event Bus, Message Broker)** 같은 **매개체**가 있다.
- 발행자는 이벤트를 **브로커에 전달**하고, 브로커가 **구독자에게 이벤트를 전달**한다.

```tsx
class EventBus {
  private subscribers: Map<string, ((data: any) => void)[]> = new Map();

  subscribe(eventType: string, callback: (data: any) => void) {
    if (!this.subscribers.has(eventType)) {
      this.subscribers.set(eventType, []);
    }
    this.subscribers.get(eventType)!.push(callback);
  }

  publish(eventType: string, data: any) {
    const callbacks = this.subscribers.get(eventType);
    if (callbacks) {
      for (const callback of callbacks) {
        callback(data);
      }
    }
  }
}

// 사용 예시
const eventBus = new EventBus();

// 구독자 등록
eventBus.subscribe("message", (data) => {
  console.log(`Subscriber 1 received: ${data}`);
});
```

발행/구독 패턴 구현한 코드 예제 ⇒ https://github.com/addyosmani/pubsubz

🤔 탬플릿 코드/jQuery 코드로 예제를 들고와서,, 잘 안쓰이는 것 같기도 하고,,

리액트에서는 RxJs라이브러리를 소개하는데 이것도 요새는 잘 안쓰이는 것같은데,,

```tsx
import { useEffect, useState } from "react";
import { Subject } from "rxjs";

// 2. Subject 생성 (발행/구독 주체)
const counter$ = new Subject<number>();

// 3. React 컴포넌트
export default function CounterComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const subscription = counter$.subscribe((value) => {
      setCount(value);
    });

    // 언마운트될 때 구독 해제
    return () => {
      subscription.unsubscribe();
    };
  }, []);

  const increment = () => {
    counter$.next(count + 1);
  };

  return (
    <div className="p-4 text-center">
      <h1 className="text-2xl mb-4">RxJS Counter</h1>
      <p className="text-xl mb-4">Count: {count}</p>
      <button
        onClick={increment}
        className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
      >
        Increment
      </button>
    </div>
  );
}
```

## 중재자 패턴

하나의 객체가 이벤트 발생 시 다른 여러 객체들에게 알림을 보낼 수 있는 디자인 패턴

- 하나의 객체가 다른 객체에서 발생한 특정 유형의 이벤트에 대해 알림을 받을 수 있다.

| **항목**               | **중재자 패턴 (Mediator)**                                       | **발행/구독 패턴 (Pub/Sub)**                          |
| ---------------------- | ---------------------------------------------------------------- | ----------------------------------------------------- |
| **상호작용 구조**      | **중재자**가 중앙에서 상호작용을 중재                            | 발행자와 구독자가 **간접적**으로 연결                 |
| **컴포넌트 간 의존성** | 컴포넌트는 중재자만 알며, 서로 직접적으로 알지 않음              | 구독자는 발행자를 모르고, 발행자는 구독자를 알지 못함 |
| **통신 방식**          | **중재자**를 통한 중앙 집중식 통신                               | **브로커**를 통한 이벤트 기반 비동기 통신             |
| **유연성**             | 중재자가 모든 상호작용을 관리하기 때문에 유연성이 떨어질 수 있음 | 구독자와 발행자가 서로 독립적이므로 유연성 있음       |
| **사용 예시**          | GUI의 버튼과 텍스트박스 사이의 상태 관리                         | 이벤트 시스템, 메시징 시스템 등                       |
| **복잡성**             | 중재자가 많은 역할을 하므로 복잡할 수 있음                       | 발행자와 구독자의 관계가 간단하고, 관리하기 용이함    |

```tsx
class Mediator {
  private components: any[] = [];

  addComponent(component: any) {
    this.components.push(component);
  }

  notify(sender: any, event: string) {
    for (const component of this.components) {
      if (component !== sender) {
        component.receive(event);
      }
    }
  }
}

class Component {
  constructor(private mediator: Mediator) {}

  send(event: string) {
    console.log(`${this.constructor.name} sends: ${event}`);
    this.mediator.notify(this, event);
  }

  receive(event: string) {
    console.log(`${this.constructor.name} receives: ${event}`);
  }
}

// 사용 예시
const mediator = new Mediator();
const componentA = new Component(mediator);
const componentB = new Component(mediator);

mediator.addComponent(componentA);
mediator.addComponent(componentB);

componentA.send("Event A");
// 출력:
// Component A sends: Event A
// Component B receives: Event A
```

🤔 실무에서 중재자 패턴으로 사용할 수 있는 곳?

Context API를 활용하여 특정 컴포넌트를 중재하는 역할로 중앙 집중식으로 상태를 관리하는 형태?

## 커맨드 패턴

메서드 호출, 요청 또는 작업을 단일 객체로 캡슐화하여 추후에 실행할 수 있도록 하는 패턴

🤔 ex) 특정 동작이나 UI상태를 객체로 캡슐화하여 명령을 만드는 것?

```tsx
import React, { useState } from "react";

// Command 인터페이스
interface Command {
  execute(): void;
}

// Receiver (구체적인 동작을 수행하는 객체)
class ButtonHandler {
  handleClick(action: string) {
    console.log(`Button clicked: ${action}`);
  }
}

// ConcreteCommand (각 버튼 클릭에 대한 동작)
class ButtonClickCommand implements Command {
  constructor(private handler: ButtonHandler, private action: string) {}

  execute() {
    this.handler.handleClick(this.action);
  }
}

// Invoker (버튼 클릭을 관리하는 컴포넌트)
const ButtonPanel = () => {
  const [message, setMessage] = useState("");

  const handler = new ButtonHandler();

  const handleButtonClick = (action: string) => {
    const command = new ButtonClickCommand(handler, action);
    command.execute();
    setMessage(`Button clicked: ${action}`);
  };

  return (
    <div>
      <button onClick={() => handleButtonClick("Start")}>Start</button>
      <button onClick={() => handleButtonClick("Pause")}>Pause</button>
      <button onClick={() => handleButtonClick("Stop")}>Stop</button>
      <p>{message}</p>
    </div>
  );
};

export default ButtonPanel;
```

- useReducer로 바꿔도 가능할 듯 합니다.
