# Manipulating the DOM with Refs

## 들어가면서

React는 렌더링 출력과 일치하도록 DOM을 자동으로 업데이트하므로 컴포넌트를 자주 조작할 필요가 없음. 하지만 때로는 노드에 초점을 맞추거나 스크롤하거나 크기와 위치를 측정하기 위해 React가 관리하는 DOM 요소에 접근해야 할 수도 있음. React에는 이러한 작업을 수행할 수 있는 빌트인된 방법이 없으므로 DOM 노드에 대한 ref가 필요

## 노드에 대한 ref 가져오기

React가 관리하는 DOM 노드에 접근하려면, useRef hook import -> 컴포넌트 내부에서 ref를 선언(const myRef = useRef(null)) -> 마지막으로, DOM 노드를 가져올 JSX 태그에 ref 속성으로 참조를 전달(\<div ref={myRef}>)

이 useRef 훅은 current 라고 하는 프로퍼티가 포함된 객체를 반환. 처음에는 myRef.current 는 null 이 될 것. React가 이 \<div>에 대한 DOM 노드를 생성하면, React는 이 노드에 대한 참조를 myRef.current에 넣음. 그런 다음 이벤트 핸들러(렌더링 완료 후 실행되는 것)에서 이 DOM 노드에 액세스하고 여기에 정의된 빌트인 브라우저 API를 사용할 수 있음.

## 예: 텍스트 input에 초점 맞추기

```javascript
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

DOM 조작이 ref의 가장 일반적인 사용 사례. state와 유사하게 ref는 렌더링 사이에 유지됨. ref는 state 변수와 비슷하지만 설정할 때 리렌더링을 촉발하지 않음.

## 예: element로 스크롤하기

컴포넌트에 ref를 하나 이상 포함할 수도 있음. 이 예시에는 세 개의 이미지로 구성된 캐러셀이 있음. 각 버튼은 DOM 노드에서 브라우저 scrollIntoView() 메서드를 호출하여 해당 이미지를 중앙에 배치.

```javascript
import { useRef } from "react";

export default function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>Tom</button>
        <button onClick={handleScrollToSecondCat}>Maru</button>
        <button onClick={handleScrollToThirdCat}>Jellylorum</button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
              ref={firstCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
              ref={secondCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
              ref={thirdCatRef}
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

- 참고: ref 콜백을 사용하여 refs 목록을 관리하는 방법

  - 위의 예에서는 ref를 항목 수만큼 미리 정의해 둠. 그러나 목록의 각 항목에 대해 얼마나 많은 ref가 필요할지 알 수 없는 경우도 있음. 이런 경우에는 제대로 작동하지 않을 것.
    ```javascript
    <ul>
      {items.map((item) => {
        // Doesn't work! 작동하지 않습니다!
        const ref = useRef(null);
        return <li ref={ref} />;
      })}
    </ul>
    ```
  - **이는 훅은 컴포넌트의 최상위 레벨에서만 호출해야 하기 때문**. 반복문 또는 map() 내부에서는 useRef를 호출할 수 없음.
  - 이 문제를 해결할 수 있는 한 가지 방법은 부모 엘리먼트에 대한 단일 ref를 가져온 다음 querySelectorAll과 같은 DOM 조작 메서드를 사용하여 개별 하위 노드를 “찾는” 것. 하지만 이 방법은 DOM 구조가 변경되면 깨질 수 있음.
  - **또 다른 해결책은 ref 속성에 함수를 전달하는 것**. 이를 ref 콜백이라고 함. React는 ref를 설정할 때가 되면 DOM 노드로, 지울 때가 되면 null로 ref 콜백을 호출할 것. 이를 통해 자신만의 배열이나 Map을 유지 관리하고, 인덱스나 일종의 ID로 모든 ref에 접근할 수 있음.
  - 다음 예제는 이러한 접근으로 긴 목록에서 임의 노드로 스크롤하는 방법을 보여줌.

    ```javascript
    import { useRef } from "react";

    export default function CatFriends() {
      const itemsRef = useRef(null);

      function scrollToId(itemId) {
        const map = getMap();
        const node = map.get(itemId);
        node.scrollIntoView({
          behavior: "smooth",
          block: "nearest",
          inline: "center",
        });
      }

      function getMap() {
        if (!itemsRef.current) {
          // Initialize the Map on first usage.
          itemsRef.current = new Map();
        }
        return itemsRef.current;
      }

      return (
        <>
          <nav>
            <button onClick={() => scrollToId(0)}>Tom</button>
            <button onClick={() => scrollToId(5)}>Maru</button>
            <button onClick={() => scrollToId(9)}>Jellylorum</button>
          </nav>
          <div>
            <ul>
              {catList.map((cat) => (
                <li
                  key={cat.id}
                  ref={(node) => {
                    // 주목.
                    const map = getMap();
                    if (node) {
                      map.set(cat.id, node);
                    } else {
                      map.delete(cat.id);
                    }
                  }}
                >
                  <img src={cat.imageUrl} alt={"Cat #" + cat.id} />
                </li>
              ))}
            </ul>
          </div>
        </>
      );
    }

    const catList = [];
    for (let i = 0; i < 10; i++) {
      catList.push({
        id: i,
        imageUrl: "https://placekitten.com/250/200?image=" + i,
      });
    }
    ```

  - 이 예제에서 itemsRef는 단일 DOM 노드를 보유하지 않음. 대신 항목 ID에서 DOM 노드로의 Map을 보유. (ref는 모든 종류의 값을 보유할 수 있음)
  - 모든 목록 항목의 ref 콜백은 맵을 업데이트.
  - 이렇게 하면 나중에 Map에서 개별 DOM 노드를 읽을 수 있음.

## 다른 컴포넌트의 DOM 노드에 접근하기

\<input />과 같은 브라우저 엘리먼트를 출력하는 빌트인 컴포넌트에 ref를 넣으면, React는 해당 ref의 current 프로퍼티를 해당 DOM 노드(예: 브라우저의 실제 \<input />)로 설정.

그러나 \<MyInput />과 같은 직접 만든 컴포넌트에 ref를 넣으려고 하면 기본적으로 null이 반환됨. 그리고 이때, React는 콘솔에 다음과 같은 오류를 출력함.

```
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
// 경고: 함수 컴포넌트에는 refs를 지정할 수 없습니다. 이 ref에 접근하려는 시도는 실패합니다. React.forwardRef()를 사용하려고 하셨나요?
```

**이는 기본적으로 React가 컴포넌트가 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않기 때문.** 심지어 자신의 자식에게도. **이것은 의도적인 것**. ref는 탈출구이기 때문에 아껴서 사용해야. **다른 컴포넌트의 DOM 노드를 수동으로 조작하면 코드가 훨씬 더 취약해짐.**

대신, DOM 노드를 노출하길 원하는 컴포넌트에 해당 동작을 설정해야. 컴포넌트는 자신의 ref를 자식 중 하나에 “전달”하도록 지정할 수 있음. -> forwardRef

다음과 같이 사용.

```javascript
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

작동 방식은 다음과 같음.

1. \<MyInput ref={inputRef} />는 React에게 해당 DOM 노드를 inputRef.current에 넣으라고 지시합니다. 그러나 이를 선택할지는 MyInput 컴포넌트에 달려 있으며, 기본적으로 그렇지 않음.
2. MyInput 컴포넌트를 forwardRef를 사용하여 선언하면, props 다음의 두 번째 ref 인수에 위의 inputRef를 받도록 설정됨.
3. MyInput은 수신한 ref를 내부의 \<input>으로 전달함.

**디자인 시스템에서 버튼, 입력 등과 같은 저수준 컴포넌트는 해당 ref를 DOM 노드로 전달하는 것이 일반적인 패턴.** 반면 양식, 목록 또는 페이지 섹션과 같은 **상위 수준 컴포넌트는 일반적으로 DOM 구조에 대한 우발적 의존성을 피하기 위해** 해당 DOM 노드를 노출하지 않음.

- 참고: 명령형 핸들(imperative handle)로 API의 하위 집합 노출하기

  - 위의 예시에서 MyInput은 원본 DOM input 엘리먼트를 노출함.
  - 이를 통해 부모 컴포넌트가 이 요소에 focus()를 호출할 수 있음. 그런데 이렇게 하면 부모 컴포넌트가 다른 작업(예: CSS 스타일 변경)을 할 수도 있음. 드문 경우지만 노출되는 기능을 제한하고 싶을 수도 있을 것.
  - 그럴 땐 useImperativeHandle을 사용하면됨.

    ```javascript
    import { forwardRef, useRef, useImperativeHandle } from "react";

    const MyInput = forwardRef((props, ref) => {
      const realInputRef = useRef(null);
      useImperativeHandle(ref, () => ({
        // Only expose focus and nothing else
        focus() {
          realInputRef.current.focus();
        },
      }));
      return <input {...props} ref={realInputRef} />;
    });

    export default function Form() {
      const inputRef = useRef(null);

      function handleClick() {
        inputRef.current.focus();
      }

      return (
        <>
          <MyInput ref={inputRef} />
          <button onClick={handleClick}>Focus the input</button>
        </>
      );
    }
    ```

  - 여기서 MyInput 내부의 realInputRef는 실제 input DOM 노드를 보유.
  - 하지만 useImperativeHandle은 부모 컴포넌트에 대한 ref 값으로 고유한 특수 객체를 제공하도록 React에 지시.
  - 따라서 Form 컴포넌트 내부의 inputRef.current에는 focus 메서드만 있게됨. 이 경우 ref “핸들”은 DOM 노드가 아니라 useImperativeHandle() 내부에서 생성한 사용자 정의 객체.

## React가 ref를 첨부할 때

React에서 모든 업데이트는 두 단계로 나뉨.

- **렌더링** 하는 동안 React는 컴포넌트를 호출하여 화면에 무엇이 표시되어야 하는지 파악.
- **커밋(commit)** 하는 동안 React는 DOM에 변경 사항을 적용.

일반적으로 렌더링 중에는 ref에 액세스하는 것을 원하지 않을 것. DOM 노드를 보유하는 ref도 마찬가지. 첫 번째 렌더링 중에는 DOM 노드가 아직 생성되지 않았으므로 ref.current는 null이 됨. 그리고 업데이트를 렌더링하는 동안에는 DOM 노드가 아직 업데이트되지 않았음. 따라서 이를 읽기에는 너무 이른 것.

**React는 커밋하는 동안에 ref.current를 설정함. React는 DOM이 업데이트 되기 전에는 ref.current의 값을 null로 설정하였다가, DOM이 업데이트된 직후 해당 DOM 노드로 다시 설정함.**

일반적으로 이벤트 핸들러에서 ref에 접근함. ref로 무언가를 하고 싶지만 그 작업을 수행할 특정 이벤트가 없다면, Effect가 필요할 수 있음. Effect에 대해서는 다음 페이지에서 설명.

- 참고: 플러싱(flushing) state는 flushSync와 동기식으로 업데이트됨

  - 다음과 같이 새 할 일을 추가하고 목록의 마지막 하위 항목까지 화면을 아래로 스크롤하는 코드를 고려해 봐라. 어떤 이유에서인지 항상 마지막으로 추가한 할 일의 바로 앞에 있던 할 일로 스크롤되는 것을 볼 수 있음.

    ```javascript
    import { useState, useRef } from "react";

    export default function TodoList() {
      const listRef = useRef(null);
      const [text, setText] = useState("");
      const [todos, setTodos] = useState(initialTodos);

      function handleAdd() {
        const newTodo = { id: nextId++, text: text };
        setText("");
        setTodos([...todos, newTodo]);
        listRef.current.lastChild.scrollIntoView({
          behavior: "smooth",
          block: "nearest",
        });
      }

      return (
        <>
          <button onClick={handleAdd}>Add</button>
          <input value={text} onChange={(e) => setText(e.target.value)} />
          <ul ref={listRef}>
            {todos.map((todo) => (
              <li key={todo.id}>{todo.text}</li>
            ))}
          </ul>
        </>
      );
    }

    let nextId = 0;
    let initialTodos = [];
    for (let i = 0; i < 20; i++) {
      initialTodos.push({
        id: nextId++,
        text: "Todo #" + (i + 1),
      });
    }
    ```

  - 문제는 바로 다음 두 줄
    ```javascript
    setTodos([...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView();
    ```
  - React에서는 state 업데이트가 큐에 등록됨. 일반적으로 이것은 사용자가 원하는 것. 그러나 여기서는 setTodos가 DOM을 즉시 업데이트하지 않기 때문에 문제가 발생함. 따라서 목록을 마지막 요소로 스크롤할 때 할 일이 아직 추가되지 않은 상태. 이것이 스크롤이 항상 한 항목씩 “뒤처지는” 이유.
  - 이 문제를 해결하기 위해 React가 DOM을 동기적으로 업데이트(“플러시”)하도록 강제할 수 있음. 이렇게 하려면 react-dom에서 flushSync를 import하고 state 업데이트를 flushSync 호출로 감싸면 됨.

    ```javascript
    flushSync(() => {
      setTodos([...todos, newTodo]);
    });
    listRef.current.lastChild.scrollIntoView();
    ```

  - 이렇게 하면 flushSync로 감싼 코드가 실행된 직후 React가 DOM을 동기적으로 업데이트하도록 지시. 그 결과 스크롤을 시도할 때 DOM에 이미 마지막 할 일이 있을 것.

  ## ref를 이용한 DOM 조작 모범 사례

  Ref는 탈출구. “React 외부로 나가야” 할 때만 사용해야. **일반적인 예로는 초점을 맞추거나, 스크롤 위치를 관리하거나 React가 노출하지 않는 브라우저 API를 호출하는 것이 있음.**

  포커스나 스크롤 같은 비파괴적 동작을 고수한다면 문제가 발생하지 않을 것. **그러나 DOM을 수동으로 수정하려고 하면 React가 수행하는 변경 사항과 충돌할 위험이 있음**

  다음 예시에는 이 문제를 설명하기 위해 환영 메시지와 두 개의 버튼이 포함되어 있음. 첫 번째 버튼은 React에서 일반적으로 사용하는 것처럼 조건부 렌더링과 state를 사용하여 그 존재 여부를 전환. 두 번째 버튼은 remove() DOM API를 사용하여 React가 제어할 수 없는 'DOM에서 강제로 제거하기.'

  “Toggle with setState”를 몇 번 눌러봐라. 메시지가 사라졌다가 다시 나타날 것. 그런 다음 “Remove from the DOM”을 눌러라. 그러면 강제로 제거될 것입니다. 마지막으로 “Toggle with setState”를 눌러봐라.

  DOM 엘리먼트를 수동으로 제거한 후 setState를 사용하여 다시 표시하려고 하면 충돌이 발생함. **이는 사용자가 DOM을 변경했고 React가 이를 계속 올바르게 관리하는 방법을 모르기 때문.**

  **React가 관리하는 DOM 노드를 변경하지 마라.** React가 관리하는 요소를 수정하거나, 자식을 추가하거나 제거하면, 위와 같이 일관성 없는 시각적 결과나 충돌이 발생할 수 있음.

  그렇다고 전혀 할 수 없다는 건 아니고, 주의가 필요하다는 의미. **React가 업데이트할 이유가 없는 DOM의 일부는 안전하게 수정할 수 있음.** 예를 들어, **JSX에서 일부 \<div>가 항상 비어 있는 경우, React는 그 자식 목록을 건드릴 이유가 없습니다. 따라서 수동으로 요소를 추가하거나 제거하더라도 안전.**
