# ✏️ forwardRef

## Reference

forwardRef(render)
컴포넌트가 ref를 받아 자식 컴포넌트로 전달하려면 forwardRef()를 호출하면 된다.

```
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  // ...
});
```

이전 버전의 React에서는 **React.forwardRef()**로 사용되었지만, 현재는 forwardRef를 import하는 것이 일반적이다.

### 매개변수

매개변수로 쓰이는 render는 컴포넌트의 렌더링 함수다.
React는 컴포넌트가 부모로부터 받은 props와 ref를 가지고 이 함수를 호출한다.
반환하는 JSX는 컴포넌트의 결과물이 된다.

### 반환값

forwardRef는 JSX에서 렌더링할 수 있는 React 컴포넌트를 반환한다.
일반 함수로 정의된 React 컴포넌트와 달리 forwardRef가 반환하는 컴포넌트는 ref prop을 받을 수도 있다.

### render function

forwardRef는 렌더링 함수를 인자로 받는다.
React는 props, ref와 함께 이 함수를 호출한다.

```
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```

### 매개변수

props: 부모 컴포넌트가 전달한 props
ref: 부모 컴포넌트가 전달한 ref 속성으로 객체나 함수일 수 있다. 부모 컴포넌트가 전달하지 않은 경우 null이 된다. 받은 ref를 다른 컴포넌트에 전달하거나 **useImperativeHandle**에 전달해야 한다.

### 반환값

forwardRef는 JSX에서 렌더링할 수 있는 React 컴포넌트를 반환한다.
일반 함수로 정의된 React 컴포넌트와 달리 forwardRef가 반환하는 컴포넌트는 ref prop을 받을 수도 있다.

---

## Usage

### 부모 컴포넌트에 DOM 노드 노출하기

기본적으로 각 컴포넌트의 DOM 노드는 비공개지만 포커싱을 허용하기 위해 부모 컴포넌트에 DOM 노드를 노출해야 한다. 이 때 컴포넌트 정의를 forwardRef()로 감싸면 된다.
props 다음 두 번째 인자로 ref를 받고, 노출하려는 DOM 노드에 전달한다.

```
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} />
    </label>
  );
});
```

아래 코드와 같이 접근하면 부모 Form 컴포넌트가 MyInput에 의해 노출된 <input> DOM 노드에 접근할 수 있다.

```
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

이 Form 컴포넌트는 MyInput에 대한 ref를 전달한다.
MyInput 컴포넌트는 해당 ref를 <input> 브라우저 태그에 전달한다.
결과적으로 Form 컴포넌트는 <input> DOM 노드에 접근, 해당 노드에서 focus()를 호출할 수 있다.

컴포넌트 내부의 DOM 노드에 대한 ref를 노출하면 유지보수가 어려워질 수 있다.
(내부 DOM 구조 변경 시 외부에서 의존하고 있는 코드들도 변경해야 하기 때문)

### 여러 컴포넌트를 통해 ref 전달하기

ref를 DOM 노드로 전달하지 않고 MyInput 같은 커스텀 컴포넌트로 전달할 수 있다.
아래는 예제의 일부분으로 Form 컴포넌트가 ref를 정의한 뒤 FormField에 전달한다.
FormField 컴포넌트는 해당 ref를 MyInput으로 전달하고, ref가 <input> DOM 노드로 전달된다.
이런 식으로 Form이 해당 DOM 노드에 접근할 수 있게 된다.

```
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label="Enter your name:" ref={ref} isRequired={true} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

### DOM 노드 대신 명령형 핸들 노출하기

전체 DOM 노드를 노출하는 것보다 제한된 메서드 집합을 사용해 **명령형 핸들**이라는 사용자 정의 객체를 노출할 수 있다.
이것을 사용하려면 DOM 노드를 보유할 별도의 ref를 정의해야 한다.

```
const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  // ...

  return <input {...props} ref={inputRef} />;
});
```

받은 ref를 useImperativeHandle에 전달하고 ref에 노출할 값을 지정한다.

```
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```

일부 컴포넌트가 MyInput에 대한 ref를 받으면 DOM 노드 대신 { focus, scrollIntoView } 객체만 받는다.
이를 통해 DOM 노드에 노출하는 정보를 최소한으로 제한할 수 있다.

📌 선언적인 방식(Declarative)와 명령적인 방식(Imperative)
선언적인 방식은 '어떤 결과'를 원하는지 명시하며 가독성이 높다. JSX를 통한 UI 작성이나 컴포넌트의 상태 및 프로퍼티에 주로 사용된다.
명령적인 방식은 '어떻게 수행'되어야 하는지를 명시하며 시스템의 상태와 흐름을 직접 제어한다. DOM 조작이나 이벤트 핸들링 등에 주로 사용된다.
React는 주로 선언적인 방식을 채택해 UI를 작성, 관리하고 있다.

📌 ref를 과도하게 사용하면 안된다.
ref는 주로 명령형(imperative)인 작업을 수행할 때 주로 사용된다.
이런 동작은 DOM 조작이나 외부 라이브러리에서 ref로 특정 동작을 수행하는 경우와 같이, 일반적으로 어떤 순서나 상태에 따라 직접적인 제어를 필요로 하는 작업을 말한다.
명령형 작업은 선언적인 React 스타일과 상반되며, 유지보수를 어렵게 만들 수 있다.

📌 커스텀 컴포넌트는 ref 속성이 존재하지 않아 forwardRef로 랩핑해 ref를 전달해줘야 한다.

---

출처: https://react-ko.dev/reference/react/forwardRef#exposing-a-dom-node-to-the-parent-component
