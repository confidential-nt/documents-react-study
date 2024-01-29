# ✏️ useState

## Reference

📌 useState(initialState)
컴포넌트의 최상위 레벨에서 useState를 호출, state 변수 선언
구조 분해 할당으로 state 변수 이름을 지정하는 것이 관례

```
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
```

📌 매개변수: 초기에 state를 설정할 값으로 모든 데이터 타입을 허용한다.
만약 함수를 전달한다면 순수해야 하며, 인자를 받지 않고, 반드시 어떤 값을 반환해야 한다.

📌 반환값: 2개의 값을 가진 '배열'을 반환한다.

1. state (첫 번째 렌더링 중에는 전달한 매개변수 initialState와 동일하다.)
2. set 함수 (state를 다른 값으로 업데이트, 리렌더링 할 수 있는 설정자 함수)
   이 때 다음 state를 직접 전달하거나 이전 state를 계산해 다음 state를 도출하는 함수를 전달할 수도 있다.

```
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
  // ...
```

📌 set 함수
매개변수: state가 될 값을 넣어주며, 모든 데이터 타입을 허용한다.

함수를 전달하면 **업데이터 함수**로 취급한다.
업데이터 함수는 순수해야 하며, 대기중인 state를 유일한 인수로 사용하고, 다음 state를 반환해야 한다.
React는 업데이터 함수를 대기열에 넣고 컴포넌트를 리렌더링하므로 다음 렌더링 중에 대기열의 업데이터를 이전 state에 적용해 다음 state를 계산한다. (후술)

반환값: 따로 없다.
특이사항

1. set 함수는 다음 렌더링에 대한 state 변수만 업데이트 한다.
2. set 함수를 호출해도 state 변수에는 호출 전 화면에 있던 이전 값이 담겨 있다! (후술2)
3. Object.is로 새로운 값과 현재 state가 동일한 경우 리렌더링을 하지 않는다. (최적화)
4. React는 state 업데이트를 일괄처리하므로 모든 이벤트 핸들러가 실행되고 set 함수를 호출한 후 화면을 업데이트한다.
   이렇게 해야 단일 이벤트 중 여러 번 리렌더링 되는 것을 방지한다.
5. 렌더링 도중 set 함수를 호출하는 것은 '현재 렌더링 중인 컴포넌트 내에서만' 허용된다. (이전 렌더링 정보 저장 시 사용)

📌 주의사항

1. useState는 Hook이므로 **컴포넌트의 최상위 레벨**이나 직접 만든 훅에서만 호출할 수 있다. (반복문, 조건문 내에서 호출 불가)
2. Strict Mode에서는 불순물 제거 목적으로 초기화 함수를 두 번 호출하나 프로덕션에서는 영향을 미치지 않는다.

---

## Usage

📌 컴포넌트에 state 추가하기
**set 함수를 호출해도 이미 실행중인 코드의 현재 state는 변경되지 않는다.**
set 함수는 다음 렌더링에서 반환할 useState에만 영향을 준다.

```
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');

  function handleClick() {
  setName('Robin');
  console.log(name); // Still "Taylor"!
                     // 아직 "Taylor"입니다!
}
```

📌 이전 state를 기반으로 state 업데이트하기
age가 42라고 가정할 때, handleClick 이벤트를 실행하면 누적합 되어 45이 되는 게 아니라 43이 된다.

```
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

**이를 해결하기 위해 setAge에 업데이터 함수를 전달할 수 있다.**
업데이터 함수는 대기중인 state를 가져와 다음 state를 계산한다.
React는 업데이터 함수를 큐에 넣고 다음 렌더링 중에 순서대로 호출한다.
따라서 42+1 => 43+1 => 44+1 을 순서대로 반환해 마지막에 45가 현재 state로 저장된다.

```
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

📌 객체 및 배열 state 업데이트
state에는 모든 타입의 값이 들어갈 수 있으므로 객체와 배열도 넣을 수 있다.
단, React에서 state는 읽기 전용으로 간주되므로 기존 객체를 '변이'하지 않고 '교체'해야 한다.

React에서는 상태를 업데이트 할 때 불변성을 유지해야 한다.
만약 상태를 직접 변경해버리면 상태 변화를 감지하기 어려워 렌더링 되지 않을 수 있다.
또 상태를 교체하면 변화를 감지하는 것 뿐만 아니라 필요한 부분만 렌더링 해 성능을 최적화 할 수 있다.

```
import { useState } from 'react';

export default function Form() {
  const [form, setForm] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com',
  });

  return (
    <>
      <label>
        First name:
        <input
          value={form.firstName}
          onChange={e => {
            setForm({
              ...form,
              firstName: e.target.value
            });
          }}
        />
      </label>
      <label>
        Last name:
        <input
          value={form.lastName}
          onChange={e => {
            setForm({
              ...form,
              lastName: e.target.value
            });
          }}
        />
      </label>
      <label>
        Email:
        <input
          value={form.email}
          onChange={e => {
            setForm({
              ...form,
              email: e.target.value
            });
          }}
        />
      </label>
      <p>
        {form.firstName}{' '}
        {form.lastName}{' '}
        ({form.email})
      </p>
    </>
  );
}
```

📌 key로 state 재설정하기
key 속성은 목록을 렌더링 할 때 뿐만 아니라 컴포넌트에 다른 key를 전달해 컴포넌트의 state를 재설정 할 수 있다.

React는 가상 DOM을 사용해 실제 DOM에 최소한의 변경만 반영하도록 최적화 되어 있다.
이 때 key는 가상 DOM이 엘리먼트를 식별하는데 사용된다. (key 변경 === 새로운 엘리먼트 => 리렌더링)
따라서 부모 컴포넌트가 자식 컴포넌트를 렌더링 할 때도 key가 변경된다.

즉 key가 변경되면 Form 컴포넌트와 그 자식 컴포넌트가 모두 재생성되므로 state가 초기화된다.

```
import { useState } from 'react';

export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      <Form key={version} />
    </>
  );
}

function Form() {
  const [name, setName] = useState('Taylor');

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <p>Hello, {name}.</p>
    </>
  );
}
```

📌 이전 렌더링에서 얻은 정보 저장하기
일반적으로 잘 사용하지 않지만 props의 변경에 따라 state를 변경하고 싶을 경우가 있을 때 사용한다.
카운터의 마지막 변경 이후 증감을 표시하려 한다면 이전 state 변수를 추적해 확인한다.

이 때 같지 않다는 조건과 조건문 내부 안에서 setPrevCount(count) 처럼 호출을 해야 한다.
그렇지 않으면 리렌더링을 반복하게 된다.

```
import { useState } from 'react';

export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);
  if (prevCount !== count) {
    setPrevCount(count);
    setTrend(count > prevCount ? 'increasing' : 'decreasing');
  }
  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}
```

**일반적으로 지양하는 것이 좋은 패턴이다.**
상태가 변경될 때마다 새로운 값을 설정해 상태를 업데이트하기 때문에 불필요한 리렌더링이 발생한다.
또 상태를 업데이트하는 로직이 컴포넌트에 '종속'되어 컴포넌트의 재사용성이 줄어들고 유지보수가 어려워진다.

따라서 함수형 업데이트나 useReducer로 상태를 업데이트하는 방법을 권장한다.

```
import { useState } from 'react';

export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);

  // 함수형 업데이트를 사용하여 상태를 업데이트
  const handleCountUpdate = (newCount) => {
    setPrevCount(newCount);
    setTrend((prevTrend) => (newCount > prevTrend ? 'increasing' : 'decreasing'));
  };

  if (prevCount !== count) {
    // 상태가 변경될 때 함수형 업데이트 호출
    handleCountUpdate(count);
  }

  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}
```

---

## TroubleShooting

🏳 ️state에 업데이트 했지만 로그에 이전 값이 표시되는 경우
**set 함수를 호출해도 실행중인 코드의 state는 변경되지 않는다.**

state는 스냅샷처럼 동작하기 때문에 state를 업데이트 하면 새로운 값으로 다음 렌더링을 요청할 뿐,
이미 실행중인 이벤트 핸들러의 state에는 영향을 미치지 않는다!

따라서 다음 state를 바로 써야 하는 경우에는 set 함수에 변수를 저장하거나 업데이터 함수를 전달한다.

🏳 state를 업데이트 해도 화면이 바뀌지 않는 경우
React는 Object.is 비교 결과 다음 state가 이전 state와 같으면 업데이트를 무시한다.
보통 객체나 배열의 state를 직접 변경할 때 발생된다.

**업데이트를 하기 위해서는 항상 객체나 배열의 state를 교체해야 한다.**

```
obj.x = 10;  // 🚩 Wrong: mutating existing object
             // 🚩 잘못된 방법: 기존 객체를 변이
setObj(obj); // 🚩 Doesn't do anything
             // 🚩 아무것도 하지 않습니다.
```

```
// ✅ Correct: creating a new object
// ✅ 올바른 방법: 새로운 객체 생성
setObj({
  ...obj,
  x: 10
});
```

🏳 리렌더링 횟수가 너무 많다는 에러가 발생할 경우
렌더링 중 state를 무조건적으로 설정하고 있다는 뜻으로 대개 이벤트 핸들러를 지정하는 과정에서 실수로 발생한다.
컴포넌트가 렌더링, state 설정(렌더링 유발), 렌더링, state 설정과 같은 루프에 들어가지는 않았나요?

```
// 🚩 Wrong: calls the handler during render
// 🚩 잘못된 방법: 렌더링 동안 핸들러 요청 (클릭이 발생하기 전 함수를 호출해 함수가 즉시 실행하게 된다.)
return <button onClick={handleClick()}>Click me</button>

// ✅ Correct: passes down the event handler
// ✅ 올바른 방법: 이벤트 핸들러로 전달
return <button onClick={handleClick}>Click me</button>

// ✅ Correct: passes down an inline function
// ✅ 올바른 방법: 인라인 함수로 전달 (추가적인 함수가 생성돼 성능에 영향을 미칠 수 있으므로 주의!)
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

인라인 함수를 사용하면 각 렌더링 시마다 새로운 함수가 생성되어 메모리에 부담을 줄 수 있다.
따라서 웬만하면 함수를 직접 전달하거나 필요 시 **useCallback 훅으로 함수를 메모이제이션하는 것이 좋다.**

```
function MyComponent({ onClick }) {
  return <button onClick={onClick}>Click me</button>;
}

// 렌더링되는 컴포넌트에서 handleClick을 인라인 함수로 전달
function ParentComponent() {
  const handleClick = () => {
    console.log('Button clicked!');
  };

  return <MyComponent onClick={() => handleClick()} />;
}
```

🏳 state의 값으로 함수를 설정했는데 설정 대신 호출이 되는 경우
=> state에 함수를 넣을 수 없다.

```
const [fn, setFn] = useState(someFunction); // ✅ useState(() => someFunction)

function handleClick() {
  setFn(someOtherFunction); // ✅ setFn(() => someFunction)
 }
```

함수를 값으로 전달하면 someFunction을 초기화 함수로 여기고 someOtherFunction을 업데이터 함수로 받아들인다.
따라서 이들을 호출해 결과를 저장하려고 하는데, 결과가 아닌 함수 자체를 저장하려면 앞에 () => 를 넣어준다.

---

출처: https://react-ko.dev/reference/react/useState#storing-information-from-previous-renders
