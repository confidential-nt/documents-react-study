# ✏ Escape Hatches : Manipulating the DOM with Refs

React는 렌더링 출력과 일치하도록 DOM을 자동으로 업데이트하기 때문에 컴포넌트가 조작할 필요가 없다. 하지만 때로는 노드에 focus 하거나, 스크롤 하거나, 크기와 위치를 측정하기 위해 React가 관리하는 DOM 요소에 접근해야 할 수도 있다. 이런 작업을 수행할 때 DOM 노드에 대한 ref가 필요하다.

### 노드에 대한 ref 가져오기

```
import { useRef } from 'react';

const myRef = useRef(null);
...

<div ref={myRef}>

...

myRef.current.scrollIntoView();
```

- useRef 훅은 current 프로퍼티가 포함된 객체를 반환한다.
- 처음에는 null이지만 React가 div에 대한 DOM 노드를 생성하면 React는 이 노드에 대한 참조를 myRef.current에 넣는다.
- 다음 이벤트 핸들러에서 DOM 노드에 접근하고 여기에 정의된 빌트인 브라우저 API를 사용할 수 있다.

```
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

### 다른 컴포넌트의 DOM 노드에 접근하기

```
import { useRef } from 'react';

function MyInput(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

MyInput 처럼 커스텀 컴포넌트에 ref를 넣으면 null이 반환되며 포커스가 맞지 않는다.
React는 자기 자식이어도 컴포넌트가 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않기 때문이다. (다른 컴포넌트의 DOM 노드를 수동으로 조작하면 코드는 취약해진다.)
대신 DOM 노드를 노출하기 원하는 컴포넌트에 해당 동작을 설정한다.
컴포넌트는 자신의 ref를 자식 중 하나에 전달할 수 있다.

```
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

1. <MyInput ref={inputRef} />는 제대로 작동하지 않는다.
2. MyInput 컴포넌트를 forwardRef를 사용해 선언하면 props 다음 두 번째 ref 인수에 위의 inputRef를 받도록 설정된다.
3. MyInput은 수신한 ref를 내부 <input>으로 전달한다.

```
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

- 디자인 시스템에서 버튼, input 같은 낮은 수준의 컴포넌트는 해당 ref를 DOM 노드로 전달하는 게 일반적이지만 form이나 List 같이 상위 수준의 컴포넌트는 DOM 구조에 대한 의존성을 피하기 위해 DOM 노드를 노출하지 않는다.

### React가 ref를 첨부할 때

React에서 모든 업데이트는 2단계로 나뉜다.

1. 렌더링 하는 동안 컴포넌트를 호출해 화면에 무엇이 표시되어야 하는지 파악한다.
2. 커밋하는 동안 DOM에 변경사항을 적용한다.

일반적으로 렌더링 중에는 ref에 접근하는 것을 지양한다.
커밋하는 동안 ref.current를 설정하면 기본값인 null에서 DOM이 업데이트된 직후 해당 DOM 노드로 재설정된다.
일반적으로 이벤트 핸들러에서 ref에 접근하며 작업을 수행할 특정 이벤트가 없다면 effect으로 접근할 수 있다.

### ref를 이용한 DOM 조작 사례

```
import { useState, useRef } from 'react';

export default function Counter() {
  const [show, setShow] = useState(true);
  const ref = useRef(null);

  return (
    <div>
      <button
        onClick={() => {
          setShow(!show);
        }}>
        Toggle with setState
      </button>
      <button
        onClick={() => {
          ref.current.remove();
        }}>
        Remove from the DOM
      </button>
      {show && <p ref={ref}>Hello world</p>}
    </div>
  );
}
```

DOM 요소를 수동으로 제거하고 setState를 사용해 다시 표시하려고 하니 충돌이 발생한다.
React가 관리하는 DOM 노드를 변경해버리면 충돌이 발생하기 때문이다.

그렇다고 아예 업데이트를 할 수 없는 것은 아니고, 안전하게 수정할 수 있는 경우가 존재하긴 한다. (정적 요소, 비어 있는 업데이트, 포털, 외부 라이브러리 등)

---

참고
https://react-ko.dev/learn/manipulating-the-dom-with-refs
