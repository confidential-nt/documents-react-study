# ✏️ Adding Interactivity : Queueing a Series of State Updates

state 변수를 설정하면 다음 렌더링이 대기열 큐에 들어간다.
하지만 경우에 따라 다음 렌더링을 큐에 넣기 전에 값에 대해 여러 작업을 수행하고 싶을 수도 있다.
이 때 React가 state 업데이트를 어떻게 배치하면 좋을까?

### state 업데이트 일괄 처리

```
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

위 코드를 보면 setNumber(number + 1)을 3번 호출한다.
렌더링에서 state의 값은 0으로 고정되어 있기 때문에 setNumber(0+1), 즉 setNumber(1)을 3번 반복하는 것 뿐이다.

**React는 state를 업데이트 하기 전 이벤트 핸들러의 모든 코드가 실행될 때까지 기다린다.** 때문에 리렌더링은 모든 setNumber() 호출이 완료된 후 일어난다.

이렇게 해야 너무 많은 리렌더링이 발생하지 않고, 여러 컴포넌트에서 나온 다수의 state 변수를 업데이트할 수 있다. => 하지만 이 말은 곧 이벤트 핸들러와 그 안의 코드가 완료될 때까지 UI가 업데이트 되지 않는다는 말이 된다.

일괄처리(batching)라고도 하는 이 동작으로 React는 훨씬 빠르게 실행되며 일부 변수만 업데이트된, 혼란스러운 렌더링을 처리하지 않아도 된다.

### 다음 렌더링 전 동일한 state를 여러 번 업데이트 하기

위에 예시로 들었던 +3 코드는 아래와 같이 작성되었을 때 +3이 된다.

```
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

1. setNumber(n => n + 1): n => n + 1 함수를 큐에 추가하는 과정을 3번 반복한다.
2. n은 0에서 1로, 1에서 2로 업데이트되고 마지막에는 3이라는 최종 결과를 저장, 반환한다.

### 네이밍 컨벤션

일반적으로 업데이터 함수 인수의 이름은 해당 state 변수의 첫 글자로 지정한다.

```
setEnabled(e => !e);
setEnabled(enabled => !enabled)
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

### 요약

1. state를 설정해도 기존 렌더링의 변수는 변경되지 않고 대신 새로운 렌더링을 '요청'한다.
2. React는 이벤트 핸들러가 실행을 마친 후 state를 업데이트하는데 이를 일괄처리라고 한다.
3. 하나의 이벤트에서 일부 state를 여러 번 업데이트 하려면 업데이터 함수를 사용하면 된다.

---

참고
https://react-ko.dev/learn/queueing-a-series-of-state-updates
