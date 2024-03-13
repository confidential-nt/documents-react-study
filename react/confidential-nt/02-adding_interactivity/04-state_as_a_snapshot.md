# State as a Snapshot

## 들어가면서

state 변수는 일반 js 변수처럼 보일 수 있다. 하지만 state는 '스냅샷'처럼 동작한다.

state 변수를 변경하면 **기존의 state 변수는 변경되지 않고 리렌더링이 실행된다.**

## state를 설정하면 렌더링이 촉발됩니다

인터페이스가 이벤트에 반응하려면 state를 업데이트해야함. (이벤트에 반응하여 인터페이스가 직접 변경되는 것이 아님.)

아래와 같은 코드가 있다고 했을 때,

```javascript
import { useState } from "react";

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState("Hi!");
  if (isSent) {
    return <h1>Your message is on its way!</h1>;
  }
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        setIsSent(true);
        sendMessage(message);
      }}
    >
      <textarea
        placeholder="Message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

버튼을 클릭하면 다음과 같은 일이 발생

1. onSubmit 이벤트 핸들러가 실행됨.
2. setIsSent(true)가 isSent를 true로 설정하고 새 렌더링을 큐에 대기시킴
3. React는 새로운 isSent 값에 따라 컴포넌트를 다시 렌더링함

state와 렌더링의 관계를 자세히 살펴보자.

## 렌더링은 그 시점의 스냅샷을 찍습니다

**“렌더링”이란 React가 컴포넌트, 즉,함수를 호출한다는 뜻.**

**해당 함수에서 반환하는 JSX는 시간상 UI의 스냅샷과 같다. prop, 이벤트 핸들러, 로컬 변수는 모두 렌더링 당시의 state를 사용해 계산된다.**

사진이나 동영상 프레임과 달리 반환하는 **UI ‘스냅샷’은 대화형.** (상호작용 가능하다는 뜻인듯.)

여기에는 input에 대한 응답으로 어떤 일이 일어날지 지정하는 이벤트 핸들러와 같은 로직이 포함됨. 그러면 React는 이 스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러를 연결. 결과적으로 버튼을 누르면 JSX에서 클릭 핸들러가 발동됨.

React가 컴포넌트를 다시 렌더링할 때:

1. React가 함수를 다시 호출함.
2. 함수가 새로운 JSX 스냅샷을 반환. (렌더링 당시의 state 스냅샷을 사용하여 계산된 prop + 이벤트 핸들러 + 로컬 변수 포함)
3. 그러면 React가 반환한 스냅샷과 일치하도록 화면을 업데이트.

컴포넌트의 메모리로서 **state는 함수가 반환된 후 사라지는 일반 변수와 다름.** state는 실제로 함수 외부에 **마치 선반에 있는 것처럼 React 자체에 “존재”**

React가 **컴포넌트를 호출하면 특정 렌더링에 대한 state의 스냅샷을 제공.** 컴포넌트는 **해당 렌더링의 state 값을 사용해 계산된 새로운 props 세트와 이벤트 핸들러가 포함된 UI의 스냅샷을 JSX에 반환.**

state가 업데이트되면:

1. React에게 state를 업데이트 하도록 명령
2. React가 state값을 업데이트
3. React가 state 값의 스냅샷을 컴포넌트에 보냄

아래 코드가 어떻게 동작할 것 같은가?

```javascript
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          // 버튼을 한번 클릭할 때 마다 3번씩 증가할까?
          setNumber(number + 1);
          setNumber(number + 1);
          setNumber(number + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

“+3” 버튼을 한 번 클릭하면 어떤 일이 발생하는가? -> +3 만큼 증가하지 않고 +1 만큼 증가하더라...

**state를 설정하면 다음 렌더링에 대해서만 변경됨.** 첫 번째 렌더링에서 number는 0이었음. 따라서 해당 렌더링의 onClick 핸들러에서 setNumber(number + 1)가 호출된 후에도 number의 값은 여전히 0.

이 버튼의 클릭 핸들러가 React에게 지시하는 작업은 다음과 같음.

1. setNumber(number + 1): number는 0이므로 setNumber(0 + 1)입니다.
   - React는 다음 렌더링에서 number를 1로 변경할 준비를 합니다.
2. setNumber(number + 1): number는 0이므로 setNumber(0 + 1)입니다.
   - React는 다음 렌더링에서 number를 1로 변경할 준비를 합니다.
3. setNumber(number + 1): number는 0이므로 setNumber(0 + 1)입니다.
   - React는 다음 렌더링에서 number를 1로 변경할 준비를 합니다.

**setNumber(number + 1)를 세 번 호출했지만, 이 렌더링에서 이벤트 핸들러의 number는 항상 0이므로 state를 1로 세 번 설정함.(스냅샷) 이것이 이벤트 핸들러가 완료된 후 React가 컴포넌트안의 number 를 3이 아닌 1로 다시 렌더링하는 이유.**

## 시간 경과에 따른 state

아래 코드는 또 어떻게 동작할까?

```javascript
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          alert(number); // 첫번째 클릭 때 뭐가 표시될까? 0? 5?
        }}
      >
        +5
      </button>
    </>
  );
}
```

정답은 0이 출력된다. (스냅샷)

```javascript
setNumber(0 + 5);
alert(0);
```

하지만 아래와 같이 alert에 타이머를 설정하여 컴포넌트가 다시 렌더링된 후에만 발동하도록 하면 어떨까? 0? 5?

```javascript
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          setTimeout(() => {
            alert(number); // 0 ? 5?
          }, 3000);
        }}
      >
        +5
      </button>
    </>
  );
}
```

0이 표시되는 것을 알 수 있다.

아래와 같이 동작하기 때문.

```javascript
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

위와 같이 substitution method를 사용하면, alert에 전달된 state의 ‘스냅샷’을 볼 수 있음.

**React에 저장된 state는 alert가 실행될 때 변경되었을 수 있지만, 사용자가 상호작용한 시점에 state 스냅샷을 사용하는 건 이미 예약되어 있던 것.**

**state 변수의 값은 이벤트 핸들러의 코드가 비동기적이더라도 렌더링 내에서 절대 변경되지 않는다.** 해당 렌더링의 onClick 내에서, setNumber(number + 5)가 호출된 후에도 number의 값은 계속 0. **이 값은 컴포넌트를 호출해 React가 UI의 “스냅샷을 찍을” 때 “고정”된 값임.**

다음은 이벤트 핸들러가 타이밍 실수를 줄이는 방법을 보여주는 예. 아래는 5초 지연된 메시지를 보내는 양식. 이 시나리오를 상상해 보라.

1. “보내기” 버튼을 눌러 앨리스에게 “안녕하세요”를 보냄.
2. 5초 지연이 끝나기 전에 “받는 사람” 필드의 값을 “Bob”으로 변경.

alert에 어떤 내용이 표시될까? “앨리스에게 인사했습니다”라고 표시될까, 아니면 “당신은 밥에게 인사했습니다”라고 표시될까? 다음 코드를 실행해보라.

```javascript
import { useState } from "react";

export default function Form() {
  const [to, setTo] = useState("Alice");
  const [message, setMessage] = useState("Hello");

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{" "}
        <select value={to} onChange={(e) => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

앨리스에게 인사했습니다 가 표시된다.

React는 하나의 렌더링 이벤트 핸들러 내에서 state 값을 “고정”으로 유지.

**코드가 실행되는 동안 state가 변경되었는지 걱정할 필요가 없음**

하지만 다시 렌더링하기 전에 최신 state를 읽고 싶다면 어떻게 해야 할까? 다음 페이지에서 계속...
