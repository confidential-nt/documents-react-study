# Queueing a Series of State Updates

## 들어가면서

state 변수를 설정하면 다음 렌더링이 큐에 들어감. 그러나 다음 렌더링을 큐에 넣기 전에 값에 대해 여러 작업을 수행하고 싶을 수 있음. 이를 위해서는 React가 state 업데이트를 어떻게 배치하는 지에 대해 이해하면 좋다.

## state 업데이트 일괄처리

지난 파트에서 등장한 아래의 코드를 기억할 것.

```javascript
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
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

이번에 알리고 싶은 내용은, **React는 state 업데이트를 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 기다린다는 것임.** -> 이 때문에 리렌더링은 모든 setNumber() 호출이 완료된 이후에만 일어남.

웨이터는 첫번째 요리를 말하자마자 주방으로 달려가지 않음, 대신 주문이 끝날 때까지 기다렸다가 주문을 변경하기도 하고, 심지어 테이블에 있는 다른 사람의 주문도 받음. 이것과 같음.

**이렇게 하면 너무 많은 리렌더링을 촉발하지 않으면서도 다수의 state 변수를 업뎃 할 수 있음.** 하지만 이는 이벤트 핸들러와 그 안에 있는 코드가 완료될 때까지 UI가 업데이트되지 않는다는 의미이기도 함. **일괄처리(배칭, batching)라고도 하는 이 동작은 React 앱을 훨씬 빠르게 실행할 수 있게 해줌. 또한 일부 변수만 업데이트된 “반쯤 완성된” 혼란스러운 렌더링을 처리하지 않아도 됨.**

단, React는 클릭과 같은 여러 의도적인 이벤트에 대해 일괄 처리하지 않으며, 각 클릭은 개별적으로 처리됨. React는 일반적으로 안전한 경우에만 일괄 처리를 수행하니 안심할 것. e.g) 첫 번째 버튼 클릭으로 양식이 비활성화되면 두 번째 클릭으로 양식이 다시 제출되지 않도록 보장함

## 다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트하기

다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트 하고 싶다면 setNumber(number + 1) 와 같은 **다음 state 값**을 전달하는 대신, setNumber(n => n + 1) 와 같이 **큐의 이전 state를 기반**으로 다음 state를 계산하는 함수를 전달할 수 있음.

이는 단순히 state 값을 대체하는 것이 아니라 React에게 “state 값으로 무언가를 하라”고 지시하는 방법

예시)

```javascript
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

위 버튼을 클릭하면 +3이 증가한다!

여기서 n => n + 1 는 **업데이터 함수(updater function)**라고 부름.

이를 state 설정자 함수에 전달 할 때:

1. React는 이벤트 핸들러의 다른 코드가 모두 실행된 후에 이 함수(업데이터 함수)가 처리되도록 큐에 넣는다.
2. 다음 렌더링 중에 React는 큐를 순회하여 최종 업데이트된 state를 제공한다.

즉, 과정을 더 자세히 풀어서 설명하자면,

1. setNumber(n => n + 1): n => n + 1 함수를 큐에 추가합니다.
2. setNumber(n => n + 1): n => n + 1 함수를 큐에 추가합니다.
3. setNumber(n => n + 1): n => n + 1 함수를 큐에 추가합니다.
4. 다음 렌더링 중에 useState 를 호출하면 React는 큐를 순회.
5. 이전 number state는 0이었으므로 React는 이를 첫 번째 업데이터 함수에 n 인수로 전달. 그런 다음 React는 이전 업데이터 함수의 반환값을 가져와서 다음 업데이터 함수에 n 으로 전달하는 식으로 반복.
6. React는 3을 최종 결과로 저장하고 useState에서 반환.
7. 이것이 위 예제 “+3”을 클릭하면 값이 3씩 올바르게 증가하는 이유.

## state를 교체한 후 업데이트하면 어떻게 될까요

```javascript
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

이 이벤트 핸들러는 어떨까? 다음 렌더링에서 number가 어떻게 될까?

이 이벤트 핸들러가 React에게 지시하는 작업은 다음과 같다:

1. setNumber(number + 5) : number는 0이므로 setNumber(0 + 5)입니다. React는 큐에 “*5*로 바꾸기”를 추가함.
2. setNumber(n => n + 1) : n => n + 1 는 업데이터 함수임. React는 해당 함수를 큐에 추가함.
3. 다음 렌더링 동안 React는 state 큐를 순회함.

| queued update    | n          | returns   |
| ---------------- | ---------- | --------- |
| ”replace with 5” | 0 (unused) | 5         |
| n => n + 1       | 5          | 5 + 1 = 6 |

**React는 6을 최종 결과로 저장하고 useState에서 반환함.**

참고: setState(x)와 setState(n => x)는 결국 같은 동작을 하지만 다만 n이 직접적으로 사용되지 않는 것.

## 업데이트 후 state를 바꾸면 어떻게 될까요

다음 렌더링에서 number가 어떻게 될까?

```javascript
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

이 이벤트 핸들러를 실행하는 동안 React가 이 코드를 통해 작동하는 방식은 다음과 같다:

1. setNumber(number + 5): number 는 0 이므로 setNumber(0 + 5)이다. React는 *“5로 바꾸기”*를 큐에 추가함.
2. setNumber(n => n + 1): n => n + 1 는 업데이터 함수다. React는 이 함수를 큐에 추가함.
3. setNumber(42): React는 *“42로 바꾸기”*를 큐에 추가함.

다음 렌더링 동안, React는 state 큐를 순회:
| queued update | n | returns |
| ---------------- | ---------- | --------- |
| ”replace with 5” | 0 (unused) | 5 |
| n => n + 1 | 5 | 5 + 1 = 6 |
| ”replace with 42”| 6 (unused) | 42 |

그런 다음 React는 42를 최종 결과로 저장하고 useState에서 반환

요약하자면, setNumber state 설정자 함수에 전달할 내용은 다음과 같이 생각할 수 있음:

- 업데이터 함수 (예: n => n + 1)가 큐에 추가됨.
- 다른 값 (예: 숫자 5)은 큐에 “5로 바꾸기”를 추가하며, 이미 큐에 대기중인 항목은 무시.

이벤트 핸들러가 완료되면 React는 리렌더링을 실행. 리렌더링하는 동안 React는 큐를 처리. **업데이터 함수는 렌더링 중에 실행되므로, 업데이터 함수는 순수해야 하며 결과만 반환해야 함.** (당연함. 결과가 예상치 못하게 동작하면 안됨. 컴포넌트가 이상해짐. 동일 입력에 동일 결과를 반환하지 못하게 될 거임.) 업데이터 함수 내부에서 state를 변경하거나 다른 사이드 이팩트를 실행하려고 하지마라. Strict 모드에서 React는 각 업데이터 함수를 두 번 실행(두 번째 결과는 버림)하여 실수를 찾을 수 있도록 도와줌.

## 명명 규칙

업데이터 함수 인수의 이름은 해당 state 변수의 첫 글자로 지정하는 것이 일반적

```javascript
setEnabled((e) => !e);
setLastName((ln) => ln.reverse());
setFriendCount((fc) => fc * 2);
```

좀 더 자세한 코드를 선호하는 경우 setEnabled(enabled => !enabled)와 같이 전체 state 변수 이름을 반복하거나, setEnabled(prevEnabled => !prevEnabled)와 같은 접두사(prefix “prev”)를 사용하는 것이 일반적인 규칙
