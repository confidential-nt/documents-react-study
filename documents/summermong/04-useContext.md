# ✏️ useContext

컴포넌트에서 Context를 읽고 구독할 수 있게 해주는 React Hook
Context란 컴포넌트 간 어떤 값을 공유할 수 있게 하는 기능으로, 주로 전역으로 필요한 값을 다룰 때 사용한다. 즉 Context는 React에서 Props가 아닌 또 다른 방식으로 컴포넌트에 값을 전달하는 방법이라고 보면 된다.

## Reference

### 📌 useContext(SomeContext)

컴포넌트의 최상위 레벨에서 useContext를 호출해 context를 읽고 구독한다.

### 📌 매개변수

SomeContext: 이전에 createContext로 생성한 context로, 이 자체는 정보를 갖지 않는다. 컴포넌트에서 제공하거나 읽을 수 있는 정보의 종류를 나타낸다.

### 📌 반환값

useContext는 호출하는 컴포넌트에 대한 context 값을 반환한다.
이 값은 호출한 컴포넌트에서 트리상 위에 있는 가장 가까운 SomeContext.Provider에 전달된 value다.
Provider가 없을 경우 반환되는 값은 해당 context에 대해 createContext에 전달한 기본값이 된다.
반환값은 항상 최신이며, React는 context 변경 시 해당 context를 읽는 컴포넌트를 자동으로 '리렌더링'한다.

### 📌 주의사항

1. Context.Provider는 반드시 useContext 호출을 수행하는 컴포넌트의 **위**에 있어야 한다.
2. React는 변경된 value를 받는 provider부터 해당 context를 사용하는 자식 컴포넌트까지 전부 자동으로 **리렌더링**한다. 이전 값과 다음 값은 Object.is로 비교해 변경된 것을 확인한다. memo로 리렌더링을 건너뛰어도 새로운 context를 수신하게 된다.

=> Context.Provider에 제공하는 value Props가 바뀌면 하위 컴포넌트도 모두 리렌더링 된다.
=> 이를 막고 싶다면 리렌더링을 방지하고 싶은 컴포넌트에 React.memo를 적용하면 된다.
=> 다만 함수/객체 등은 Object.is로 비교할 때 다르기 때문에 리렌더링 되어 방지가 제대로 되지 않는다.
=> 이를 useMemo로 해결할 수 있다. (의존성 배열이 변경되지 않는 한 리렌더링 X)

React.memo: 업데이트 전후 props가 동일하다면 리렌더링 방지 (얕은 비교 사용)
useMemo: 의존성 배열이 변경되지 않으면 이전 값 유지

---

## Usage

### 📌 데이터 전달하기

props로만 데이터를 전달하면 props drilling이라는 문제가 발생한다.
A, B, C, D 순서로 자식 컴포넌트를 가진 부모 컴포넌트에서 D 컴포넌트에 데이터를 전달하려고 할 때, A, B, C를 모두 거쳐 데이터를 전달해야 한다면 개발 시 불편하고, 유지보수가 어려워진다.

```
// 기존 방식
function App() {
  return <GrandParent value="Hello World!" />;
}

function GrandParent({ value }) {
  return <Parent value={value} />;
}

function Parent({ value }) {
  return <Child value={value} />;
}

function Child({ value }) {
  return <GrandChild value={value} />;
}

function GrandChild({ value }) {
  return <Message value={value} />;
}

function Message({ value }) {
  return <div>Received: {value}</div>;
}
```

```
// useContext 사용
import { createContext, useContext } from 'react';
const MyContext = createContext();

function App() {
  return (
    <MyContext.Provider value="Hello World">
      <GrandParent />
    </MyContext.Provider>
  );
}

function GrandParent() {
  return <Parent />;
}

function Parent() {
  return <Child />;
}

function Child() {
  return <GrandChild />;
}

function GrandChild() {
  return <Message />;
}

function Message() {
  const value = useContext(MyContext);
  return <div>Received: {value}</div>;
}

export default App;
```

createContext로 context 객체를 생성한다.
provider로 생성한 context를 자식 컴포넌트에 전달한다.

### 📌 context를 통해 전달된 데이터 업데이트하기

context 업데이트 시 state를 사용한다.
부모 컴포넌트에 state 변수를 선언하고 현재 state를 context 값으로 provider에 전달한다.

provider 내부에서는 theme 값을 받게 된다.
provider에게 전달한 theme 값을 업데이트 하기 위해 setTheme을 호출하면 모든 자식 컴포넌트가 새로운 light 값으로 리렌더링된다.

```
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <Button onClick={() => {
        setTheme('light');
      }}>
        Switch to light theme
      </Button>
    </ThemeContext.Provider>
  );
}
```

context로 업데이트 하는 경우는 다양하다.

1. context를 통해 값 업데이트
   state 변수가 ThemeContext provider로 전달되고, 체크 여부에 따라 state가 업데이트 된다.
   즉 해당 값을 변경하면 이 context를 사용하는 모든 자식 컴포넌트가 리렌더링된다.

```
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <label>
        <input
          type="checkbox"
          checked={theme === 'dark'}
          onChange={(e) => {
            setTheme(e.target.checked ? 'dark' : 'light')
          }}
        />
        Use dark mode
      </label>
    </ThemeContext.Provider>
  )
}
...
```

2. context를 통해 객체 업데이트
   객체를 보관하는 currentUser라는 state 변수가 있다.
   { currentUser, setCurrentUser }를 하나의 객체로 결합하고 value={} 내부의 context로 전달한다.
   이렇게 하면 LoginButton 같은 자식 컴포넌트가 setCurrentUser를 호출할 수 있다.

```
import { createContext, useContext, useState } from 'react';

const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <CurrentUserContext.Provider
      value={{
        currentUser,
        setCurrentUser
      }}
    >
      <Form />
    </CurrentUserContext.Provider>
  );
}

function Form({ children }) {
  return (
    <Panel title="Welcome">
      <LoginButton />
    </Panel>
  );
}

function LoginButton() {
  const {
    currentUser,
    setCurrentUser
  } = useContext(CurrentUserContext);

  if (currentUser !== null) {
    return <p>You logged in as {currentUser.name}.</p>;
  }

  return (
    <Button onClick={() => {
      setCurrentUser({ name: 'Advika' })
    }}>Log in as Advika</Button>
  );
}
...
```

또한 다중으로 context를 사용하거나, reducer를 통해 확장하는 경우도 있다.
(참고: https://react-ko.dev/learn/scaling-up-with-reducer-and-context)

### 📌 contex 기본값 지정하기

React가 부모 트리에서 특정 context의 provider를 찾지 못할 경우 useContext가 반환하는 context 값은 해당 context를 생성할 때 지정한 기본값이 된다.

```
const ThemeContext = createContext(null);
const ThemeContext = createContext('light');
```

기본값은 state로 변경하는 게 아닌 이상 절대 변경되지 않는다.
null 대신 기본값으로 유의미한 값을들 사용하면 provider를 찾지 못해도 중단되지 않는다.

### 📌 리렌더링 최적화

context를 통해 객체와 함수를 포함한 모든 값을 전달할 수 있다.
이 때 주의사항에서 언급한 것과 같이 리렌더링 관련 성능 이슈가 발생할 수 있다.

아래 예제를 통해 성능 최적화를 고민해보자!

1. useCallback은 함수를, useMemo는 값을 메모이제이션한다. (이전 값을 저장하고 동일 입력이 주어진 경우 저장된 값을 재사용한다.)
2. useCallback으로 login 함수를 메모이제이션한다.
3. useMemo로 currentUser와 login 값을 메모이제이션한다.
4. AuthContext.Provider로 하위 컴포넌트에 contextValue를 제공한다.
5. 이 때 login 함수와 currentUser 값이 달라지지 않는 이상 리렌더링이 발생하지 않는다.

```
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

💡 login 함수는 useCallback으로 메모이제이션을 했는데 왜 또 useMemo로 감싸줄까?
=> useCallback은 MyApp 컴포넌트가 리렌더링 될때마다 새 함수가 생성되는 것을 방지한다. 현재 의존성 배열이 비어있기 때문에 myApp이 처음 마운트 될 때만 생성되게 되어 있다.
=> useMemo는 contextValue 객체를 메모이제이션한다. 즉 전자는 불필요한 함수 생성, 후자는 중복 계산을 방지하기 위함이다.

---

## TroubleShooting

### 🏳 기본값이 다른데도 계속 context에서 undefined가 나오는 경우

=> 트리에 value가 없을 수 있다. 또는 prop 명을 다른 것을 사용했을 수도 있다.
=> value로 지정하지 않으면 value={undefined}를 전달하는 것이 되어버린다.

```
// 🚩 Doesn't work: prop should be called "value"
<ThemeContext.Provider theme={theme}>
   <Button />
</ThemeContext.Provider>
```

---

참고
https://react-ko.dev/reference/react/useContext#reference
https://velog.io/@jminkyoung/TIL-6-React-Hooks-useContext-%EB%9E%80#context-%EB%9E%80
