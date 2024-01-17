# ✏️ useRef

## Reference

useRef(초기값)
컴포넌트의 최상위 레벨에서 useRef를 호출해 ref를 선언한다.
초기값에는 어떤 유형의 값이든 지정할 수 있으며 초기 렌더링 이후 무시된다.

```
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

useRef는 단일 프로퍼티를 가진 객체를 반환한다.
처음에는 전달한 초기값으로 설정되며 나중에 다른 값으로 바꿀 수 있다.
ref 객체의 JSX 노드의 ref 속성으로 리액트에 전달하면 리액트는 current 프로퍼티를 설정한다.

**아래의 렌더링 사례에서 useRef는 동일한 객체를 반환한다.**

1. 리액트에서 state를 직접적으로 바꾸는 것은 안되지만, ref.current 프로퍼티는 직접적으로 바꿀 수 있다.
2. 단, 렌더링 과정에 사용되는 객체는 변경 시키면 안된다.
3. ref.current 프로퍼티가 변경되어도 리액트는 컴포넌트를 다시 렌더링하지 않는다.

- 리액트는 ref를 JS 객체로 간주해 해당 객체의 변경 사항을 감지하지 못한다.
- 리액트는 가상 DOM의 변화를 통해 UI 업데이트를 수행한다. 즉, 컴포넌트의 상태나 프로퍼티가 변경되는 상황을 감지한다.
  **4. 초기화를 제외하고 렌더링 중에 ref.current를 쓰거나 읽어선 안된다. (추후 서술)**

5. Strict Mode에서는 컴포넌트 함수를 2번 호출해 의도치 않은 부작용이 있는지 확인한다.

- useRef를 사용하면 함수 컴포넌트가 렌더링 될 때마다 새로운 ref 객체를 생성하게 돼 Strict Mode에서 경고가 발생할 수 있다.
- 하지만 생성된 2개의 객체 중 하나는 버려지며 개발 환경에서만 동작되므로 production에서는 영향을 미치지 않는다.

---

## Usage

### ref로 값 참조하기

컴포넌트 최상위 레벨에서 useRef를 호출해 하나 이상의 ref를 선언한다.
useRef는 초기값으로 설정된 단일 current 프로퍼티가 있는 ref 객체를 반환한다.

```
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
```

다음 렌더링에서 useRef는 동일한 객체를 반환한다.
정보를 저장하고 나중에 읽을 수 있도록 current 속성을 변경할 수 있다.

**state와 유사해보이지만, ref를 변경해도 리렌더링을 하지 않는다.**
ref는 컴포넌트의 시각적 출력에 영향을 미치지 않는 정보를 저장하는데 적합하다.
나중에 ref에서 interval ID를 읽어 초기화 할 수 있다.

```
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

```
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

즉, ref를 사용하면

1. 렌더링할 때마다 재설정되는 일반 변수와 달리 리렌더링을 해도 정보를 저장할 수 있다.
2. 리렌더링을 야기하는 state 변수와 달리 변경해도 리렌더링이 발생하지 않는다.
3. 정보가 공유되는 외부 변수와 달리 각 컴포넌트에 로컬로 저장된다. (local scope)

**렌더링 중에 왜 ref.current를 쓰면 안되는 걸까?**
리액트의 렌더링은 동일한 입력에 대해 항상 동일한 결과를 갖도록 순수 함수처럼 동작해야 한다.
이를 통해 리액트는 변화를 예측하고, 가상 돔을 비교해 렌더링을 하는 프로세스이기 때문이다.

렌더링 중 ref.current를 변경하면 컴포넌트의 렌더링을 예측할 수 없게 된다.
해당 컴포넌트의 렌더링 사이클 동안 ref.current 값이 바뀌면 이전과 다른 출력이 생성된다.
리액트가 이러한 변화를 인지하지 못하고 올바르게 업데이트 할 수 없으며, 부작용이 생길 수 있다.

```
function MyComponent() {
  // ...
  // 🚩 Don't write a ref during rendering
  // 🚩 렌더링 중에 ref를 작성하지 마세요.
  myRef.current = 123;
  // ...
  // 🚩 Don't read a ref during rendering
  // 🚩 렌더링 중에 ref를 읽지 마세요.
  return <h1>{myOtherRef.current}</h1>;
}
```

대신 이벤트 핸들러나 Effect 훅에서 ref를 읽고 쓸 수 있다.
위의 경우는 컴포넌트의 렌더링 사이클과 별개로 실행되기 때문이다.

이벤트 핸들러는 사용자와의 상호 작용에 응답해 실행되는 콜백 함수다.
사용자의 입력에 따른 동작을 수행하기에 렌더링과 직접적인 연결이 없다.

```
function handleClick() {
  console.log(myRef.current); // ref 읽기
  myRef.current = 123; // ref 쓰기
}

return <button onClick={handleClick}>Click me</button>;
```

useEffect 훅은 컴포넌트의 렌더링 사이클 외부에서 비동기적인 작업을 수행할 때 사용된다.
ref를 컴포넌트 렌더링과 별개로 다룰 수 있다.
이 훅 안에서 ref를 읽거나 쓸 수 있으며 클린업 함수로 언마운트 시 리소스 해제/정리 시에도 쓸 수 있다.

```
useEffect(() => {
  console.log(myRef.current); // ref 읽기
  myRef.current = 456; // ref 쓰기

  return () => {
    // 클린업 함수: 컴포넌트가 언마운트될 때 실행됨
    console.log("Component unmounted");
  };
}, []); // 빈 배열은 componentDidMount와 같은 효과
```

### ref로 DOM 조작하기

리액트에는 ref를 사용해 DOM을 조작하는 내장 기능이 있다.

1. 초기값이 null인 ref 객체를 선언한다.
2. 해당 객체를 ref 속성으로 조작하려는 DOM 노드의 JSX에 전달한다.
3. 리액트가 DOM 노드를 생성, 화면에 그린 후 ref 객체의 current 프로퍼티를 DOM 노드로 설정한다.
4. DOM 노드에 접근해 메서드를 호출할 수 있다.
5. 노드가 화면에서 제거되면 리액트는 current 프로퍼티를 다시 null로 설정한다.

```
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
  // ...
  return <input ref={inputRef} />;

    function handleClick() {
    inputRef.current.focus();
  }
```

text input에 focus 두기, 이미지를 스크롤하기, 비디오를 재생하기 등 useRef로 다양하게 DOM을 조작할 수 있다.
(컴포넌트에 ref를 노출하는 것도 가능하나 이는 forwardRef를 참고하자.)

```
import { useRef } from 'react';

export default function CatFriends() {
  const listRef = useRef(null);

  function scrollToIndex(index) {
    const listNode = listRef.current;
    // This line assumes a particular DOM structure:
    // 다음 코드는 특정 DOM 구조를 가정합니다.
    const imgNode = listNode.querySelectorAll('li > img')[index];
    imgNode.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToIndex(0)}>
          Tom
        </button>
        <button onClick={() => scrollToIndex(1)}>
          Maru
        </button>
        <button onClick={() => scrollToIndex(2)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul ref={listRef}>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
            />
          </li>
        </ul>
      </div>
    </>
  );
}

```

### ref 콘텐츠 재생성 피하기

리액트는 초기에 ref 값을 한 번 저장하고 다음 렌더링부터 이를 무시한다.
하지만 호출 자체는 이후 모든 렌더링에서 여전히 이뤄지므로 낭비가 될 수 있다.
이 때 ref를 초기화 하면 ref의 재생성을 방지할 수 있다.

```
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

일반적으로 렌더링 중 ref.current를 읽고 쓰는 것은 허용되지 않지만,
이 경우 결과가 항상 동일하며 초기화 중에만 조건이 실행되어 충분히 예측할 수 있으므로 사용할 수 있다.

---

### 📌 useState와 useRef 비교하기

useState
목적: 컴포넌트의 상태 관리 (변경 시 리렌더링을 트리거 + UI 업데이트)
사용 사례: 유저 인터랙션에 의한 데이터 변경, **컴포넌트의 라이프 사이클에 관련된 데이터 저장**, UI 상태 관리 등

useRef
목적: 렌더링과 무관하게 데이터를 저장/유지함
사용 사례: 이전 값 저장, DOM 직접 조작, 렌더링과 독립적인 데이터 저장

---

출처: https://react-ko.dev/reference/react/useRef#avoiding-recreating-the-ref-contents
