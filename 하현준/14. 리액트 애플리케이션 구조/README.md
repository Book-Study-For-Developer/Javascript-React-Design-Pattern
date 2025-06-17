## 파일을 그룹화 하는 방법

### 1. 모듈, 기능 또는 경로별 그룹화

파일 구조를 비지니스 모델이나 에플리케이션의 흐름에 따라 묶는 방법

```tsx
common / Avatar.js;
Avatar.css;
ErrorUtils.js;
ErrorUtils.test.js;
product / index.js;
product.css;
price.js;
product.test.js;
checkout / index.js;
checkout.css;
checkout.test.js;
```

장점

- 모듈에 변경사항이 있을 때 관련된 모든 파일이 같은 폴더에 모여 있어 변경사항이 제한된다.

단점

- 모듈 간에 공통적으로 사용되는 컴포넌트, 로직, 스타일을 주기적으로 파악해야만 중복을 피하고 일관성과 재사용성을 높일 수 있다.

### 2. 파일 유형별 그룹화

파일 유형 별로 서로 다른 폴더를 생성하는 것, **논리적으로** 관련된 파일들이 파일 유형에 따라 서로 다른 폴더에 위치한다.

```tsx
css / global.css;
checkout.css;
product.css;
lib / date.js;
currency.js;
gtm.js;
pages / product.js;
productlist.js;
checkout.js;
```

장점

- 표준 구조 재사용: 여러 프로젝트에서 동일하게 사용할 수 있는 표준적인 구조를 가짐
- 새로운 팀원의 빠른 적응: 애플리케이션 별 로직에 대한 지식이 부족한 신규 팀원도 빠르게 파악할 수 있다.
- 공통 컴포넌트 및 스타일 변경 용이: 여러 경로나 모듈에서 가져오는 공통컴포넌트나 스타일을 변경하면 전체에 적용된다.

단점

- 모듈 수정 시 여러 폴더 수정 필요: 특정 모듈의 로직을 수정하려면 여러 폴더를 수정해야한다.
- 파일 찾기 어려움: 애플리케이션의 기능이 많아질수록 각 폴더의 파일 수가 증가하여 특정 파일을 찾기 어려움

🤔 저는 2번의 단점이 굉장히 부각되는 것 같아 예전에 개발 시작할때는 자주 썼던 방식인데, 최근에는 안쓰는 방식입니다. 관련성이 높은 파일끼리 묶어놓아야 한다고 생각하여 위 구조는 도움이 되지 않는 듯 해요

### 3. 도메인 및 공통 컴포넌트 기반의 혼합 그룹화

애플리케이션 전체에서 공통적으로 사용되는 컴포넌트들을 모두 Components 폴더, 나머지 비지니스 로직에 관련된 것들은 domain폴더에 그룹화 하는 것

```tsx
css / global.css;
components / User / porfile.js;
profile.test.js;
avatar.js;
date.js;
currency.js;
gtm.js;
errorUtils.js;
domain / product / product.js;
product.css;
product.test.js;
checkout / checkout.js;
checkout.css;
checkout.test.js;
```

1, 2번의 장점을 모두 활용하는 방법이다.

🤔 지금 내가 있는 팀에서의 폴더 구조는?

마지막에 소개한 혼합형을 쓰고 있습니다.

```tsx
css/
 global.css
shared/
  utils/
  components/
  helpers/
  hooks/
  providers
  ...
lib/ // 특정 라이브러리와 같은 역할을 하는 곳들
	error/
	eventManager/
	...
apis/ // api와 관련된 로직
	...
src/app // 앱라우터
  product/
	  _components/
	  _utils/
	  _helpers/
	  _hooks/
	...
```

한가지 여기서 더 특별한 점은 컴포넌트 안에도 utils.ts나 helpers.ts를 넣을 때도 있습니다.

Product 컴포넌트라면?

```tsx
Product / index.tsx;
index.module.scss;
helpers.ts; // (product 컴포넌트에 필요한 헬퍼들)
utils.ts;
useXXX; // 사용되는 훅이 있다면 여기에 정의
```

이런식으로 가까운 곳에 위치시키는 것을 선호합니다. 또 비슷한 역할을 하는 컴포넌트는 **폴더로 한번 더 묶기**도 합니다.

ex) 특정 앱라우터 하위의 폴더

```tsx
// 바텀시트 역할을하는 컴포넌트들, 에러바운더리 역할을 하는 컴포넌트들
_components/
  bottomsheet/
	  xxxBottomsheet.tsx
	  ...
	errorBoundary/
		xxxErrorBoundary.tsx
```

또 모든 컴포넌트를 1뎁스로 나열하지 않고 컴포넌트 안에 컴포넌트를 위치시키는 경우도 있습니다.

```tsx
_components / ProfileList / ProfileItem / index.tsx;
MyProfile / index.tsx;
index.tsx;
```

이런식으로 내부에서만 사용되는 컴포넌트들을 쭉 나열합니다.

😆 다른분들은 혹시 Next에서 특별히 사용되는 컴포넌트 구조가 있으실까요?

- 최근에 폴더 구조 관련해서 글이 올라왔더라고요. [폴더구조의 변화로 이해하는 프론트엔드 멘탈모델 변천사
  ](https://velog.io/@teo/folder-structure)
