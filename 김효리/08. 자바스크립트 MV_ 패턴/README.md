애플리케이션 아키텍처 패턴

- MVC(모델, 뷰, 컨트롤러)
- MVP(모델, 뷰, 프리젠터)
- MVVM(모델, 뷰, 뷰모델)

## MVC 패턴

> 애플리케이션의 구조를 개선하기 위해 관심사의 분리를 활용하는 아키텍처 디자인 패턴

### 8.1.1 Smalltalk-80의 MVC 패턴

- 모델: **도메인 관련 데이터**. 모델이 변경되면 관찰자 객체에게 알림을 보낸다.
- 뷰: **모델의 현재 상태** 표현. 관찰자 패턴을 사용해 모델이 변경되면 뷰가 알아차릴 수 있도록 한다.
- 컨트롤러: **사용자의 상호작용을 처리**하고 뷰에 무엇을 보여줄지, 사용자 입력을 어떻게 처리할지 결정한다.

### 8.2 자바스크립트의 MVC

- 현대 자바스크립트 개발자는 MVC 패턴과 **그 변형 버전들이 제공하는 장점**을 이해해야 한다.

### 8.2.1 모델

모델은 데이터 관리. 이 데이터가 변경되면 뷰에게 알린다.

보통 MVC/MV\* 프레임워크에서는 모델을 그룹화하여 그룹 내 특정 모델이 변경될 때 그룹의 알림으로 애플리케이션 로직을 작성한다. 이러면 개별 모델 인스턴스를 직접 관찰할 필요가 없다.

### 8.2.2 뷰

모델에 대한 시각적인 표현으로, 현재 상태의 특정 부분만 보여준다.

모델에 변화가 생기면 알림을 받아 **스스로 업데이트** 할 수 있다.

사용자는 뷰와 상호작용할 수 있다.

### 8.2.3 템플릿

div 남발, 컨씨컨븨 하는 코드는 좋지 않다.

따라서 자바스크립트 태그 템플릿 리터럴을 사용하면 좋다.

```tsx
// html 태그 템플릿 함수 정의
function html(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const val = values[i] !== undefined ? values[i] : '';
    return result + str + val;
  }, '');
}

// 재사용 가능한 컴포넌트 함수
function UserCard(name, email) {
  return html`
    <div class="user-card">
      <h2>${name}</h2>
      <p>${email}</p>
    </div>
  `;
}

// 여러 사용자 데이터 사용
const users = [
  { name: '철수', email: 'chulsoo@example.com' },
  { name: '영희', email: 'younghee@example.com' },
];

const resultHTML = html`
  <section>
    <h1>사용자 목록</h1>
    ${users.map((u) => UserCard(u.name, u.email)).join('')}
  </section>
`;

console.log(resultHTML);
```

```tsx
<section>
  <h1>사용자 목록</h1>
  <div class='user-card'>
    <h2>철수</h2>
    <p>chulsoo@example.com</p>
  </div>
  <div class='user-card'>
    <h2>영희</h2>
    <p>younghee@example.com</p>
  </div>
</section>
```

템플릿은 뷰의 일부 혹은 전체를 선언적으로 지정하는 방법이 될 수 있다.

### 8.2.4 컨트롤러

모델과 뷰 사이의 중재자 역할을 한다.

사용자가 뷰를 조작할 때 모델을 업데이트하는 역할을 한다.

## 8.3 MVC를 사용하는 이유는?

- 전반적인 유지보수의 단순화: 변경사항이 데이터 중심인지, 시각적인 변경인지 명확하게 구분할 수 있다.
- 모델과 뷰의 분리: 단위 테스트 작성이 훨씬 편해진다.
- 하위 수준의 모델 및 컨트롤러 코드 중복이 제거된다.
- 코어 로직을 담당하는 개발자와 UI 작업을 담당하는 개발자가 동시에 작업할 수 있다.

## 8.4 자바스크립트와 Smalltalk-80의 MVC

피터 미쇼의 [Maria.js](https://github.com/petermichaux/maria)는 기존의 MVC 근본에 충실한 구현을 재현한 프레임워크다.

```tsx
// 모델 정의
var TaskModel = maria.Model.subclass({
  properties: {
    title: '',
    completed: false,
  },
});

// 뷰 정의
var TaskView = maria.View.subclass({
  build: function () {
    this.el = document.createElement('div');
    this.el.textContent = this.model.get('title');
  },
});

// 인스턴스 생성
var task = new TaskModel({ title: '할 일 1' });
var view = new TaskView({ model: task });

// 뷰를 DOM에 추가
document.body.appendChild(view.el);
```

## 8.6 MVP 패턴

Model View Presenter 패턴은 프레젠테이션 로직의 개선에 초점을 맞춘 MVC 디자인 패턴의 파생이다.

### 8.6.1 모델, 뷰, 프리젠터

프리젠터는 뷰에 대한 UI 비즈니스 로직을 담당하는 구성 요소이다.

뷰에서의 이벤트 호출은 프리젠터로 위임된다.

프리젠터는 뷰와 분리되어 있지만 인터페이스를 통해 뷰와 통신한다.

| **항목**                | **MVC (Model-View-Controller)**                          | **MVP (Model-View-Presenter)**                                   |
| ----------------------- | -------------------------------------------------------- | ---------------------------------------------------------------- |
| **View**                | 사용자에게 UI를 보여줌                                   | 사용자에게 UI를 보여줌 (MVC와 동일)                              |
| **Model**               | 데이터 및 비즈니스 로직                                  | 데이터 및 비즈니스 로직 (MVC와 동일)                             |
| **Control / Presenter** | View의 이벤트를 처리하고 Model 업데이트                  | View의 로직과 데이터를 완전히 관리                               |
| **View ↔ Controller**   | View가 Controller에 이벤트 위임, Controller가 Model 변경 | **View는 Presenter에게만 의존**, Presenter가 Model과 View를 연결 |
| **의존 방향**           | View ↔ Controller → Model                                | View → Presenter → Model (View는 Presenter에만 의존)             |
| **단방향/양방향**       | 양방향 (View가 Controller 호출 가능)                     | 단방향 (View는 Presenter만 알고 있음)                            |
| **테스트 용이성**       | Controller에 뷰 로직이 엉킬 수 있음                      | Presenter는 View 인터페이스만 알고 있어 **테스트 용이**          |

![image.png](attachment:b405e2f7-3ed0-42e1-ae16-b05c89fe80e0:image.png)

Model이 View에 관여를 하냐 안 하냐로 갈린다.

**MVP에서 View는 Model을 알지 못한다.**

## 8.7 MVVM 패턴

애플리케이션 UI 개발 부분과 비즈니스 로직, 동작 부분을 명확하게 분리한다. 동일한 코드베이스 내에서 UI 작업과 개발 작업을 거의 동시에 진행할 수 있다.

### 뷰

수동적 뷰: 단순히 화면을 출력할 뿐 사용자의 입력을 받아들이지 않는다.

능동적 뷰: 데이터 바인딩, 이벤트, 동작들을 포함하고 있다.

### 뷰모델

데이터 변환기의 역할을 하는 특수한 컨트롤러.

뷰의 상태를 유지하고, 뷰에서 발생한 동작에 기반해 모델을 업데이트한다. 뷰에 이벤트를 발생시키는 등의 기능을 수행하기 위한 메서드도 제공할 수 있다.

모델과 뷰모델의 속성은 **양방향 데이터 바인딩**을 통해 동기화되고 업데이트된다.

### 장점

- UI와 이를 구동하는 요소를 동시에 개발할 수 있게 해준다.
- 뷰를 추상화함으로써 비즈니스 로직의 양을 줄여준다.

### 단점

- 단순한 UI일 경우 과도한 구현이 될 수 있다.
- 복잡한 애플리케이션에서는 데이터 바인딩이 상당한 관리 부담을 만들어낼 수 있다.

(???? 그럼 단순해도 안되고 복잡해도 안된다고 …? 😮)

### 8.10 최신 MV\* 패턴

Vue.js: MVVM 패턴

리액트: MVC가 아니다.

- **Vue**는 프레임워크가 **MVVM 흐름**을 거의 정석으로 제공하므로 “패턴”이 명확하게 느껴진다.
- **React**는 필요한 아키텍처를 **개발자가 조합**(Redux로 MVC 흉내, MobX로 MVVM 등)할 수 있는 **플랫폼**에 가깝다.
