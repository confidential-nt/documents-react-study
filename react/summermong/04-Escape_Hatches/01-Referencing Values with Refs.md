# ✏ Escape Hatches : Referencing Values with Refs

컴포넌트가 특정 정보를 기억하되 새 렌더링은 촉발 하지 않게 하려 할 때, ref를 사용하면 된다.

### 컴포넌트에 ref 추가하기

React의 useRef 훅으로 컴포넌트에 ref를 추가하고, 참조할 초기값을 인자로 전달한다.

ref는 current라는 단일 프로퍼티를 가진 일반 JS 객체로 읽거나 설정할 수 있다.

```
import { useRef } from 'react';

const ref = useRef(0);
```

useRef는 { current: 0 } 이라는 객체를 반환한다.

ref.current 속성을 통해 해당 ref의 현재 값에 접근할 수 있다.
이 값은 의도적인 변이가 가능해 읽기/쓰기 모두 가능하다.
React가 추적하지 않아 단방향 데이터 흐름의 탈출구가 된다.

```
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```

위 코드는 버튼을 클릭할 때마다 ref.current가 증가하는 코드다.
여기서 ref는 현재 숫자를 가리키지만 어떤 값이든 가질 수 있다.
state와 달리 ref는 current 속성을 읽고 수정할 수 있는 **일반 JS 객체**다.

ref가 증가되어도 컴포넌트는 리렌더링되지 않는다.

### 스톱워치 만들기

ref와 state를 단일 컴포넌트로 결합해 스톱워치를 만들어보자.
Start를 누른 후 얼마나 시간이 지났는지 표시하려면 버튼을 누른 시점과 현재 시간을 추적해야 하는데 이 정보는 렌더링에 사용되므로 state를 유지해야 한다.

```
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```

### ref와 state의 차이점

**ref**

- { current : initialValue }를 반환함
- 변경 시 리렌더링 촉발 X
- mutable
- 렌더링 중에는 current 값을 읽거나 쓰지 않아야 함

```
import React, { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(null);

  // 컴포넌트가 렌더링되는 동안 interval을 사용하여 ref의 current 값을 업데이트
  intervalRef.current = setInterval(() => {
    console.log('Interval fired');
  }, 1000);

  // 렌더링 결과를 반환
  return <div>My Component</div>;
}
```

이런 식으로 컴포넌트가 렌더링 되는 동안 ref의 값이 업데이트 되면 React는 변경사항을 감지하지 못한다.

```
import React, { useRef, useEffect } from 'react';

function MyComponent() {
  const intervalRef = useRef(null);

  useEffect(() => {
    // 컴포넌트가 마운트되면서 실행
    intervalRef.current = setInterval(() => {
      console.log('Interval fired');
    }, 1000);

    // 컴포넌트가 언마운트될 때 실행
    return () => {
      clearInterval(intervalRef.current);
    };
  }, []); // 빈 배열을 전달하여 마운트와 언마운트 시에만 실행되도록 함

  return <div>My Component</div>;
}
```

따라서 이렇게 useEffect에서 사용하거나 컴포넌트 외부에서 사용해야 한다.

state

- useState(initialValue)는 state 변수의 현재값과 state 설정자 함수를 반환함
- 변경 시 리렌더링 촉발
- immutable
- 언제든지 state를 읽을 수 있음 (각 렌더링에는 변경되지 않는 자체 state snapshot이 있음)

state와 ref를 각각 사용해 카운터 버튼을 만들면 후자의 경우 제대로 동작하지 않는 것을 확인할 수 있다.

```
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      You clicked {count} times
    </button>
  );
}
```

```
import { useRef } from 'react';

export default function Counter() {
  let countRef = useRef(0);

  function handleClick() {
    // This doesn't re-render the component!
    countRef.current = countRef.current + 1;
  }

  return (
    <button onClick={handleClick}>
      You clicked {countRef.current} times
    </button>
  );
}
```

### ref를 사용해야 하는 경우

일반적으로 ref는 컴포넌트가 외부 API(컴포넌트에 영향을 주지 않는)등과 통신할 때 사용된다.

또는 timeout ID를 저장하거나, DOM 요소를 저장 및 조작, JSX 계산 시 필요하지 않은 다른 객체를 저장하는 용도로 사용된다.

즉 일부 값을 저장해야 하지만 렌더링 로직에 영향을 미치지 않는 경우에 ref를 사용하면 된다.

### ref 모범 사례

1. ref를 탈출구 취급 해라
2. 렌더링 중에는 ref.current를 읽거나 쓰지 마라
   렌더링 중에 일부 정보가 필요하다면 state를 사용해라. React는 ref.current가 언제 변경되는지 알지 못하기에 렌더링 중에 읽어도 변화를 제대로 인지하지 못한다. (ref는 현재 값을 변이하면 즉시 변경됨 / state는 스냅샷처럼 작동해 동기적으로 업데이트됨) => ref 자체가 일반 JS이기에 그렇게 동작해서 그렇다.

### Ref와 DOM

ref의 가장 보편적인 사용 사례는 DOM 요소에 접근하는 것이다.
(예를 들어 input에 focus를 맞추는 경우)
