## MVC 패턴
애플리케이션을 **세 가지 역할로 분리해서 관리**하는 구조적 설계 패턴이다.
비즈니스 데이터(Model), UI(View), 로직과 사용자 입력 관리(Controller)로 나뉜다.

### 모델
애플리케이션의 데이터를 관리하는 역할을 한다. 모델이 변경 될 때 관찰자(View)에게 변경사항을 알린다.
### 뷰
사용자와 상호작용하는 인터페이스(UI)를 담당하며, 모델의 상태를 화면에 보여준다.
모델에 변화가 생기면 알림을 받고, 뷰 스스로를 업데이트 한다.
#### 템플릿
템플릿은 **화면에 찍어 낼 HTML 구조의 틀**을 말한다.
템플릿은 뷰 자체가 아니라, 뷰 객체의 일부 또는 전체를 선언적으로 정의하는 방식이다.
Tagged templete literal을 이용해서 구성할 수 있다.
### 컨트롤러
모델과 뷰 사이의 중재자 역할을 하며, 사용자 입력에 따라 적절한 뷰나 모델을 업데이트 한다.
애플리케이션 내에서 모델과 뷰 간의 로직, 연동을 관리한다.

### 예제
관찰자 패턴이 사용된다!!
```js
/* ── Model ── */
class TextModel {
  #value = '';
  #listeners = new Set();
  get value()        { return this.#value; }
  set value(v)       { this.#value = v; this.#notify(); }
  subscribe(f)       { this.#listeners.add(f); }
  #notify()          { this.#listeners.forEach(f => f(this)); }
}

/* ── View (템플릿 + 이벤트 hookup) ── */
class TextView {
  constructor(root){ this.root = root; }
  render(value) {
    this.root.innerHTML = `
      <input id="inp" placeholder="Type…" />
      <p>${value.toUpperCase()}</p>`;        /* View 가 직접 대문자 변환 */
    this.$inp = this.root.querySelector('#inp');
  }
  bindInput(handler){ this.$inp.addEventListener('input', e => handler(e.target.value)); }
}

/* ── Controller ── */
class TextController {
  constructor(m, v){
    v.render(m.value);                          // 초기 화면
    v.bindInput(val => m.value = val);          // View → Model
    m.subscribe(({value}) => v.render(value));  // Model → View
  }
}
new TextController(
  new TextModel(),
  new TextView(document.getElementById('mvc-root'))
);
```


### MVC를 사용하는 이유?
- **유지보수가 쉬워짐**
	- 변경이 데이터 로직에 관한 것인지, 화면(UI)에 관한 것인지를 명확히 구분할 수 있어 관리가 편해짐.
- **역할 분담이 쉬움**
	- 규모가 큰 프로젝트에서도 UI 개발자와 로직 개발자가 동시에 작업 가능.
* **모델과 뷰의 분리**
	* 비즈니스 로직을 독립적으로 테스트 가능 → 단위 테스트 작성이 쉬워짐.
- **중복 코드 제거**
	- 모델과 컨트롤러를 재사용함으로써 하위 수준 코드의 중복이 줄어듦.


## MVP 패턴
프레젠테이션 로직의 개선에 초점을 맞춘 MVC 패턴의 파생 패턴이다.
로직을 거의 가지고 있지 않은 수동형 뷰를 활용할 때 널리 사용된다.

### 모델, 뷰, 프리젠터
프리젠터는 데이터를 가져오고, 조작하고 , 이 데이터가 어떻게 뷰에 표시되어야하는지를 결정한다.

mvc와 달리 뷰가 직접 모델을 관찰하지 않는다.
따라서 mvc에 비해 뷰와 모델간의 분리를 더욱 명확하게 해주며, 애플리케이션 테스트가 용이해진다.
그리고 프레젠테이션 로직의 재사용이 쉬워진다.


### 예제
```js
/* ── Model ── */
class TextModel {
  #value = ''; #listeners = new Set();
  get value()       { return this.#value; }
  set value(v)      { this.#value = v; this.#notify(); }
  subscribe(f)      { this.#listeners.add(f); }
  #notify()         { this.#listeners.forEach(f => f(this)); }
}

/* ── View : 표시·이벤트 슬롯만 ── */
class TextView {
  constructor(root){ this.root = root; }
  display(dto){                       // Presenter가 준 DTO만 사용
    this.root.innerHTML = `
      <input id="inp" placeholder="Type…" value="${dto.raw}" />
      <p>${dto.upper}</p>`;
    this.$inp = this.root.querySelector('#inp');
  }
  onInput(cb){ this.$inp.addEventListener('input', e => cb(e.target.value)); }
}

/* ── Presenter : Model 조작 + DTO 가공 + View 호출 ── */
class TextPresenter {
  constructor(m, v){
    this.m = m; 
    this.v = v;
    v.onInput(val => m.value = val);          // View → Model
    m.subscribe(()=> this.refresh());         // Model → View
    this.refresh();                           // 초기 화면
  }
  refresh(){
    const raw = this.m.value;
    this.v.display({                          // View는 DTO만 받음
      raw,
      upper: raw.toUpperCase()                // **가공 책임은 Presenter**
    });
  }
}
new TextPresenter(
  new TextModel(),
  new TextView(document.getElementById('mvp-root'))
);
```


## MVVM 패턴
애플리케이션의 UI 개발 부분과 비즈니스로직, 동작 부분을 명확하게 분리하기 위한 아키택처 패턴이다.
선언적 데이터 바인딩을 활용해서 뷰에 대한 작업을 다른 계층과 분리할 수 있도록 한다.

### 모델
애플리케이션이 사용할 도메인 관련 데이터나 정보를 관리 및 제공한다.

### 뷰
사용자와 상호작용이 가능한 사용자 인터페이스(UI)이다. 뷰모델의 상태를 표현한다.
뷰모델과 정보, 상태를 항상 동기화된 상태로 유지하기 때문에 뷰는 상태를 관리할 책임이 없다.

### 뷰모델
데이터 변환기의 역할을 하는 특수한 컨트롤러이다.
모델의 정보를 뷰가 사용할 수 있는 형태로 변환하고, 뷰에서 발생한 사용자 조작이나 이벤트를 모델로 전달한다.
뷰와 뷰모델은 데이터 바인딩과 이벤트를 통해 소통한다

#### 데이터 바인딩
데이터 바인딩(Data Binding)은 **“상태(state)를 화면(UI)과 자동으로 동기화해 주는 기법”** 이다.
즉, 모델(데이터)이 바뀌면 뷰(화면)가, 뷰가 사용자 이벤트로 바뀌면 다시 모델이 자동으로 갱신되도록 연결해 두는 것 이다.

**데이터 바인딩 없이 순수 DOM 조작**
```html
<!-- index.html -->
<input id="msg" placeholder="메시지를 입력하세요" />
<p id="preview"></p>

<script>
  // 모델(데이터) – 단순 변수
  let message = '';

  // DOM 캐싱
  const $input   = document.getElementById('msg');
  const $preview = document.getElementById('preview');

  // View → Model → View 수동 처리
  $input.addEventListener('input', (e) => {
    message = e.target.value;           // ① 모델 갱신
    $preview.textContent = message;     // ② DOM 직접 갱신
  });
</script>
```

**단방향 바인딩 (React)**
```jsx
import { useState } from 'react';

export default function MessagePreview() {
  const [message, setMessage] = useState('');

  return (
    <>
      <input
        placeholder="메시지를 입력하세요"
        value={message}
        onChange={(e) => setMessage(e.target.value)}   // Model 갱신
      />
      <p>{message}</p>                                 {/* 상태 → UI */}
    </>
  );
}
```

**양방향 바인딩 (Vue)**
``` vue
<script setup>
import { ref } from 'vue';
const message = ref('');
</script>

<template>
  <input v-model="message" placeholder="메시지를 입력하세요" />
  <p>{{ message }}</p>
</template>
```

### 예제
```js
/* Model */
const model = { text: '' };

const target = {
    setText(v) {
        this.text = v
    },
    get upper() {
        return this.text.toUpperCase()
    },
}

const handler = {
    get(_, p) {
        if (p in model) {
            return model[p]
        }
        return Reflect.get(...arguments)
    },
    set(_, p, v) {
        model[p] = v
        render()
        return true
    },
}
/* ViewModel : 원본(text) + 파생 upper (getter) */
const vm = new Proxy(target, handler);

const root = document.getElementById('mvvm-root');

/* View */
function render() {
  root.innerHTML = `
    <input id="inp" value="${vm.text}" placeholder="Type…" />
    <p>${vm.upper}</p>`;                            <!-- 뷰는 그대로 표시만 -->
  root.querySelector('#inp')
      .addEventListener('input', e => vm.setText(e.target.value));
}
render();
```
객체의 동작을 중간에서 가로채는 **Proxy**를 이용해서 VM의 역할을 구현

**get(_, p)**
- vm.text → model.text에서 가져옴
- vm.upper → model.upper가 없으므로 fallback → target.upper 호출됨
    - 이때 내부에서 호출된 this.text는 model.text가 됨 (핵심 트릭)

**set(_, p, v)**
- vm.setText('abc') → model.setText가 없으므로 fallback → target.setText가 호출됨
	- 이때 내부에서 this.text를 set하는 로직에서 model에 직접 저장되고 render() 호출됨 (핵심 트릭)

### 장점
- UI와 이를 구동하는 로직(ViewModel, Model)을 **동시에** 개발할 수 있음
- View 내부에 **비즈니스 로직이나 이벤트 처리 코드**가 줄어듦
- ViewModel은 UI와 분리되어 있어 **UI 고려 없이 테스트 가능**

### 단점
- UI가 간단한 경우에는 MVVM이 **오히려 복잡하고 과한** 구조가 될 수 있음
- 바인딩이 많아질수록 **성능과 관리 부담**이 커지고, 바인딩 코드가 **오히려 객체보다 무거워지는 상황**도 발생함
- 앱이 클수록 다양한 뷰모델을 미리 설계하기가 어렵고, 일반화된 뷰모델 제공이 **도전 과제**가 될 수 있음


## MVC vs MVP vs MVVM
| **구성요소**   | **MVC (Model-View-Controller)**              | **MVP (Model-View-Presenter)**                | **MVVM (Model-View-ViewModel)**                                  |
| ---------- | -------------------------------------------- | --------------------------------------------- | ---------------------------------------------------------------- |
| **Model**  | 비즈니스 로직, 데이터 처리                              | 동일                                            | 동일                                                               |
| **View**   | UI, 사용자 입력 전달                                | UI, 사용자 입력 전달                                 | UI, ViewModel과 데이터 바인딩                                           |
| **중간 계층**  | **Controller**- 사용자 입력을 처리하고 Model과 View를 연결 | **Presenter**- View와 Model을 연결하며, View에 결과 전달 | **ViewModel**- View와 Model 사이에서 데이터 상태 관리- View와 **양방향 데이터 바인딩** |
| **역할 분리**  | View와 Controller 간 결합이 큼                     | View는 Presenter에 의존, 테스트 용이                   | View와 ViewModel은 느슨하게 연결됨 (데이터 바인딩 이용)                           |
| **데이터 흐름** | 사용자 입력 → Controller → Model → View           | 사용자 입력 → View → Presenter → Model → View      | 사용자 입력 → View  ↔ ViewModel → Model<br>View는 ViewModel과 바인딩됨      |
| **단위 테스트** | 어렵거나 제한적                                     | Presenter 단독 테스트 가능                           | ViewModel 단독 테스트 용이                                              |
