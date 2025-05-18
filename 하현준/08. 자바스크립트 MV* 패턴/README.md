## MVC 패턴

MVC 패턴은 비지니스 데이터(모델)과 UI(뷰)를 분리하고, 세 번째 구성 요소가 로직과 사용자 입력을 관리하는 구조이다.

- **모델**: 도메인 관련 데이터를 표현했으며 UI에 대해서는 관여하지 않는다. 모델이 변경되면 자신의 관찰자 객체에게 알림을 보냈다.
- **뷰**: 모델의 현재 상태를 표현한다. 관찰자 패턴을 사용해 모델이 변경되거나 수정될 때마다 뷰가 알아차릴 수 있도록 한다.
- **컨트롤러**: 키보드 입력이나 클릭 같은 사용자의 상호작용을 처리하고 뷰에 무엇을 보여줄지, 사용자 입력을 어떻게 처리할지 등을 결정하는 역할을 한다.

🤔 Smalltalk-80이 무엇인가?

> **스몰토크**(Smalltalk)는 1970년대 초, [미국](https://namu.wiki/w/%EB%AF%B8%EA%B5%AD) 제록스(XEROX)사의 팰러 앨토 연구 센터(PARC)에서 개발한 [객체 지향 프로그래밍](https://namu.wiki/w/%EA%B0%9D%EC%B2%B4%20%EC%A7%80%ED%96%A5%20%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D) 언어이다. [C++](https://namu.wiki/w/C%2B%2B), [Java](https://namu.wiki/w/Java)와 같은 **객체 지향 프로그램의 시초**이다.(단, Simula라는 시뮬레이션 전용 언어가 객체지향의 기본적인 개념을 제시하기는 했다.) 이 외에도 [Ruby](https://namu.wiki/w/Ruby), [Python](https://namu.wiki/w/Python), AppleScript, Dylan, [Perl](https://namu.wiki/w/Perl), [Scala](https://namu.wiki/w/Scala) 같은 수많은 프로그래밍 언어에 영향을 주었으며, 그 외에 최초로 [GUI](https://namu.wiki/w/GUI)(그래픽 사용자 인터페이스)를 제공하는 언어로 [매킨토시](<https://namu.wiki/w/Macintosh(%EC%BB%B4%ED%93%A8%ED%84%B0)>)의 GUI에 영향을 준, 쉽게 말하자면 프로그래밍 언어의 한 획을 그은 언어이다.

## 자바스크립트의 MVC

현재 여러 프레임워크에 MVC 패턴이 녹아져 있다.

![image.png](attachment:c595f7ed-b874-49fd-a82b-82b834136c00:image.png)

### 모델

모델은 애플리케이션의 데이터를 관리하는 역할, 애플레케이션에 필요한 고유 데이터 형식을 나타낸다.

- 한 가지 모델을 여러 뷰가 관찰할 수 있다.
- 모델을 그룹으로 관리하면 그룹 내에 특정 모델이 변경될 때 그룹의 알림을 기반으로 로직을 작성할 수 있어 개별 모듈 인스턴스를 직접 관찰할 필요가 없어진다.

정리하자면 ‘상태’ 라는 용어로 사용되며, **비지니스 데이터**와 주로 관련이 있다.

### 뷰

모델에 대한 시각적인 표현한다.

- 사용자는 뷰와 상호작용을 할 수 있다.
- 뷰는 프레젠테이션 계층이기 때문에 모델을 실제로 업데이트 하는 기능은 컨트롤러가 담당한다.

### 템플릿

문자열의 연결을 통해 메모리에 큰 HTML을 수동으로 생성하고 업데이트하는 것은 성능적으로 좋지 않는 이슈가 있다. 태그 템플릿 리터럴을 사용하여 동적인 HTML 컨텐츠를 생성하는 방법이다.

- 템플릿은 뷰를 생성하기 위해 사용될 수 있는 것이지 뷰가 아니다.
- 뷰 객체의 일부 또는 전체를 선언적으로 지정하는 방법중 하나이다.

### 컨트롤러

모델과 뷰 사이의 중재자 역할, 뷰를 조작할 때 모델을 업데이트하는 역할을 한다.

## 사용하는 이유?

- 전반적인 유지보수의 단순화: 애플리케이션을 업데이트해야 할 때, 변경사항이 데이터 중심인지 단순한 시각적인 변경인지 명확하게 구분할 수 있다.
- 모델과 뷰의 분리: 비지니스 로직에 대한 단위 테스트 작성이 훨씬 간편해진다.
- 하위 수준의 모델 및 컨트롤러 코드 중복이 제거된다.
- 애플리케이션의 규모와 역할의 분리 정도에 따라, 모듈화를 통해 코어 로직을 담당하는 개발자와 UI 작업을 담당하는 개발자가 동시에 작업할 수 있다.

## MVP

뷰의 요청에 따라, 프레젠터는 사용자 요청과 관련된 작업을 수행하고 데이터를 뷰로 다시 전달한다.

![image.png](attachment:cbba0477-7561-4cf2-ac9b-b37731043387:image.png)

- 뷰와 모델 간의 분리를 더욱 명확하게 해준다는 장점이 있다
- 데이터 바인딩이 지원되지 않기 때문에 직접 처리해야 한다.

## MVVM 패턴

선언적 데이터 바인딩을 활용하여 뷰에 대한 작업을 다른 계층과 분리할 수 있도록 한다.

![image.png](attachment:eea1a0cf-16aa-44fa-9351-050adccbdcc8:image.png)

- 뷰: 사용자 인터페이스
- 모델: 도메인에 관련된 정보를 전달
- 뷰모델: 모델과 뷰 사이의 인터페이스 역할

### 뷰 vs 뷰모델

뷰와 뷰모델은 데이터 바인딩과 이벤트를 통해 소통한다. 뷰는 자체 UI 이벤트를 처리하고 필요에 따라 뷰모델에 연결한다. 모델과 뷰모델의 속성은 양방향 데이터 바인딩을 통해 동기화되고 업데이트 된다.

### 뷰모델 vs 모델

뷰모델은 모델에 대해 전적인 책임을 진다. 뷰모델은 데이터 바인딩을 위해 모델 또는 모델의 속성을 가져올 수 있고, 뷰에 제공되는 속성을 가져오거나 조작하기 위한 인터페이스를 포함할 수 있다.

🤔 MVC, MVP, MVVM, MVI

글로 표현하는 것보다는 코드로 보는게 훨씬 와닿기 때문에 코드로 알아보는 것이 더 최고

**MVC**

```tsx
// Model
const TodoModel = {
  todos: [],
  addTodo(text) {
    this.todos.push({ text, id: Date.now() });
  },
  getTodos() {
    return this.todos;
  },
};

// Controller
function useTodoController(setTodos) {
  const addTodo = (text) => {
    TodoModel.addTodo(text);
    setTodos([...TodoModel.getTodos()]);
  };

  return { addTodo };
}

// View
function TodoView() {
  const [text, setText] = React.useState("");
  const [todos, setTodos] = React.useState(TodoModel.getTodos());
  const { addTodo } = useTodoController(setTodos);

  const handleSubmit = () => {
    if (text.trim()) {
      addTodo(text);
      setText("");
    }
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleSubmit}>추가</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

**MVP**

```tsx
// Model
const TodoModel = {
  todos: [],
  addTodo(text) {
    this.todos.push({ text, id: Date.now() });
  },
  getTodos() {
    return this.todos;
  },
};

// Presenter
function useTodoPresenter() {
  const [todos, setTodos] = React.useState(TodoModel.getTodos());

  const addTodo = (text) => {
    TodoModel.addTodo(text);
    setTodos([...TodoModel.getTodos()]);
  };

  return { todos, addTodo };
}

// View
function TodoView({ todos, onAdd }) {
  const [text, setText] = React.useState("");

  const handleAdd = () => {
    if (text.trim()) {
      onAdd(text);
      setText("");
    }
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleAdd}>추가</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}

// App (Presenter → View 연결)
function App() {
  const { todos, addTodo } = useTodoPresenter();
  return <TodoView todos={todos} onAdd={addTodo} />;
}
```

**MVVM**

```tsx
// Model
const TodoModel = {
  todos: [],
  addTodo(text) {
    this.todos.push({ text, id: Date.now() });
  },
  getTodos() {
    return this.todos;
  },
};

// ViewModel (hook)
function useTodoViewModel() {
  const [todos, setTodos] = React.useState(TodoModel.getTodos());

  const addTodo = (text) => {
    TodoModel.addTodo(text);
    setTodos([...TodoModel.getTodos()]);
  };

  return { todos, addTodo };
}

// View
function TodoView() {
  const { todos, addTodo } = useTodoViewModel();
  const [text, setText] = React.useState("");

  const handleAdd = () => {
    if (text.trim()) {
      addTodo(text);
      setText("");
    }
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleAdd}>추가</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

**MVI**

![image.png](attachment:95914d15-67b1-4023-8599-61799c633c84:image.png)

**MVI 패턴이란?**

MVI 패턴은 모델, 뷰, 인텐트로 구성됩니다. **모델은** 애플리케이션의 상태를 나타내며, **뷰는** 사용자 인터페이스를 담당한다. **인텐트는** 사용자 인터랙션을 처리하고, 모델의 상태를 변경하는 역할을 한다.

- 테스트 용이성을 높인다.
- 단반향 데이터 흐름을 통해 상태 관리의 복잡성을 줄인다.
- 초기 학습 곡선이 높고, 코드의 양이 많아질 수 있다.

```tsx
// 🧠 Model (State + Reducer)
const initialState = {
  todos: [],
  input: "",
};

function todoReducer(state, action) {
  switch (action.type) {
    case "INPUT_CHANGE":
      return { ...state, input: action.payload };
    case "ADD_TODO":
      if (!state.input.trim()) return state;
      return {
        ...state,
        todos: [...state.todos, { id: Date.now(), text: state.input }],
        input: "",
      };
    default:
      return state;
  }
}

// 🎯 Intent: dispatch를 통해 사용자 행동 전달
// 👁️ View
function TodoView() {
  const [state, dispatch] = React.useReducer(todoReducer, initialState);

  const onInputChange = (e) =>
    dispatch({ type: "INPUT_CHANGE", payload: e.target.value });
  const onAdd = () => dispatch({ type: "ADD_TODO" });

  return (
    <div>
      <input value={state.input} onChange={onInputChange} />
      <button onClick={onAdd}>추가</button>
      <ul>
        {state.todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

**🎯 아래 코드는 MVC인데, MVP, MVVM, MVI를 각각 적용해보는거 어떨까요?**

**구현이 필요한 요구사항들**

- 이메일/비밀번호 입력
- 로그인 버튼 클릭 시 유효성 검사
- 오류 메시지, 성공 메시지 표기

```tsx
// 🧩 Model (데이터 구조)
const createLoginForm = (email = "", password = "") => ({ email, password });

// ⚙️ Service (비즈니스 로직)
const AuthService = {
  login({ email, password }) {
    if (!email.includes("@"))
      throw new Error("이메일 형식이 올바르지 않습니다");
    if (password.length < 6)
      throw new Error("비밀번호는 최소 6자 이상이어야 합니다");

    // 예시용 성공 반환 (실제에선 API 호출)
    return { success: true, token: "abc123" };
  },
};

// 🧠 Controller (이벤트 → 로직 → 결과 전달)
function useLoginController(setState) {
  const handleLogin = (email, password) => {
    try {
      const form = createLoginForm(email, password);
      const result = AuthService.login(form);
      setState({ error: "", message: "로그인 성공!" });
    } catch (err) {
      setState({ error: err.message, message: "" });
    }
  };

  return { handleLogin };
}

// 🎨 View (UI만 담당)
function LoginView() {
  const [email, setEmail] = React.useState("");
  const [password, setPassword] = React.useState("");
  const [state, setState] = React.useState({ error: "", message: "" });

  const { handleLogin } = useLoginController(setState);

  return (
    <div
      style={{ maxWidth: 300, margin: "50px auto", fontFamily: "sans-serif" }}
    >
      <h2>로그인</h2>
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        style={{ display: "block", marginBottom: 10, width: "100%" }}
      />
      <input
        type="password"
        placeholder="Password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        style={{ display: "block", marginBottom: 10, width: "100%" }}
      />
      <button
        onClick={() => handleLogin(email, password)}
        style={{ width: "100%", padding: 8 }}
      >
        로그인
      </button>

      {state.error && (
        <p style={{ color: "red", marginTop: 10 }}>{state.error}</p>
      )}
      {state.message && (
        <p style={{ color: "green", marginTop: 10 }}>{state.message}</p>
      )}
    </div>
  );
}
```

+++ 좋은글
[https://velog.io/@teo/프론트엔드에서-MV-아키텍쳐란-무엇인가요](https://velog.io/@teo/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EC%97%90%EC%84%9C-MV-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94)
