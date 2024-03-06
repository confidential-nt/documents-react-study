## Render and Commit

### 들어가면서

**컴포넌트를 화면에 표시하려면 리액트에서 렌더링을 해야함.**

이 과정을 이해하면 코드가 어떻게 실행되는지 이해할 수 있고, 리액트 렌더링 동작을 설명하는 데 도움이 됨.

다음과 같이 가정해보자.

컴포넌트 -> 주방에서 재료(컴포넌트 자신)로 요리를 만드는 요리사

리액트 -> 고객의 요청을 접수하고 그들의 주문을 가져오는 웨이터.

이렇게 가정했을 때, UI를 요청하고 제공하는 단계를 다음과 같은 3단계로 나눠볼 수 있음.

1. 렌더링 촉발 (손님의 주문을 주방으로 전달)
2. 컴포넌트 렌더링 (주방에서 주문 받기)
3. DOM에 커밋 (테이블에 주문한 요리 내놓기)

### Step 1: 렌더링을 촉발시킵니다

컴포넌트 렌더링이 일어나는 두가지 이유

1. 컴포넌트의 첫 렌더링인 경우
2. 컴포넌트의 state(또는 상위 요소 중 하나)가 업데이트된 경우

- 첫 렌더링

  - 앱을 시작하기 위해서는 첫 렌더링을 촉발시켜야
  - 대상 DOM 노드로 createRoot를 호출한 다음 컴포넌트로 render 메서드를 호출하면 됨.

    ```javascript
    import Image from "./Image.js";
    import { createRoot } from "react-dom/client";

    const root = createRoot(document.getElementById("root"));
    root.render(<Image />);
    ```

- state가 업데이트되면 리렌더링합니다
  - set (설정자) 함수로 state를 업데이트하여 추가 렌더링을 촉발시킬 수 있음
  - 컴포넌트의 state를 업데이트하면 자동으로 렌더링이 대기열에 추가됨.
  - 이것은 식당에서 손님이 첫 주문 이후에 갈증이 나거나 배고파져서 차, 디저트 등을 추가로 주문하는 모습으로 상상해 볼 수 있음.

### Step 2: React가 컴포넌트를 렌더링합니다.

렌더링을 촉발시키면, React는 컴포넌트를 호출하여 **화면에 표시할 내용을 파악.**

**“렌더링”은 React에서 컴포넌트를 호출하는 것**

- 첫 렌더링에서 React는 루트 컴포넌트를 호출
- 이후 렌더링에서 React는 state 업데이트에 의해 렌더링이 발동된 함수 컴포넌트를 호출

이 과정은 재귀적임. 업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 React는 다음으로 해당 컴포넌트를 렌더링하고 해당 컴포넌트도 컴포넌트를 반환하면 반환된 컴포넌트를 다음에 렌더링하는 방식.

중첩된 컴포넌트가 더 이상 없고 React가 화면에 표시되어야 하는 내용을 정확히 알 때까지 이 단계는 계속됨.

다음 예제에서 React는 Gallery()와 Image()를 여러 번 호출함.

```javascript
// index.js
import Gallery from "./Gallery.js";
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(<Gallery />);

// Gallery.js

export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

**첫 렌더링을 하는 동안 React는 \<section>, \<h1> 그리고 3개의 \<img> 태그에 대한 DOM 노드를 생성**

리렌더링하는 동안 React는 이전 렌더링 이후 변경된 속성을 계산. **다음 단계인 커밋 단계까지는 해당 정보로 아무런 작업도 수행하지 않음.**

- 참고: 렌더링은 항상 순수한 계산이어야 함.

  - 동일한 입력에는 동일한 출력을 해야함.동일한 입력이 주어지면 컴포넌트는 항상 동일한 JSX를 반환해야 함.
  - 자신의 일에만 신경 써야함. 렌더링 전에 존재했던 객체나 변수를 변경해서는 안됨.

    - 위반하는 예)

      ```javascript
      let guest = 0;

      function Cup() {
        // Bad: changing a preexisting variable!
        // 나쁨: 기존 변수를 변경합니다!
        guest = guest + 1;
        return <h2>Tea cup for guest #{guest}</h2>;
      }

      export default function TeaSet() {
        return (
          <>
            <Cup />
            <Cup />
            <Cup />
          </>
        );
      }
      ```

  - Strict Mode 사용을 통해 위반하는 경우를 검출해낼 수 있다.

- 참고: 성능 최적화
  - 업데이트된 컴포넌트 내에 중첩된 모든 컴포넌트를 렌더링하는 기본 동작은 업데이트된 컴포넌트가 트리에서 매우 높은 곳에 있는 경우 성능 최적화되지 않음. 성능 문제가 발생하는 경우 성능 섹션에 설명된 몇 가지 옵트인 방식으로 문제를 해결 할 수 있음. 성급하게 최적화하지 마!

### Step 3: React가 DOM에 변경사항을 커밋

컴포넌트를 렌더링(호출)한 후 React는 DOM을 수정.

- **초기 렌더링의 경우 React는 appendChild() DOM API를 사용**하여 생성한 모든 DOM 노드를 화면에 표시
- 리렌더링의 경우 React는 필요한 최소한의 작업(렌더링하는 동안 계산된 것!)을 적용하여 DOM이 최신 렌더링 출력과 일치하도록 함.

**React는 렌더링 간에 차이가 있는 경우에만 DOM 노드를 변경**

### 에필로그: 브라우저 페인트

렌더링이 완료되고 React가 DOM을 업데이트한 후 브라우저는 화면을 다시 그림.

이 단계를 “브라우저 렌더링”이라고 하지만 이 문서의 나머지 부분에서 혼동을 피하고자 “페인팅”이라고 부를 것.
