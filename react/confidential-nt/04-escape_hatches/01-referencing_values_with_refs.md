## Referencing Values with Refs

### 들어가면서

컴포넌트가 어떤 정보를 저장하기를 원하지만, 그 정보가 새로운 렌더링을 트리거하지 않기를 원할 때, ref를 쓸 수 있다.

### 컴포넌트에 ref 추가하기

```javascript
const ref = useRef(0);
```

위와 같이 initial value를 넣어라. (숫자, 배열, 객체 등 어떤 것이든 된다.)

ref 안에 들어있는 것은 다음과 같은 일반 javascript 객체다.

```
{
  current: 0 // The value you passed to useRef
}
```

ref.current를 통해 현재의 값에 접근할 수 있으며, **읽고 쓸 수 있다.**

버튼을 클릭할 때 마다 ref.current의 값을 1씩 증가시킬 수 있다는 말이다.

그런데 ref는 리렌더링을 트리거 하지 않는다고 했다.

공식문서에서는 이러한 ref를 다음과 같이 표현.

> It’s like a secret pocket of your component that React doesn’t track. (This is what makes it an “escape hatch” from React’s one-way data flow)

또한 버추얼 돔의 비교과정에 포함되지 않음 -> 리렌더링 유발 x.

### 예제: 스톱워치 만들기

스톱워치 만들 때 setInterval을 사용하게 되는데, 얘를 멈추려면 clearInterval이 필요함. 그런데 interval ID가 있어야 setInterval를 멈출 수 있음. interval ID를 어딘가에 보관해야 함. interval ID는 렌더링에 사용되지 않으므로 ref에 보관할 수 있음.

렌더링에 정보가 사용되는 경우 해당 정보를 state로 유지하세요. 이벤트 핸들러만 정보를 필요로 하고 변경해도 다시 렌더링할 필요가 없는 경우에는 ref를 사용하는 것이 더 효율적일 수 있음.

### ref와 state의 차이

set 어쩌구 함수를 사용할 필요 없이 변경할 수 있다는 점에서 ref가 state보다 덜 '엄격'하다고 생각할 수 있음. 하지만 대부분의 경우 state를 사용하게 됨. 참조는 자주 사용하지 않는 '탈출구'일뿐.

| ref                                                                                                                                                                                                                                                                                                                                                                                                                                | state                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| { current: initialValue }를 리턴                                                                                                                                                                                                                                                                                                                                                                                                   | [value, setValue]를 리턴                                                                                                                                    |
| 바뀌었을 때 re-render 트리거 안함                                                                                                                                                                                                                                                                                                                                                                                                  | 바뀌었을 때 re-render 트리거 함                                                                                                                             |
| Mutable. 렌더링 프로세스에서 current 값을 직접 수정 가능.                                                                                                                                                                                                                                                                                                                                                                          | Immutable. set..어쩌구 함수를 사용해야 하며, shallow 비교를 통해 다른 참조값이 들어와야 다른 값이라고 인식을 하고 리렌더링 수행                             |
| 렌더링이 수행될 동안 값을 읽거나 쓰지 말아야 함. 정확한 이유는 잘 이해안되지만 뭐가 잘 안맞을 수가 있나봄. 경험 상 렌더링 시(해당 컴포넌트 함수의 모든 스코프 + JSX를 리턴하는 부분) ref를 사용하려고 했을 때, 내가 원하는 값이 아닌 null이나 undefined가 나온 적이 있는데 아마 이런 경우를 주의하라는 게 아닐까..좋은 관행은 useEffect나 이벤트 핸들러와 같은 렌더링 완료 후 실행되는 부분에서 useRef의 current 값을 읽고 쓰는 것 | 언제든지 상태를 읽을 수 있으나 각 렌더링에는 변경되지 않는 자체 상태 스냅샷(특정 시점에 컴포넌트의 상태(state)나 props를 "스냅샷"으로 캡처하는 개념)이 있음 |

<br/>
버튼 누르는 횟수를 기록하는 앱을 만든다고 했을 때, state 대신 ref.current를 통해 횟수를 표기할 경우, 횟수가 업뎃이 안됨. -> This is why reading ref.current during render leads to unreliable code

<br/>

- useRef는 내부적으로 어떻게 동작하는가?

  쉽게 생각하면 useState을 통해 구현된 것이라 생각하면 됨. initialValue를 인자로 받아 상태로 저장하고, 해당 상태를 return하는. 대신 state setter는 사용하지 않는 상태로 남겨두는. 왜나하면? useRef는 그냥 매 렌더링 때마다 **항상 같은 오브젝트**를 반환만 하기 때문에.

  > ... But you can think of it as a regular state variable without a setter.

  ```javascript
  // Inside of React
  function useRef(initialValue) {
    const [ref, unused] = useState({ current: initialValue });
    return ref;
  }
  ```

### 언제 ref를 사용해야 하는가

일반적으로 컴포넌트가 React를 "벗어나" 외부 API(주로 컴포넌트의 모양에 영향을 주지 않는 브라우저 API)와 통신해야 할 때 ref를 사용.

드문 몇 개의 사례

1. 타임아웃 ID 저장하기
2. DOM 요소 저장 및 조작하기
3. JSX를 계산하는 데 필요하지 않은 다른 객체 저장.

컴포넌트에 일부 값을 저장해야 하지만 렌더링 로직에 영향을 주지 않는 경우 ref를 선택하라.

### Best practices for refs

1. 참조를 탈출구로 취급하라. 참조는 외부 시스템이나 브라우저 API로 작업할 때 유용. 애플리케이션 로직과 데이터 흐름의 대부분이 참조에 의존한다면 접근 방식을 다시 생각해 볼 수 있음.
2. 렌더링 중에는 ref.current를 읽거나 쓰지 마라. **렌더링 중에 일부 정보가 필요하다면 state를 대신 사용하라.** React는 ref.current가 언제 변경되는지 모르기 때문에 렌더링 중에 읽어도 컴포넌트의 동작을 예측하기 어렵다. (유일한 예외는 첫 번째 렌더링 중에 ref를 한 번만 설정하는 if (!ref.current) ref.current = new Thing() 같은 코드. 예측가능하기 때문.)
3. ref.current는 state와는 달리 업뎃하는 즉시 업뎃된다. 자바스크립트 객체이기 때문에.

### ref 와 DOM

참조는 모든 값을 가리킬 수 있는데 가장 대표적인 사용 사례는 DOM이다. 프로그래밍 적으로 입력(들어오는 입력?)에 초점을 맞추고자 할 때 사용함.

```javascript
function Component() {
  return <div ref={myRef} />;
  // ...
}
```

React는 해당 DOM 엘리먼트를 myRef.current에 넣는다. 엘리먼트가 DOM에서 제거되면 React는 myRef.current를 null로 업데이트한다.

### 우아한 타입스크립트 with 리액트에서..

모든 렌더링 과정에서 객체의 참조를 동일하게 유지하고자 한다면 useRef를 사용해라. (모든 렌더링 과정에서 객체의 참조를 동일하게 유지하고자 한다는 말은, 해당 값이 시간이 지나도 절대로 변하지 않는 값이라는 의미이고, 즉, 상태로 관리되면 안된다는 의미이고, 그저 컴포넌트 내부의 상수로 관리해버리면 매번 새로운 객체를 참조하기 때문에...이런 경우에는 의미론적, 기능적으로 useRef를 사용하는 게 맞다.)

**useRef와 {current: ...} 객체를 직접 생성하는 방법 간의 유일한 차이: useRef는 매 렌더링 마다 동일한 ref 객체를 제공한다는 것.**

예시

```javascript
const store = useRef < Store > null;

if (!store.current) {
  store.current = new Store(); // 이렇게 하는 이유는 매 렌더링 마다 불필요한 인스탄스가 생성되지 않도록 하기 위하여.
}
```
