## 09. 비동기 프로그래밍 패턴

> #### [들어가며]
>
> Promise, async/await 등이 자바스크립트 개념은 코드를 깔끔하고 읽기 쉽게 만들어줍니다.
> async 함수는 최근 ES7의 일부로 도입되었으며 이러한 기능을 활용해 애플리케이션의 흐름을 구성하는 몇 가지 패턴에 대해 살펴보겠습니다.

### 09-01. 비동기 프로그래밍

- 자바스크립트에서 동기 코드는 블로킹 방식으로 실행됩니다. === 현재 문장의 코드가 실행이 완료된 이후에만 다음 코드가 실행될 수 있음 === 앞선 작업을 기다려야 하는 블로킹 현상 발생
- 반면 비동기 코드는 논블로킹 방식으로 실행됩니다.

  - 자바스크립트 엔진은 현재 실행중인 코드가 다른 작업을 기다리는 동안 백그라운드에서 해당 비동기 코드를 실행 === 이벤트 루프 메커니즘 생각

- 비동기 로직 작성시 async/await, Promise 와 같은 문법을 통해서 비동기 코드를 마치 동기 코드처럼 읽히도록 작성할 수 있어 코드의 가독성과 이해도를 개선할 수 있습니다.

### 09-02. 배경

- 자바스크립트에서 콜백함수는 다른 함수에 인수로 전달되어, 비동기 작업이 완료된 후 실행됨.
- 콜백을 사용할 때 주요 단점 중 하나는 `Callback Hell`으로 불리는 상황을 초래할 수 있다는 것.
  - 중첩된 콜백 구조로 인해 코드의 가독성과 유지보수성이 크게 저하되는 상황을 뜻함
    > - TMI: 들여쓰기가 기본 4로 되어있는 저희 소스코드도 가독성이 크게 저하되어있는 상황입니다. tkffuwntpdy...

### 09-03. 프로미스 패턴

- Promise란 책에서는 최신 방법이라고만 설명하지만 사실 비동기 처리 상태와 처리 결과를 관리하는 객체이죠
  - Promise 생성자 함수는 성공과 실패 시 실행할 resolve, reject함수를 인수로 전달받고, 생성된 Promise는 pending, fulfilled, rejected의 상태를 가지며 결국에는 pending이 아닌 상태로 settled됩니다...!

#### 09-03.1. 프로미스 체이닝

- 프로미스 체이닝 패턴을 사용하면 여러 개의 프로미스를 함께 연결하여 보다 복잡한 비동기 로직을 만들 수 있습니다.

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

interface Post {
  id: number;
  title: string;
  userId: number;
}

// 간단한 API 함수들
function fetchUser(userId: number): Promise<User> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        id: userId,
        name: `User ${userId}`,
        email: `user${userId}@example.com`,
      });
    }, 1000);
  });
}

function fetchUserPosts(userId: number): Promise<Post[]> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, title: "First Post", userId },
        { id: 2, title: "Second Post", userId },
      ]);
    }, 800);
  });
}

function fetchPostComments(postId: number): Promise<string[]> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([`Comment 1 on post ${postId}`, `Comment 2 on post ${postId}`]);
    }, 500);
  });
}

// 프로미스 체이닝 예시
function getUserWithPosts(userId: number) {
  let userData: User;

  return fetchUser(userId)
    .then((user) => {
      console.log("1. 사용자 정보 가져옴:", user.name);
      userData = user;
      return fetchUserPosts(user.id);
    })
    .then((posts) => {
      console.log("2. 게시글 가져옴:", posts.length + "개");
      // 첫 번째 게시글의 댓글 가져오기
      if (posts.length > 0) {
        return fetchPostComments(posts[0].id);
      }
      return [];
    })
    .then((comments) => {
      console.log("3. 댓글 가져옴:", comments.length + "개");
      return {
        user: userData,
        firstPostComments: comments,
      };
    })
    .catch((error) => {
      console.error("오류 발생:", error);
      throw error;
    });
}

// 사용
getUserWithPosts(123).then((result) => {
  console.log("최종 결과:", result);
});

// 사실 저는 then을 그닥 잘 사용하지는 않는것 같습니다. 🙂
```

#### 09-03.5. 프로미스 메모이제이션

- 프로미스 메모이제이션 패턴은 캐시를 사용하여 프로미스 함수의 호출 결과 값을 저장합니다. 이를 통해 중복된 요청을 방지할 수 있습니다.

```ts
// 기본적인 메모이제이션 구현
function createPromiseMemo<T>() {
  const cache = new Map<string, Promise<T>>();

  return function memoizedFetch(
    key: string,
    fetcher: () => Promise<T>
  ): Promise<T> {
    // 이미 진행 중이거나 완료된 Promise가 있으면 재사용
    if (cache.has(key)) {
      return cache.get(key)!;
    }

    // 새로운 Promise 생성 및 캐시에 저장
    const promise = fetcher().catch((error) => {
      // 실패한 Promise는 캐시에서 제거 (재시도 가능하게)
      cache.delete(key);
      throw error;
    });

    cache.set(key, promise);
    return promise;
  };
}

// 사용 예시
const memoizedFetch = createPromiseMemo<any>();

function fetchUser(userId: number): Promise<any> {
  return new Promise((resolve) => {
    console.log(`실제 API 호출: User ${userId}`);
    setTimeout(() => {
      resolve({ id: userId, name: `User ${userId}` });
    }, 1000);
  });
}

// 메모이제이션 적용
function getUserMemoized(userId: number) {
  return memoizedFetch(`user-${userId}`, () => fetchUser(userId));
}

// 테스트
console.log("첫 번째 호출");
getUserMemoized(1).then(console.log);

console.log("두 번째 호출 (즉시)");
getUserMemoized(1).then(console.log); // 캐시된 Promise 재사용

console.log("세 번째 호출 (즉시)");
getUserMemoized(1).then(console.log); // 캐시된 Promise 재사용
```

- 이미 요청한 내용이 캐시에 존재한다면 캐시된 데이터가 반환되는 형식인듯

#### 09-03.7. 프로미스 재시도

- tanstack query 설정에서 retry 0 으로 설정하자고 건의했다가 기각당했는데... 재시도 활용하시는분이 계신가요...?
  console.log만 어지러워지는 너낌입니다

#### 09-03.8. 프로미스 데코레이터

> 까먹어서 다시써보는 데코레이터
> 데코레이터는 함수나 클래스를 감싸서 추가 기능을 제공하는 패턴입니다. 원본 코드를 수정하지 않고 기능을 확장할 수 있습니다.

- 프로미스 데코레이터 또한 프로미스 함수를 감싸는 데코레이터가 존재하는것으로 보임.
- 프로미스 데코레이터 패턴은 고차 함수를 사용하여 프로미스에 적용할 수 있는 데코레이터를 생성합니다. 이를 통해 프로미스에 추가적인 기능을 부여할 수 있습니다. === 복잡한 로직들을 추상화하고 또 추가적인 기능으로 연동해주는 개념인듯 함

##### 사용 예시

- 재시도 데코레이터
- 타임아웃 데코레이터
- 캐싱 데코레이터

### 09-04. async/await 패턴

- async/await은 프로미스 기반으로 구축되었습니다.
- await 키워드는 fetch 호출이 완료될 때까지 함수 실행을 일시 중지하게 됩니다.
- 제너레이터에 대한 개념도 존재하는것으로 알고있지만 일단 패스하겠습니다.

#### 09-04.2. 비동기 반복

- 비동기 반복 (async Iteration) 패턴은 for-await-of 반복문을 사용하여 비동기 반복 가능 객체를 순회할 수 있도록 합니다

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

// hello iteration object..!
```

#### 비동기 병렬 혹은 순차 실행

- 최근 생각나는 관련 처리는 토이프로젝트에서 React router v7 에서 새로 도입?된 loader를 사용해서 로그인 상태에 먼저 확인 후 필요한 페이지를 병렬처리로 load 해봤던 기억이 있네요

#### async/await 데코레이터

- 고차함수가 언급되어서 차이점이 존재하는줄 알았지만, Promise 데코레이터와 동일한 개념이고 문법만 변경된 형태라고 합니다 :)
