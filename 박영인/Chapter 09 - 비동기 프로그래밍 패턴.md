### Promise
JavaScript의 **Promise**는 **비동기 작업의 성공 또는 실패 상태를 나타내는 객체**이다.
콜백 지옥을 피하고 **비동기 흐름을 더 직관적이고 예측 가능하게 관리**하기 위해 도입되었다.

예전에는 비동기 작업을 처리할 때 콜백 함수 안에 콜백 함수를 중첩해서 작성하는 방식이 일반적이었는데, 이는 코드의 가독성을 떨어뜨리고 유지보수를 어렵게 만들었다. Promise는 이러한 문제를 해결하고, 이후 등장한 async/await의 기반이 되는 개념이기도 하다.

#### 상태

| **상태**    | **설명**               |
| --------- | -------------------- |
| pending   | 대기 상태, 아직 결과가 없음     |
| fulfilled | 작업 성공, resolve() 호출됨 |
| rejected  | 작업 실패, reject() 호출됨  |

#### 예시
```js
const promise = new Promise((resolve, reject) => {
  const success = true;
  
  if (success) {
    resolve('작업 성공');
  } else {
    reject(new Error('작업 실패'));
  }
});
```
비동기 작업의 결과에 따라 resolve() 또는 reject()가 호출된다. 이후 .then()과 .catch()를 사용하여 결과를 처리할 수 있다.

![[Pasted image 20250526235935.png]]

Promise는 **Microtask Queue**에서 처리된다.
JavaScript는 단일 스레드 기반으로 동작하지만, 이벤트 루프를 통해 비동기 작업을 처리한다. 
이때, setTimeout, DOM 이벤트 등은 일반 **Task Queue**에 들어가고, Promise, MutationObserver 같은 작업은 **Microtask Queue**에 들어간다.

> [!좀 뜬금없는 TMI]
    > 이벤트 루프랑 비동기 작업 큐 이미지를 보니까, 전에 자동 스크롤 기능을 구현하다가 처음 requestAnimationFrame을 써봤던 기억이 떠오르네요..!
    > 그때 처음으로 Animation Frames Queue의 존재를 알게 되었습니다.. 😳😳
    > 그전엔 그냥 Microtask Queue랑 Task Queue만 있는 줄 알았는데,
    > 프레임 주기에 맞춰 일정 간격으로 렌더링을 해주는 또 다른 큐가 있다는 걸 그때 처음 알게 됐네요🥲🥲
    > 
    > 
    > 그때 참고했던 requestAnimationFrame 관련 글인데요,
    > 혹시 저처럼 처음 접하시는 분이 계시면 도움이 될 것 같아서 함께 공유드려요!
    > https://inpa.tistory.com/entry/%F0%9F%8C%90-requestAnimationFrame-%EA%B0%80%EC%9D%B4%EB%93%9C

### 비동기 병렬
#### Promise.all
```js
const ids = [1, 2, 3];

const fetchUser = async (id) => {
  const res = await fetch(`/user/${id}`);
  return await res.json();
};

const getUsers = async () => {
  try {
    const users = await Promise.all(ids.map(fetchUser));
    console.log(users); // 모든 유저 정보 배열
  } catch (e) {
    console.error('하나라도 실패하면 전체 실패이다.', e);
  }
};
```

- **여러 Promise가 모두 성공했을 때만 결과를 반환**한다.
- 하나라도 실패하면 전체가 실패로 처리되고 catch로 넘어간다. 이후 프로미스들의 결과는 무시된다.
- **모두 성공해야만 의미가 있는 작업**에 적합하다.
- 사용예시: 여러 사용자 정보 한번에 가져오기

#### Promise.allSettled
```js
const urls = [
  '/img/a.jpg',
  '/img/b.jpg',
  '/img/c.jpg'
];

const loadImage = url => fetch(url);

async function loadAllImages() {
  const results = await Promise.allSettled(urls.map(loadImage));

  results.forEach((result, i) => {
    if (result.status === 'fulfilled') {
      showImage(urls[i]);
    } else {
      showPlaceholderImage(); // 실패한 이미지 자리는 기본 이미지로
    }
  });
}

loadAllImages();
```

- **성공하든 실패하든 모든 Promise의 결과가 나올 때까지 기다린다.**
- status: 'fulfilled' | 'rejected' 형식으로 각각의 결과를 확인할 수 있다.
- 실패한 것만 골라 재요청하거나, **일부 실패해도 가능한 것만 보여주고 싶을 때** 유용하다.
- 사용 예시: 썸네일 리스트 로드 (실패는 대체 이미지 처리)

#### Promise.race
```js
const fetchWithTimeout = (url, timeoutMs = 3000) => {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('⏰ 요청 시간이 초과되었습니다')), timeoutMs)
  );

  return Promise.race([
    fetch(url),
    timeout
  ]);
};

fetchWithTimeout('/api/data')
  .then(res => res.json())
  .then(console.log)
  .catch(console.error);
```

- **여러 Promise 중 가장 먼저 끝난 하나의 결과만 반환**한다.
- 성공이든 실패든 상관없이, 먼저 settle된 Promise의 결과로 전체가 결정된다.
- 하나라도 너무 느릴 경우, 먼저 응답 온 쪽으로 빠르게 결정해주는 데 적합하다.
- 사용예시: 타임아웃 처리

#### Promise.any
```js
Promise.any([
  getFromGPT(),
  getFromClaude(),
  getFromGemini()
])
  .then(answer => showAnswer(answer))
  .catch(() => showError('죄송합니다, 모두 실패했어요.'));
```

- **성공한 것 중 가장 먼저 resolve된 값 하나만 반환**한다.
- 모든 요청이 실패할 경우에만 reject된다.
- 대체안을 제공하여 빠르고 유연한 사용자 경험 제공이 필요할 때 적합하다.
- 사용 예시: 여러 AI 응답 중 빠른 쪽 보여주기

### 비동기 반복
```js
async function* createAsyncIterable() {
  yield 1;
  yield 2;
  yield 3;
}

async function main() {
  for await (const value of createAsyncIterable()) {
    console.log(value);
  }
}
```
for await...of는 **Async Iterable 객체를 순회**할 때 사용하는 반복문이다.
async function* 로 정의된 제너레이터는 Promise를 반환하며, 하나씩 await하며 처리할 수 있다.

사용 예시: 실시간 로그 같은 스트리밍 데이터 처리


### 비동기 메모이제이션
```js
const cache = new Map();

async function memoizedMakeRequest(url) {
  if (cache.has(url)) {
    return cache.get(url);
  }

  const response = await fetch(url);
  const data = await response.json();

  cache.set(url, data);
  return data;
}
```
네트워크 요청이나 시간이 오래 걸리는 비동기 작업의 결과를 **캐시에 저장해두고, 동일한 요청이 들어오면 다시 처리하지 않는 방식**이다.

### 비동기 재시도
```js
async function makeRequestWithRetry(url) {
  let attempts = 0;

  while (attempts < 3) {
    try {
      const response = await fetch(url);
      const data = await response.json();
      return data; // 성공하면 이대로 종료!
    } catch (error) {
      attempts++;
      console.log(`Retrying request: attempt ${attempts}`);
    }
  }

  throw new Error("Request failed after 3 attempts.");
}
```
네트워크 환경이 불안정할 때 요청을 **몇 번까지 재시도**할 것인지 정해두고 자동으로 재시도하게 하는 패턴이다.

tanstack query의 retry!?
https://github.com/TanStack/query/blob/main/packages/query-core/src/retryer.ts#L139

### 데코레이터
```js
function asyncLogger(fn) {
  return async function (...args) {
    console.log("Starting async function...");
    const result = await fn(...args);
    console.log("Async function completed.");
    return result;
  };
}

@asyncLogger
async function main() {
  const data = await makeRequest("http://example.com/");
  console.log(data);
}
```
함수를 감싸는 **고차 함수(Higher-Order Function)** 로, 호출 전후에 공통 로직을 삽입할 수 있다.

사용 예시: 함수 실행 로깅, 시간 측정, 에러 추적 등

