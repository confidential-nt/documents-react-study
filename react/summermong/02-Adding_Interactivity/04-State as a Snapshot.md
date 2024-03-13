# ✏️ Adding Interactivity : State as a Snapshot

state 변수는 일반 JS 변수처럼 보이지만 스냅샷처럼 동작한다.
따라서 state 변수를 설정해도 이미 갖고 있는 state 변수는 변경되는 대신 리렌더링이 실행된다.

### state를 설정하면 리렌더링이 발생한다

클릭같이 유저와의 상호작용으로 UI가 직접 변한다고 생각할 수 있지만, React는 state를 업데이트해서 UI를 업데이트한다. 즉, state를 설정하면 React는 리렌더링을 요청하고 이로 인해 업데이트가 진행되는 것이다.

```
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

위와 같은 코드가 있을 때, 버튼을 누르면 onSubmit 이벤트 핸들러가 실행된다.
setIsSent(true)가 되어 새 렌더링이 큐에 대기되고, React는 새로운 isSent 값에 따라 컴포넌트를 다시 렌더링한다.

### 렌더링은 그 시점의 스냅샷을 찍는다

렌더링이란 React가 컴포넌트(함수)를 호출한다는 뜻이다.
해당 함수에서 반환하는 JSX는 시간 상 UI의 스냅샷과 같아 **prop, 이벤트 핸들러, 로컬 변수 모두 렌더링 당시의 state를 사용해 계산한다.** 이는 특정 시점에 UI가 어떻게 나타나야 하는지에 대한 어떤 계획(DOM 트리를 어떻게 업데이트할지)과 같다. (React 컴포넌트를 화가라고 생각할 때, 이 화가는 그림(스냅샷)을 그린다. 장면이 변하면 화가가 새 그림을 그리듯이 React는 UI를 새로고침해 변경된 UI 상태와 동일하게 만든다.)

그래서 컴포넌트의 메모리로서 state는 함수 반환 후 사라지는 일반 변수와 달리 React에 존재한다. React가 컴포넌트를 호출하면 특정 렌더링에 대한 state의 스냅샷을 제공하고, 컴포넌트는 해당 렌더링의 state 값으로 계산된 새로운 props 세트와 이벤트 핸들러가 포함된 UI의 스냅샷을 JSX에 반환한다.

(\*\*3줄 요약

1. 리렌더링: 상태(state)가 업데이트되면 React는 해당 컴포넌트를 리렌더링한다.
2. 스냅샷 생성: 리렌더링 과정에서 현재 state를 기반으로 한 새로운 UI 스냅샷이 생성한다.
3. UI 업데이트: 이 새로운 UI 스냅샷을 이전의 스냅샷과 비교한 후 변경된 부분만을 찾아 실제 DOM을 업데이트하여 화면을 갱신한다.)

state 변수의 값은 이벤트 핸들러 코드가 비동기적이어도 렌더링 내에서 절대 변경되지 않는다.
(스냅샷을 찍을 때 고정된 값으로 진행되므로)

---

참고: https://react-ko.dev/learn/state-as-a-snapshot
