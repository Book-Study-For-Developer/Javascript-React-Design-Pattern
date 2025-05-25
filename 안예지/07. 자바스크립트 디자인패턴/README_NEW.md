## 7.17 행위 패턴

행위 패턴은 객체 간의 상호 작용과 책임 분배에 관한 패턴입니다. 주요 행위 패턴으로는 다음과 같은 것들이 있습니다.

- **관찰자 패턴**: 객체 간 일대다 의존성을 정의하여 한 객체의 상태가 변경되면 다른 객체들에게 자동으로 알립니다.
- **중재자 패턴**: 객체 간의 복잡한 통신을 캡슐화하여 객체들이 서로 직접 참조하지 않고도 상호작용할 수 있게 합니다.
- **커맨드 패턴**: 요청을 객체로 캡슐화하여 매개변수화된 클라이언트를 큐에 넣고, 요청에 대한 작업을 지원합니다.
- **전략 패턴**: 알고리즘군을 정의하고 각각을 캡슐화하여 교체 가능하게 만듭니다.
- **상태 패턴**: 객체의 내부 상태가 바뀜에 따라 객체의 행동을 바꿀 수 있게 합니다.

이러한 행위 패턴들은 객체 간의 의사소통과 협력을 구조화하는 방법을 제공하며, 각각의 패턴은 특정한 문제 상황에서 유용하게 활용될 수 있습니다.

## 7.18 관찰자 패턴
한 객체가 변경 될 때 다른 객체들에 변경되었음을 알릴 수 있게 해주는 패턴
누가 자신을 구독하는지 알 필요 없이 알림을 보낼 수 있습니다.
한 객체를 관찰하는 여러 객체들이 존재하며, 주체의 상태가 변화하면 관찰자들에게 자동으로 알림을 보냅니다.
ES2015+ 에서는 notify와 update 메서드를 사용하는 자바스크립트 클래스르 통해 주체와 관찰자르 구현함으로써 관찰자 패턴을 만들 수 있습니다.

관찰자 패턴의 구성 요소
- 주체(Subject): 관찰 대상이 되는 객체로, 관찰자들의 목록을 관리하고 상태 변화가 있을 때 모든 관찰자에게 알림을 보냅니다.
- 관찰자(Observer): 주체의 상태 변화를 감지하고 반응하는 인터페이스를 정의합니다.
- 구체적 주체(Concrete Subject): 주체 인터페이스를 구현한 실제 클래스로, 관찰 대상이 되는 상태와 데이터를 포함합니다.
- 구체적 관찰자(Concrete Observer): 관찰자 인터페이스를 구현한 실제 클래스로, 주체의 상태 변화에 따라 특정 동작을 수행합니다.

#### 관찰자 패턴 예제 코드

```javascript 
// 인터페이스 정의
interface Observer { update(data: any): void; }
interface Subject {
  subscribe(observer: Observer): void;
  unsubscribe(observer: Observer): void;
  notify(data: any): void;
}

// 뉴스 발행자 구현
class NewsPublisher implements Subject {
  private observers: Observer[] = [];
  private latestNews = '';

  subscribe(observer: Observer): void {
    if (!this.observers.includes(observer)) this.observers.push(observer);
  }

  unsubscribe(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index !== -1) this.observers.splice(index, 1);
  }

  notify(data: any): void {
    this.observers.forEach(observer => observer.update(data));
  }

  publishNews(news: string): void {
    this.latestNews = news;
    console.log(`새 뉴스: ${news}`);
    this.notify(news);
  }

  getLatestNews(): string { return this.latestNews; }
}

// 구독자 구현
class EmailSubscriber implements Observer {
  constructor(private email: string) {}
  update(news: string): void {
    console.log(`${this.email}로 전송: ${news}`);
  }
}

class MobileSubscriber implements Observer {
  constructor(private userId: string) {}
  update(news: string): void {
    console.log(`사용자 ${this.userId}에게 알림: ${news}`);
  }
}

// 사용 예시
const publisher = new NewsPublisher();
const email1 = new EmailSubscriber("user1@example.com");
const email2 = new EmailSubscriber("user2@example.com");
const mobile = new MobileSubscriber("user_123");

// 구독 등록
publisher.subscribe(email1);
publisher.subscribe(email2);
publisher.subscribe(mobile);

// 뉴스 발행
publisher.publishNews("관찰자 패턴 대세!");

// 구독 해지
publisher.unsubscribe(email2);

// 새 뉴스 발행
publisher.publishNews("구독 해지한 사용자는 알림 받지 않음");
```

### 7.19 발행/구독 패턴
실제 자바스크립트 환경에서는 발행/구독 패턴이라는 변형된 형태의 구현이 더 널리 사용됨. 관찰자 패턴과 유사하지만 차이점 존재함.
발행/구독 패턴에서는 이벤트 알림을 원하는 구독자와 이벤트를 발생시키는 발행자 사이에 토픽/이벤트 채널을 둠. 이러한 이벤트 시스템을 통해 애플리케이션에 특화된 이벤트 정의가 가능하고 구독자에게 필요한 값이 포함된 커스텀 인자 전달 가능함.

#### 발행/구독 패턴의 상세 설명

발행/구독(Pub/Sub) 패턴은 관찰자 패턴의 확장된 형태로 볼 수 있으며, 다음과 같은 특징을 가집니다.

1. **이벤트 채널의 존재**: 발행자와 구독자 사이에 이벤트 채널(또는 이벤트 버스, 메시지 브로커)이 있어 직접적인 의존성이 없습니다.
2. **느슨한 결합**: 발행자는 구독자에 대해 알 필요가 없고, 구독자도 발행자를 알 필요가 없습니다.
3. **다중 이벤트 타입**: 여러 종류의 이벤트(토픽)을 정의하고 구독자는 관심 있는 이벤트만 선택적으로 구독할 수 있습니다.
4. **비동기적 통신**: 대부분의 발행/구독 시스템은 비동기 방식으로 메시지를 전달합니다.

#### 관찰자 패턴과 발행/구독 패턴의 차이점

| 특성 | 관찰자 패턴 | 발행/구독 패턴 |
|------|------------|--------------|
| 결합도 | 주체가 관찰자의 존재를 알고 있음 (직접 알림) | 발행자와 구독자는 서로를 모름 (간접 알림) |
| 중간 레이어 | 없음 (주체가 직접 관찰자에게 알림) | 이벤트 채널/브로커 존재 |
| 통지 방식 | 모든 관찰자에게 동일한 알림 전송 | 토픽/이벤트 타입에 따라 선택적 구독 가능 |
| 확장성 | 주체-관찰자 간 1:n 관계 | m:n 관계 가능 (여러 발행자, 여러 구독자) |
| 구현 복잡성 | 상대적으로 단순 | 중간 채널 구현으로 인해 더 복잡 |

#### 자바스크립트에서의 발행/구독 패턴 예제
```javascript
// 간단한 발행/구독 시스템
class EventBus {
  constructor() {
    this.topics = {};
  }

  // 이벤트 구독
  subscribe(topic, callback) {
    if (!this.topics[topic]) this.topics[topic] = [];
    const index = this.topics[topic].push(callback) - 1;
    
    // 구독 취소 함수 반환
    return {
      unsubscribe: () => {
        this.topics[topic].splice(index, 1);
        if (this.topics[topic].length === 0) delete this.topics[topic];
      }
    };
  }

  // 이벤트 발행
  publish(topic, data) {
    if (!this.topics[topic]?.length) return;
    this.topics[topic].forEach(callback => setTimeout(() => callback(data), 0));
  }
  
  // 토픽 정보 조회
  getTopics() {
    return Object.keys(this.topics).map(topic => ({
      topic,
      count: this.topics[topic].length
    }));
  }
}

// 사용 예시
const bus = new EventBus();

// 구독 등록
const userSub = bus.subscribe('user:login', user => 
  console.log(`로그인: ${user.name}`));

bus.subscribe('notification', msg => 
  console.log(`알림: ${msg.message}`));

bus.subscribe('user:login', user => 
  console.log(`로그인 시간: ${new Date().toLocaleTimeString()}`));

// 발행
bus.publish('user:login', { name: '홍길동', id: 1234 });
bus.publish('notification', { message: '새 메시지 도착', id: 5678 });

// 구독 취소
userSub.unsubscribe();

// 취소 후 발행 (첫 번째 콜백은 실행되지 않음)
bus.publish('user:login', { name: '김철수', id: 5678 });

console.log('활성 토픽:', bus.getTopics());
```

#### 실무 사례: 액세스 토큰 갱신을 위한 발행/구독 패턴 📝

실무에서 Axios interceptor를 활용해 토큰 갱신 처리 구현 시 발행/구독 패턴을 적용했던 사례가 있어서 들고와봤습니다.

```typescript
// 핵심 발행/구독 패턴 구현 부분
// 1. 구독자 목록 관리
let isRefreshing = false;
let refreshSubscribers: ((token: string) => void)[] = []; 

// 2. 발행자: 토큰 갱신 결과를 모든 구독자에게 알림
const onRefreshed = (token: string) => {
  refreshSubscribers.forEach((callback) => callback(token));
  refreshSubscribers = [];
};

// 3. 구독자 등록 함수
const addRefreshSubscriber = (callback: (token: string) => void) => {
  refreshSubscribers.push(callback);
};

// 응답 인터셉터 내부 구현
async (error: AxiosError) => {
  // 401 인증 오류 처리
  if (error.response?.status === 401) {
    const originalRequest = error.config as any;
    
    // 이미 토큰 갱신 진행 중인 경우: 현재 요청을 구독자로 등록
    if (isRefreshing) {
      return new Promise((resolve, reject) => {
        addRefreshSubscriber((token: string) => {
          if (token) {
            // 새 토큰으로 원래 요청 재시도
            originalRequest.headers.Authorization = `Bearer ${token}`;
            resolve(axios(originalRequest));
          } else {
            // 토큰 갱신 실패 시 요청도 실패 처리
            reject(new Error('토큰 갱신 실패'));
          }
        });
      });
    }
    
    // 첫 번째 401 에러: 토큰 갱신 프로세스 시작
    isRefreshing = true;
    
    try {
      // 토큰 갱신 요청 
      const newToken = await authService.refreshToken();
      
      // 갱신 성공: 새 토큰을 모든 대기 중인 요청에 발행
      onRefreshed(newToken);
      
      // 원래 요청 재시도
      originalRequest.headers.Authorization = `Bearer ${newToken}`;
      return axios(originalRequest);
    } catch (error) {
      // 갱신 실패: 빈 토큰을 발행하여 모든 요청이 실패하도록 함
      onRefreshed("");
      return Promise.reject(error);
    } finally {
      isRefreshing = false;
    }
  }
  
  return Promise.reject(error);
}
```

#### 패턴 설명

위 코드에서 발행/구독 패턴은 다음과 같이 구현되어 있습니다.

1. **발행/구독 메커니즘**
   - `refreshSubscribers` 배열: 토큰 갱신을 기다리는 요청들의 콜백 함수를 저장하는 구독자 목록
   - `addRefreshSubscriber()`: 구독자를 등록하는 함수
   - `onRefreshed()`: 토큰 갱신 결과를 모든 구독자에게 알리는 발행 함수

2. **주요 시나리오**
   - 여러 API 요청이 동시에 401 오류를 받으면 첫 번째 요청만 갱신을 시도하고 나머지는 구독자로 등록
   - 갱신이 성공하면 새 토큰이 모든 구독자에게 전파되어 자동으로 원래 요청을 재시도
   - 갱신이 실패하면 빈 토큰이 전파되어 모든 요청이 실패 처리됨

3. **패턴 사용으로 얻은 효과**
   - Race Condition 방지: 여러 요청이 동시에 401을 받아도 토큰 갱신은 한 번만 수행
   - 자원 효율성: 중복 갱신 요청 없이 단일 갱신으로 모든 요청 처리
   - 우아한 에러 처리: 갱신 결과가 모든 대기 요청에 일관되게 전파됨


### 관찰자 패턴과 발행/구독 패턴의 장단점
장점
- **느슨한 결합**: 컴포넌트 간의 직접적인 의존성을 줄여줍니다.
- **확장성**: 새로운 구독자/관찰자를 기존 코드 수정 없이 쉽게 추가할 수 있습니다.
- **코드 재사용성**: 직접 연결이 아닌 주체와 관찰자의 관계로 결합도를 낮춰 코드의 관리와 재사용성을 높일 수 있습니다.

단점
- **예측 가능성 저하**: 발행자와 구독자의 연결을 분리함으로써, 애플리케이션의 특정 부분들이 기대하는 대로 동작한다는 것을 보장하기 어려워질 수 있습니다.
- **의존성 파악 어려움**: 구독자들이 서로의 존재에 대해 전혀 알 수 없고, 발행자를 변경하는 데 드는 비용을 파악하기 어렵습니다.
- **추적 복잡성**: 구독자와 발행자 관계가 동적으로 결정되기에 어떤 구독자가 어떤 발행자에 의존하는지 추적하기 어려워질 수 있습니다.

### 리액트 생태계에서의 관찰자 패턴
RxJS - 관찰자 패턴 사용하는 대표적인 라이브러리
RxJS(Reactive Extensions for JavaScript)는 관찰자 패턴을 기반으로 한 강력한 반응형 프로그래밍 라이브러리입니다. 이 라이브러리는 비동기 데이터 스트림과 이벤트 기반 프로그래밍을 선언적으로 처리할 수 있게 해줍니다.

**RxJS의 핵심 개념**
- **Observable**: 시간에 따라 발생하는 이벤트나 데이터의 스트림을 나타냅니다.
- **Observer**: Observable을 구독하고 데이터를 받아 처리하는 객체입니다.
- **Subscription**: Observable 구독의 실행을 나타내며, 구독을 취소하는 방법을 제공합니다.
- **Operators**: 함수형 프로그래밍 방식으로 Observable을 변환하는 순수 함수들입니다(map, filter, merge 등).
- **Subject**: Observable이자 Observer 역할을 동시에 할 수 있는 특별한 유형입니다.

**RxJS 활용 예시**
```javascript
import { fromEvent } from 'rxjs';
import { map, debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';

// 검색 입력 처리
const search$ = fromEvent(document.querySelector('#search'), 'input').pipe(
  map(e => e.target.value),
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => fetch(`/api/search?q=${term}`).then(res => res.json()))
);

// 구독
const sub = search$.subscribe(results => {
  renderResults(results);
});

// 정리
function unsubscribe() {
  sub.unsubscribe();
}
```

## 중재자 패턴

### 중재자 패턴의 개념

중재자 패턴(Mediator Pattern)은 객체 간의 복잡한 통신과 제어를 캡슐화하는 행위 패턴입니다. 이 패턴은 객체 간의 직접적인 참조를 제거하고, 중재자라는 중앙 객체를 통해 모든 상호작용이 이루어지도록 합니다.

중재자 패턴의 핵심 아이디어는 "객체들이 서로 직접 통신하지 않고 중재자를 통해 통신한다"는 것입니다. 이를 통해 객체 간의 결합도를 낮추고 코드의 유지보수성을 높일 수 있습니다.

### 중재자 패턴의 구성 요소

1. **중재자(Mediator)**: 객체 간 통신을 조정하는 인터페이스를 정의합니다.
2. **구체적 중재자(Concrete Mediator)**: 중재자 인터페이스를 구현하고 동료 객체 간의 조정 로직을 담당합니다.
3. **동료(Colleague)**: 중재자와 통신하는 객체들의 인터페이스를 정의합니다.
4. **구체적 동료(Concrete Colleague)**: 동료 인터페이스를 구현하고 다른 동료와의 통신이 필요할 때 중재자를 통해 처리합니다.

### 중재자 패턴 예제: 채팅방 기능

다음은 채팅방을 구현한 중재자 패턴의 예제입니다.
```javascript
// 중재자 인터페이스
interface ChatMediator {
  send(message: string, sender: User): void;
  add(user: User): void;
}

// 참여자 클래스
class User {
  constructor(private name: string, private mediator: ChatMediator) {
    this.mediator.add(this);
  }

  getName(): string { return this.name; }

  send(message: string): void {
    console.log(`${this.name} 보냄: ${message}`);
    this.mediator.send(message, this);
  }

  receive(message: string, sender: User): void {
    console.log(`${this.name} 받음: ${message} (발신: ${sender.getName()})`);
  }
}

// 구체적 중재자 구현
class ChatRoom implements ChatMediator {
  private users: User[] = [];

  add(user: User): void {
    this.users.push(user);
    console.log(`${user.getName()} 입장`);
  }

  send(message: string, sender: User): void {
    // 발신자 제외 모든 사용자에게 전달
    this.users
      .filter(user => user !== sender)
      .forEach(user => user.receive(message, sender));
  }
}

// 사용 예시
const room = new ChatRoom();
const user1 = new User('철수', room);
const user2 = new User('영희', room);
const user3 = new User('민수', room);

user1.send('안녕하세요!');
user2.send('반갑습니다!');
```
위 예제에서
- `ChatMediator`는 중재자 인터페이스입니다.
- `ChatRoom`은 구체적 중재자로, 사용자 간의 메시지 전달을 관리합니다.
- `User`는 동료 클래스로, 중재자를 통해 다른 사용자와 통신합니다.

### 중재자 패턴의 장단점

#### 장점
1. **결합도 감소**: 객체 간 직접 참조가 없어 결합도가 낮아집니다.
2. **재사용성 향상**: 동료 객체는 중재자와만 통신하므로 다른 중재자와 함께 재사용하기 쉽습니다.
3. **유지보수성 개선**: 객체 간 통신 로직이 중앙화되어 유지보수가 용이합니다.
4. **단일 책임 원칙**: 통신 로직이 중재자에 집중되어 동료 객체는 자신의 핵심 기능에만 집중할 수 있습니다.

#### 단점
1. **중재자 복잡화**: 시스템이 복잡해질수록 중재자가 복잡해지고 유지보수가 어려워질 수 있습니다.
2. **중앙 집중화**: 중재자가 단일 실패 지점(Single Point of Failure)이 될 수 있습니다.
3. **성능 오버헤드**: 모든 통신이 중재자를 거치므로 직접 통신보다 약간의 성능 저하가 발생할 수 있습니다.

## 커맨드 패턴

### 커맨드 패턴의 개념

커맨드 패턴(Command Pattern)은 요청을 객체로 캡슐화하여 클라이언트와 수신자 간의 의존성을 분리하는 행위 패턴입니다. 이 패턴을 사용하면 요청을 큐에 저장하거나 로그로 기록하고, 작업을 취소하는 등의 기능을 쉽게 구현할 수 있습니다.

커맨드 패턴의 핵심은 "하나의 행동을 객체로 표현"하는 것입니다. 이를 통해 명령을 매개변수화하고, 지연 실행, 명령 큐, 실행 취소(Undo) 등 다양한 기능을 구현할 수 있습니다.

### 커맨드 패턴의 구성 요소

1. **커맨드(Command)**: 작업 수행에 필요한 인터페이스를 선언합니다.
2. **구체적 커맨드(Concrete Command)**: 커맨드 인터페이스를 구현하고 수신자 객체와 작업을 연결합니다.
3. **수신자(Receiver)**: 실제 작업을 수행하는 객체입니다.
4. **호출자(Invoker)**: 커맨드 객체를 저장하고 실행합니다.
5. **클라이언트(Client)**: 구체적 커맨드 객체를 생성하고 수신자를 지정합니다.

### 커맨드 패턴 예제: 리모컨 시스템
다음은 TV와 조명을 제어하는 리모컨 시스템의 예제입니다.

```javascript
// 커맨드 인터페이스
interface Command {
  execute(): void;
  undo(): void;
}

// 수신자 클래스들
class Device {
  protected isOn = false;
  constructor(protected name: string) {}
  
  on(): void {
    this.isOn = true;
    console.log(`${this.name} 켜짐`);
  }
  
  off(): void {
    this.isOn = false;
    console.log(`${this.name} 꺼짐`);
  }
}

class TV extends Device {
  setVolume(level: number): void {
    console.log(`${this.name} 볼륨: ${level}`);
  }
}

class Light extends Device {}

// 커맨드 구현
class PowerCommand implements Command {
  constructor(
    private device: Device,
    private powerOn: boolean
  ) {}
  
  execute(): void {
    this.powerOn ? this.device.on() : this.device.off();
  }
  
  undo(): void {
    this.powerOn ? this.device.off() : this.device.on();
  }
}

// 리모컨 (호출자)
class Remote {
  private commands: Command[][] = [];
  private lastCommand: Command | null = null;
  
  addButton(onCmd: Command, offCmd: Command): number {
    const slot = this.commands.length;
    this.commands.push([onCmd, offCmd]);
    return slot;
  }
  
  press(slot: number, on: boolean): void {
    if (slot >= this.commands.length) return;
    const cmd = this.commands[slot][on ? 0 : 1];
    cmd.execute();
    this.lastCommand = cmd;
  }
  
  undo(): void {
    this.lastCommand?.undo();
  }
}

// 사용 예시
const tv = new TV("거실TV");
const light = new Light("주방등");

const remote = new Remote();

// 버튼 등록
const tvBtn = remote.addButton(
  new PowerCommand(tv, true),
  new PowerCommand(tv, false)
);

const lightBtn = remote.addButton(
  new PowerCommand(light, true), 
  new PowerCommand(light, false)
);

// 리모컨 사용
remote.press(tvBtn, true);     // 거실TV 켜짐
remote.press(lightBtn, true);  // 주방등 켜짐
remote.undo();                 // 주방등 꺼짐
remote.press(tvBtn, false);    // 거실TV 꺼짐
```

### 커맨드 패턴의 장단점

#### 장점
1. **단일 책임 원칙**: 작업을 수행하는 객체와 작업을 호출하는 객체를 분리합니다.
2. **개방/폐쇄 원칙**: 기존 코드를 수정하지 않고 새로운 커맨드를 추가할 수 있습니다.
3. **작업 취소(Undo)**: 이전 상태를 저장하면 작업 취소 기능을 쉽게 구현할 수 있습니다.
4. **작업 큐 및 로그**: 명령을 객체로 저장하여 큐에 넣거나 로그로 기록할 수 있습니다.
5. **트랜잭션**: 여러 명령을 하나의 트랜잭션으로 묶어 원자적으로 실행하거나 취소할 수 있습니다.

#### 단점
1. **코드 복잡성 증가**: 간단한 작업에도 여러 클래스가 필요하여 코드가 복잡해질 수 있습니다.
2. **클래스 수 증가**: 각 명령마다 별도의 클래스가 필요하여 클래스 수가 많아질 수 있습니다.
