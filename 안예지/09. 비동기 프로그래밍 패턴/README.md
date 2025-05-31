# 비동기 프로그래밍 패턴

## 개요
비동기 프로그래밍은 JavaScript에서 매우 중요한 개념입니다. 2016년 ES7에서 `async/await`가 도입되면서 JavaScript의 비동기 프로그래밍이 더욱 강력해졌다고 말할 수 있겠습니다.
비동기 코드는 논블로킹 방식으로 실행되어 백그라운드에서 작업을 처리하며, 메인 스레드의 블로킹을 방지합니다.

## 콜백 함수와 콜백 지옥

### 콜백 함수
콜백 함수는 다른 함수에 인자로 전달되어 나중에 실행되는 함수입니다. JavaScript에서 비동기 프로그래밍의 가장 기본적인 형태입니다.

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback('데이터');
  }, 1000);
}

fetchData((data) => {
  console.log(data);
});
```

### 콜백 지옥
중첩된 콜백 함수는 코드의 가독성을 떨어뜨리고 유지보수를 어렵게 만듭니다. 이를 '콜백 지옥'이라고 합니다.

```javascript
fetchUser((user) => {
  fetchUserPosts(user.id, (posts) => {
    fetchPostComments(posts[0].id, (comments) => {
      fetchCommentAuthor(comments[0].authorId, (author) => {
        console.log(author);
        // 콜백 지옥의 시작...
      });
    });
  });
});
```

이러한 콜백 지옥을 해결하기 위해 Promise와 async/await가 도입되었습니다.

## 프로미스 (Promise)

### 기본 개념
Promise는 비동기 작업의 최종 완료(또는 실패)와 그 결과값을 나타내는 객체입니다.

#### Promise 상태
- **pending**: 초기 상태, 이행되거나 거부되지 않은 상태
- **fulfilled**: 연산이 성공적으로 완료된 상태
- **rejected**: 연산이 실패한 상태

```javascript
const promise = new Promise((resolve, reject) => {
  // 비동기 작업 수행
  if (/* 성공 조건 */) {
    resolve('성공 결과');
  } else {
    reject(new Error('실패 사유'));
  }
});
```

### 프로미스 체이닝
여러 비동기 작업을 순차적으로 처리할 수 있습니다.

```javascript
fetchUser()
  .then(user => fetchUserPosts(user.id))
  .then(posts => processPosts(posts))
  .catch(error => console.error(error));
```

### 프로미스 파이프라인
프로미스와 함수형 프로그래밍을 결합하여 재사용 가능한 비동기 파이프라인을 구축할 수 있습니다.

```javascript
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

const asyncPipe = (...fns) => x => fns.reduce((p, f) => p.then(f), Promise.resolve(x));
```

## async/await 패턴

### 기본 사용법
async/await를 사용하면 비동기 코드를 동기 코드처럼 작성할 수 있습니다.

```javascript
async function getUserData() {
  try {
    const user = await fetchUser();
    const posts = await fetchUserPosts(user.id);
    return posts;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}
```

### 비동기 반복
`for await...of`를 사용하여 비동기 이터러블을 순회할 수 있습니다.

```javascript
async function processItems(items) {
  for await (const item of items) {
    await processItem(item);
  }
}
```

### 비동기 병렬 처리
`Promise.all()`을 사용하여 여러 비동기 작업을 동시에 실행할 수 있습니다.

```javascript
async function fetchAllData() {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);
  return { users, posts, comments };
}
```

### 비동기 메모이제이션
동일한 인자로 호출되는 비동기 함수의 결과를 캐시하여 성능을 최적화할 수 있습니다.

```javascript
function memoizeAsync(fn) {
  const cache = new Map();
  
  return async (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    
    const result = await fn(...args);
    cache.set(key, result);
    return result;
  };
}
```

### 비동기 재시도 패턴
네트워크 요청 등이 실패했을 때 자동으로 재시도하는 패턴입니다.

```javascript
async function retryAsync(fn, retries = 3, delay = 1000) {
  try {
    return await fn();
  } catch (error) {
    if (retries === 0) throw error;
    await new Promise(resolve => setTimeout(resolve, delay));
    return retryAsync(fn, retries - 1, delay * 2);
  }
}
```

### Promise API 비교

#### Promise.all
모든 프로미스가 이행될 때까지 기다립니다. 하나라도 실패하면 전체가 실패합니다.

```javascript
const results = await Promise.all([
  fetch('/api/users'),
  fetch('/api/posts'),
  fetch('/api/comments')
]);
```

#### Promise.allSettled
모든 프로미스의 처리가 완료될 때까지 기다리며, 성공/실패 여부와 관계없이 모든 결과를 반환합니다.

```javascript
const results = await Promise.allSettled([
  fetch('/api/users'),
  fetch('/api/posts'),
  fetch('/api/comments')
]);
```

#### Promise.race
가장 먼저 처리되는 프로미스의 결과를 반환합니다.

```javascript
const result = await Promise.race([
  fetch('/api/fast'),
  fetch('/api/slow')
]);
```

#### Promise.any
성공하는 첫 번째 프로미스의 결과를 반환합니다.

```javascript
const result = await Promise.any([
  fetch('/api/endpoint1'),
  fetch('/api/endpoint2')
]);
```

## 실용적인 예제

### 기본적인 비동기 패턴 예제

#### 1. HTTP 요청 보내기
```javascript
async function fetchWithRetry(url, options = {}, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url, options);
      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      return await response.json();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
}
```

### 2. 파일 시스템에서 파일 읽어오기
```javascript
const fs = require('fs').promises;

async function readFileContent(filePath) {
  try {
    const content = await fs.readFile(filePath, 'utf-8');
    return content;
  } catch (error) {
    console.error(`파일 읽기 실패: ${error.message}`);
    throw error;
  }
}
```

### 3. 파일 시스템에 파일 쓰기
```javascript
const fs = require('fs').promises;

async function writeFileContent(filePath, content) {
  try {
    await fs.writeFile(filePath, content, 'utf-8');
    console.log('파일 쓰기 완료');
  } catch (error) {
    console.error(`파일 쓰기 실패: ${error.message}`);
    throw error;
  }
}
```

### 4. 여러 비동기 함수 한 번에 실행하기
```javascript
async function fetchMultipleResources(urls) {
  try {
    const results = await Promise.all(
      urls.map(url => fetch(url).then(res => res.json()))
    );
    return results;
  } catch (error) {
    console.error('하나 이상의 요청이 실패했습니다:', error);
    throw error;
  }
}
```

### 5. 함수의 결과를 캐싱하기
```javascript
function createAsyncCache() {
  const cache = new Map();
  
  return {
    async get(key, asyncFn) {
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = await asyncFn();
      cache.set(key, result);
      return result;
    },
    clear() {
      cache.clear();
    }
  };
}
```

### 6. async/await로 이벤트 처리하기
```javascript
function waitForEvent(element, eventName) {
  return new Promise(resolve => {
    function handler(event) {
      element.removeEventListener(eventName, handler);
      resolve(event);
    }
    element.addEventListener(eventName, handler);
  });
}

// 사용 예시
async function handleFormSubmit(form) {
  const submitEvent = await waitForEvent(form, 'submit');
  submitEvent.preventDefault();
  // 폼 처리 로직
}
```

### 7. 비동기 함수 실패 시 자동 재시도
```javascript
async function withRetry(fn, options = {}) {
  const {
    retries = 3,
    delay = 1000,
    backoff = 2
  } = options;

  let lastError;

  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      if (i < retries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, delay * Math.pow(backoff, i))
        );
      }
    }
  }

  throw lastError;
}
```

### 8. async/await 데코레이터 작성하기
```javascript
function asyncDecorator(target, key, descriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = async function (...args) {
    console.time(key);
    try {
      const result = await originalMethod.apply(this, args);
      console.timeEnd(key);
      return result;
    } catch (error) {
      console.error(`Error in ${key}:`, error);
      throw error;
    }
  };

  return descriptor;
}

// 사용 예시
class API {
  @asyncDecorator
  async fetchData() {
    // 비동기 작업 수행
  }
}
```

## 고급 비동기 패턴 (with 데코레이터)


### 1. Promise 데코레이터 패턴
```javascript
// Promise 로깅 데코레이터
const withPromiseLogging = (fn) => async (...args) => {
  console.log(`함수 ${fn.name} 실행 시작`);
  try {
    const result = await fn(...args);
    console.log(`함수 ${fn.name} 실행 완료:`, result);
    return result;
  } catch (error) {
    console.error(`함수 ${fn.name} 실행 실패:`, error);
    throw error;
  }
};

// Promise 캐싱 데코레이터
const withPromiseCache = (fn) => {
  const cache = new Map();
  
  return async (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log('캐시된 결과 반환');
      return cache.get(key);
    }
    
    const result = await fn(...args);
    cache.set(key, result);
    return result;
  };
};

// Promise 타임아웃 데코레이터
const withPromiseTimeout = (timeout) => (fn) => async (...args) => {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('시간 초과')), timeout);
  });
  
  return Promise.race([
    fn(...args),
    timeoutPromise
  ]);
};

// 사용 예시
const fetchData = async (url) => {
  const response = await fetch(url);
  return response.json();
};

const enhancedFetchData = withPromiseLogging(
  withPromiseCache(
    withPromiseTimeout(5000)(fetchData)
  )
);
```


### 2. async/await 데코레이터 패턴
```javascript
// 재시도 데코레이터
const withRetry = (retries = 3, delay = 1000) => (fn) => async (...args) => {
  let lastError;
  
  for (let i = 0; i < retries; i++) {
    try {
      return await fn(...args);
    } catch (error) {
      lastError = error;
      if (i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw lastError;
};

// 성능 측정 데코레이터
const withPerformance = (fn) => async (...args) => {
  const start = performance.now();
  try {
    const result = await fn(...args);
    const end = performance.now();
    console.log(`${fn.name} 실행 시간: ${end - start}ms`);
    return result;
  } catch (error) {
    const end = performance.now();
    console.error(`${fn.name} 에러 발생 시간: ${end - start}ms`);
    throw error;
  }
};

// 에러 처리 데코레이터
const withErrorHandling = (errorHandler) => (fn) => async (...args) => {
  try {
    return await fn(...args);
  } catch (error) {
    return errorHandler(error);
  }
};

// 컨텍스트 주입 데코레이터
const withContext = (context) => (fn) => async (...args) => {
  return fn.apply(context, args);
};

// 사용 예시
const fetchUserData = async (userId) => {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
};

const enhancedFetchUserData = withPerformance(
  withRetry(3, 1000)(
    withErrorHandling((error) => ({ error: error.message }))(
      fetchUserData
    )
  )
);

// 체이닝 예시
const result = await enhancedFetchUserData(123);
```

이러한 데코레이터 패턴을 사용하면 다음과 같은 이점이 있습니다.

1. 코드 재사용성 향상
2. 관심사의 분리
3. 기능의 조합과 확장이 용이
4. 코드의 가독성과 유지보수성 향상

각 데코레이터는 단일 책임 원칙을 따르며, 필요에 따라 조합하여 사용할 수 있습니다.

## 결론

비동기 프로그래밍은 JavaScript에서 매우 중요한 개념이며, 콜백에서 Promise, async/await까지 발전해왔습니다. 
여기에 데코레이터 패턴을 결합하면 더욱 강력하고 유지보수가 용이한 코드를 작성할 수 있습니다.

주요 포인트
1. 콜백 패턴은 가장 기본적이지만 콜백 지옥의 위험이 있습니다.
2. Promise는 비동기 작업을 체이닝하고 에러 처리를 용이하게 합니다.
3. async/await는 비동기 코드를 동기 코드처럼 작성할 수 있게 해줍니다.
4. 데코레이터 패턴을 활용하면 비동기 코드의 재사용성과 유지보수성을 높일 수 있습니다.

