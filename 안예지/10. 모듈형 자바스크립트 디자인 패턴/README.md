# 모듈형 자바스크립트 디자인 패턴

JavaScript는 원래 작은 스크립트를 브라우저에서 실행하는 언어였지만, 점차 서버 사이드와 대규모 애플리케이션으로 확장되면서 모듈 시스템의 발전이 필요해졌습니다.

CommonJS, AMD, UMD, ESM의 역사적 흐름과 구조를 비교하고 디자인 패턴과 어떤 접점이 있는지 살펴봅시다.

#### 모듈화가 필요한 이유
1. **코드 재사용성**: 동일한 코드를 여러 곳에서 재사용 가능
2. **의존성 관리**: 코드 간의 의존 관계를 명확하게 관리
3. **네임스페이스**: 전역 스코프 오염 방지
4. **코드 캡슐화**: 내부 구현을 숨기고 인터페이스만 노출

## 모듈 시스템의 역사적 발전

### 1. 전역 스코프 & IIFE — 모듈 이전 시대
초기에는 모든 코드가 전역 스코프에 노출되어 충돌이 빈번했습니다.

```javascript
// IIFE: 즉시 실행 함수로 캡슐화
var MyApp = (function(){
    // private 변수
    var privateVal = 42;
    var privateFunction = function() {
        return privateVal * 2;
    };
    
    // public API
    return {
        getVal: () => privateVal,
        compute: () => privateFunction()
    };
})();
```

**장점**
- ✅ 변수 캡슐화 가능
- ✅ 클로저를 통한 private 스코프 생성
- ✅ 모듈 패턴의 기초 구현

**단점**
- ❌ 의존성 관리 어려움
- ❌ 모듈화 구현의 한계
- ❌ 스크립트 로딩 순서 중요

### 2. CommonJS (2009) — 서버 중심
Node.js의 등장과 함께 `require()`와 `module.exports` 기반 CommonJS가 등장했습니다.

```javascript
// math.js
const private = {
    add: (a, b) => a + b,
    multiply: (a, b) => a * b
};

module.exports = {
    sum: private.add,
    product: private.multiply
};

// app.js
const math = require('./math');
console.log(math.sum(2, 3));      // 5
console.log(math.product(2, 3));  // 6
```

**특징**
- ✅ 동기 로딩: 서버 환경에 최적화
- ✅ 간단한 문법
- ✅ Node.js 기본 모듈 시스템
- ✅ 객체만을 모듈로 지원 (함수나 원시값은 직접 export 불가)
- ❌ 브라우저에서는 Browserify 등 번들러 필요
- ❌ 동적 로딩 미지원

### 3. AMD (2011) — 브라우저 중심
브라우저 환경에서의 비동기 로딩을 위해 `define()` 기반 AMD가 등장했습니다.

```javascript
// 모듈 정의
define('mathModule', ['dependency1', 'dependency2'], 
    function(dep1, dep2) {
        // private 변수/함수
        var privateVar = 10;
        
        // public API
        return {
            add: function(a, b) {
                return a + b + privateVar;
            },
            subtract: function(a, b) {
                return a - b;
            }
        };
    }
);

// 모듈 사용
require(['mathModule'], function(math) {
    console.log(math.add(5, 3));  // 18
});
```

**특징**
- ✅ RequireJS를 통한 구현
- ✅ HTTP 요청 병렬 처리
- ✅ 모듈 간 의존성 명확
- ✅ 비동기 로딩 지원
- ✅ 여러 모듈을 하나의 파일로 번들링해서 전송하는 방식(RequireJS optimizer 등) 제공
- ✅ 스크립트 지연 로딩(Lazy Loading) 지원
- ✅ 빌드 없이 브라우저에서 모듈을 직접 로딩 가능
- ❌ 설정이 다소 복잡 - boilerplate 코드가 많음

#### AMD vs CommonJS 스타일 비교

| 구분              | AMD (define)                | CommonJS (require)      |
|-------------------|-----------------------------|-------------------------|
| 주로 사용 환경    | 브라우저                    | Node.js                |
| 의존성 선언       | 선언적 (define(['dep']))    | 명령형 (require('dep')) |
| 로딩 방식         | 비동기                      | 동기                    |

---

## RequireJS 같은 스크립트 로더는 무슨 역할을 했나요?

JavaScript 로더는 여러 개의 JS 파일(모듈)을 필요한 시점에, 정해진 순서로, 비동기적으로 로드해 주는 도구입니다.

과거에는 `<script>` 태그로 하나씩 JS 파일을 로딩했기 때문에
- 순서에 민감했고
- 전역 스코프가 오염됐으며
- 유지보수가 어려웠습니다.

이런 문제를 해결하기 위해 등장한 것이 RequireJS, curl.js 같은 모듈 로더입니다.

### 역할 1: 의존성 관리
예를 들어, jQuery 플러그인은 jQuery가 먼저 로드돼야만 동작합니다.

```javascript
define(['jquery', 'jquery.plugin'], function($) {
  // 둘 다 로드되고 난 뒤 여기 실행됨
});
```
역할: 모듈 간의 의존 관계를 명확하게 선언하고, 올바른 순서로 로딩함

### 역할 2: 비동기 로딩 (퍼포먼스 최적화)
기존 `<script>`는 동기 로딩이라 하나 로딩이 끝나야 다음이 실행됐지만,
RequireJS나 curl.js는 비동기로 병렬로 로드할 수 있어 퍼포먼스에 유리했습니다.

```javascript
require(['jquery', 'lodash', 'app'], function($, _, app) {
  // 모두 비동기 로딩된 후 이 함수 실행됨
});
```

### 역할 3: 모듈화 (코드 구조화)
ESModules이 없던 시절에는 하나의 커다란 JS 파일이 전부였지만,
RequireJS나 curl.js 덕분에 파일 단위로 코드를 분리하고 재사용하기 쉬운 구조를 만들 수 있었습니다.

```javascript
// math.js
define([], function() {
  return {
    add: (a, b) => a + b
  };
});

// app.js
define(['math'], function(math) {
  console.log(math.add(2, 3));
});
```

### RequireJS vs curl.js

| 항목      | RequireJS      | curl.js         |
|-----------|----------------|----------------|
| 출시      | 2010           | 2011           |
| 크기      | 상대적으로 큼  | 훨씬 가벼움    |
| 문법      | AMD 표준 준수  | AMD 기반, 유연성 강조 |
| 커뮤니티  | 큼             | 작음           |
| 상태      | 유지보수 거의 없음 | 유지보수 종료 |

두 로더 모두 AMD 방식을 따르며, 현대 프레임워크 등장 이후 거의 사용되지 않습니다.

### 왜 더 이상 잘 안 쓰이나요?
현대 JS 개발에서는 다음과 같은 도구들이 이를 대체했기 때문입니다.

| 도구      | 설명                                              |
|-----------|---------------------------------------------------|
| Webpack   | JS + CSS + 이미지까지 모듈로 처리하는 번들러      |
| Vite      | 빠르고 현대적인 번들러 + dev server               |
| ESModules | import/export 표준 지원 (브라우저도 지원)         |
| Parcel, Rollup | 다른 번들링 대안들                          |

---

### RequireJS의 CommonJS 스타일 모듈 래퍼(wrapper) 방식

RequireJS는 AMD 로더이지만, CommonJS 스타일 모듈도 특정 조건에서 자동으로 호환시켜주는 "래퍼(wrapper) 방식"을 지원합니다.

래퍼 방식이란?
RequireJS가 `define()`에 의존성 배열이 없을 경우, 내부 코드를 분석해서 CommonJS 스타일인지 감지하고 자동으로 wrapping 해주는 기능입니다.

공식적인 조건:
- 의존성 배열이 명시되어 있지 않거나,
- factory 함수가 최소한 하나 이상의 매개변수를 가질 경우,

RequireJS는 이를 CommonJS 스타일로 판단하고 자동 래핑합니다.

예시 1: CommonJS 스타일 (암묵적 래핑)
```javascript
define(function(require, exports, module) {
  const $ = require('jquery');
  exports.name = 'MyModule';
});
// define()은 의존성 배열이 없음
// factory 함수가 인자를 가지고 있음 (require, exports, module)
// 이 경우 RequireJS는 자동으로 CommonJS 스타일로 인식
```
내부적으로 다음처럼 래핑해서 처리합니.
```javascript
define(['require', 'exports', 'module'], function(require, exports, module) {
  // 원래 코드 실행
});
```

예시 2: AMD 스타일 (명시적 의존성)
```javascript
define(['jquery'], function($) {
  return {
    name: 'MyModule'
  };
});
// 의존성 배열이 있고
// 콜백 함수는 필요한 인자만 선언함
// 이건 명확한 AMD 방식이고, 별도의 wrapping 없음
```

왜 이렇게 작동할까?
이 방식은 Node.js 스타일 CommonJS 모듈을 브라우저에서도 쉽게 사용할 수 있게 하려는 목적입니다.  
즉, 다음과 같은 모듈을 그대로 가져와도 작동하게 만들 수 있습니다.

```javascript
// myModule.js (Node 스타일)
const _ = require('lodash');
module.exports = function () {
  return _.random(1, 10);
};
```
이걸 AMD 방식으로 감싸지 않아도 작동하도록, RequireJS가 자동으로 detect해서 define()에 필요한 인자들을 추가해주는 겁니다.

정리하자면

| 조건                                              | 처리 방식                                                        |
|---------------------------------------------------|------------------------------------------------------------------|
| define에 의존성 배열이 없음 + factory 함수에 매개변수 있음 | → RequireJS가 CommonJS 스타일로 감지하고 ['require', 'exports', 'module']로 자동 래핑 |
| 의존성 배열 명시 + 콜백 인자 일치                  | → AMD 방식으로 처리                                              |
| factory 함수 매개변수 없음                        | → 의존성 없는 AMD 모듈로 처리                                    |

---

### 4. UMD (2011경) — 범용 환경 지원
Universal Module Definition은 다양한 환경(서버사이드/브라우저)을 모두 지원하는 범용 포맷입니다.
UMD는 다양한 환경(브라우저, Node.js, AMD 등)에서 모듈을 호환성 있게 사용할 수 있도록 설계된 방식이기 때문에, 오픈소스 라이브러리나 배포용 유틸리티에서 여전히 많이 활용되고 있습니다.

#### 기본 UMD 패턴
```javascript
(function(root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['dependency'], factory);
    } else if (typeof module === 'object' && module.exports) {
        // CommonJS
        module.exports = factory(require('dependency'));
    } else {
        // Browser globals
        root.myModule = factory(root.dependency);
    }
}(typeof self !== 'undefined' ? self : this, function(dependency) {
    // 모듈 구현
    var privateMethod = function() {};
    
    return {
        publicMethod: function() {
            privateMethod();
        }
    };
}));
```

## 실제 라이브러리 구현과 사용 예시

### Lodash
Lodash의 GitHub 저장소에는 "The Lodash library exported as a UMD module."이라고 명시되어 있습니다. 이는 Lodash가 UMD 형식으로 배포되어 모든 환경에서 사용할 수 있도록 설계되었음을 의미합니다.

**1. CDN 배포 버전의 UMD 구조**
[보러가기 (눈조심)](https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js)
```javascript
/**
 * @license
 * Lodash (Custom Build) <https://lodash.com/>
 * Build: `lodash modularize exports=...`
 * The Lodash library exported as a UMD module.
 */
;(function() {
    // 전역 변수 감지
    var freeExports = typeof exports == 'object' && exports && !exports.nodeType && exports;
    var freeModule = freeExports && typeof module == 'object' && module && !module.nodeType && module;
    var freeGlobal = typeof global == 'object' && global;
    var freeSelf = typeof self == 'object' && self;
    var freeWindow = typeof window == 'object' && window;

    // 루트 객체 결정
    var root = freeGlobal || freeWindow || freeSelf || this;

    // Lodash 팩토리 함수
    (function(root, factory) {
        if (typeof define == 'function' && typeof define.amd == 'object' && define.amd) {
            // AMD
            define([], factory);
        } else if (freeModule) {
            // CommonJS
            (module.exports = factory());
        } else {
            // 브라우저 전역 변수
            root._ = factory();
        }
    }(root, function() {
        // 내부 Lodash 구현
        return {
            // Lodash 메서드들
        };
    }));
}());
```

#### Lodash 버전별 모듈 형식 비교

| 버전 | 모듈 형식 | 용도 | 특징 |
|------|-----------|------|------|
| lodash | UMD | CDN, Node.js (require) | 모든 환경 호환 |
| lodash-es | ESM | ES6 환경 | 트리쉐이킹 가능 |
*트리쉐이킹이 되는 lodash-es를 사용합시다!

**1. UMD 버전 사용 (lodash)**
```javascript
// 브라우저 (CDN)
<script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>
<script>
    _.isEmpty({});  // true
</script>

// Node.js
const _ = require('lodash');
_.isEmpty([]);  // true

// AMD
define(['lodash'], function(_) {
    _.isEmpty('');  // true
});
```

**2. ESM 버전 사용 (lodash-es)**
```javascript
// 트리쉐이킹 가능한 import
import { isEmpty, map } from 'lodash-es';

isEmpty({});  // true
map([1, 2, 3], n => n * 2);  // [2, 4, 6]
```

### 추상 팩토리 패턴과 UMD의 관계

추상 팩토리 패턴은 어떤 객체를 만들 것인지는 고정해두고, 어떻게 만들지는 상황(환경)에 따라 결정하는 패턴입니다.
UMD는 이 패턴을 활용하여 동일한 라이브러리를 브라우저, AMD, CommonJS 환경에 맞춰 포장하고, 각 환경에 맞는 export 방식을 선택적으로 제공합니다.

#### 1. 추상 팩토리 패턴의 UMD 활용
```javascript
// UMD의 추상 팩토리 패턴 구현
(function(root, factory) {
    // 각 환경에 맞는 모듈 시스템 생성
    if (typeof define === 'function' && define.amd) {
        // AMD 환경용 팩토리
        define(['dependency'], factory);
    } else if (typeof module === 'object') {
        // CommonJS 환경용 팩토리
        module.exports = factory(require('dependency'));
    } else {
        // 브라우저 환경용 팩토리
        root.myModule = factory(root.dependency);
    }
}(this, function($) {
    // 실제 모듈 구현
    return {
        doSomething: function() {
            return $('body').width();
        }
    };
}));
```

#### 2. 추상 팩토리 패턴의 특징이 UMD에서 어떻게 활용되는가?

1. **각 실행 환경에 맞춰 다른 방식으로 모듈을 등록하는 로직**
   ```javascript
   // 각 환경에 맞는 구체적인 팩토리 구현
   if (typeof define === 'function') {  // AMD 환경
       define([], factory);
   } else if (typeof module === 'object') {  // Node.js 환경
       module.exports = factory();
   }
   ```

2. **연관된 역할을 가진 모듈 구성 요소들을 함께 생성**
   ```javascript
   // 환경별 의존성 해결 및 모듈 시스템 생성
   function factory(dependencies) {
       // 의존성 처리
       const deps = processDependencies(dependencies);
       
       // 연관된 역할을 가진 모듈 구성 객체들을 함께(일괄) 생성
       return {
           moduleSystem: createModuleSystem(),
           dependencyResolver: createDependencyResolver(deps),
           exportManager: createExportManager()
       };
   }
   ```

3. **인터페이스 추상화**
   ```javascript
   // 환경에 관계없이 동일한 인터페이스 제공
   return {
       publicMethod() { 
           // 환경별 구체적인 구현은 다르지만
           // 동일한 인터페이스 제공
           return privateVar; 
       }
   };
   ```

UMD는 추상 팩토리 패턴을 활용하여
- 각 환경(AMD, CommonJS, 브라우저)에 맞는 모듈 시스템 전체를 생성
- 환경별 의존성 해결 방식 제공
- 일관된 모듈 인터페이스 유지
이러한 특징들을 통해 다양한 실행 환경에서 동일한 코드베이스를 재사용할 수 있게 합니다.

### 5. ESM (2015~) — JavaScript 표준화
ES6부터 도입된 공식 모듈 시스템으로, `import`/`export` 문법을 사용합니다.

```javascript
// mathUtils.mjs
// named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// default export
export default class Calculator {
    static multiply(a, b) {
        return a * b;
    }
}

// app.mjs
import Calculator, { add, subtract } from './mathUtils.mjs';

console.log(add(5, 3));               // 8
console.log(subtract(5, 3));          // 2
console.log(Calculator.multiply(5, 3)); // 15
```

**특징**
- ✅ 트리쉐이킹 지원
- ✅ 브라우저 & Node.js 네이티브 지원
- ✅ 정적 분석 가능
- ✅ 순환 의존성 처리 개선
- ✅ Named exports와 Default export 구분
- ❌ 구형 브라우저 지원 제한

## 모듈 시스템 비교

| 시스템 | 로딩 방식 | 주요 환경 | 의존성 해결 | 번들링 필요 | 비동기 로딩 |
|--------|-----------|-----------|-------------|-------------|-------------|
| CommonJS | 동기 (require) | Node.js | 런타임 | Yes | No |
| AMD | 비동기 (define) | 브라우저 | 런타임 | Optional | Yes |
| UMD | 환경 감지 | 브라우저 + 서버 | 런타임 | Optional | Yes |
| ESM | 정적 (import) | 표준 | 빌드타임 | Optional | Yes |

JavaScript 모듈 시스템은 전역 변수에서 시작하여 CommonJS, AMD를 거쳐 ESM으로 발전해왔습니다. 각 단계는 실행 환경, 성능, 유지보수 측면에서 진화를 거쳤다는 점과 디자인 패턴의 원칙이 녹아있음을 살펴봤습니다.

이처럼 ESM의 등장은 자바스크립트 모듈 시스템의 표준화를 이끌었을 뿐만 아니라, 서버와 클라이언트 모두에서 동일한 문법과 구조로 코드를 작성할 수 있는 환경을 만들어 주었습니다. 이 변화는 곧 '동형 자바스크립트(isomorphic JavaScript, 또는 Universal JavaScript)'라는 새로운 패러다임을 현실화하는 데 큰 역할을 했습니다.

동형 자바스크립트란, 서버와 클라이언트에서 동일한 자바스크립트 코드가 실행되는 구조를 의미합니다. 예를 들어, 날짜 포맷팅, 유효성 검사, API 유틸리티 등과 같은 공통 로직을 별도의 분기 없이 서버와 클라이언트 양쪽에서 그대로 재사용할 수 있습니다. 덕분에 서버에서는 SSR(서버 사이드 렌더링)로 SEO와 초기 렌더링 속도를 개선하고, 클라이언트에서는 동일한 컴포넌트를 Hydration하여 인터랙션을 이어갈 수 있습니다.

이러한 동형 구조는 Next.js와 같은 현대 프레임워크에서 자연스럽게 구현됩니다. Next.js는 ESM 기반의 모듈 시스템을 활용하여, 서버와 클라이언트 모두에서 import/export 문법으로 코드를 불러오고, 필요한 경우 코드 스플리팅이나 트리쉐이킹도 자동으로 처리합니다. 예를 들어, 날짜 포맷팅 유틸리티를 ESM 모듈로 작성하면, 서버의 getServerSideProps 함수와 클라이언트 컴포넌트 양쪽에서 동일하게 사용할 수 있습니다.

```javascript
// utils/formatDate.js
export function formatDate(dateStr) {
  return new Date(dateStr).toLocaleDateString('ko-KR');
}
```

```javascript
// pages/index.js
import { formatDate } from '../utils/formatDate';

export async function getServerSideProps() {
  const data = await fetch('https://api.example.com/post/1').then(res => res.json());
  return {
    props: {
      postDate: formatDate(data.date) // 서버에서 실행
    }
  };
}

export default function Home({ postDate }) {
  return <div>게시 날짜: {postDate}</div>; // 클라이언트에서 hydrate
}
```


## 📚 참고 자료
- [MDN - JavaScript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)