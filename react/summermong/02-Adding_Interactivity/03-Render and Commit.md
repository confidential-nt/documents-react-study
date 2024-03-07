# ✏️ Adding Interactivity : Render and Commit

컴포넌트를 화면에 표시하기 전에 React에서 **렌더링**을 해야 한다.
React는 렌더링 촉발 -> 컴포넌트 렌더링 -> DOM에 커밋하는 3가지 단계로 UI를 요청하고 제공한다.

### 1. 렌더링 촉발

컴포넌트 렌더링은 2가지 경우에서 발생한다.

1. 컴포넌트의 첫 렌더링
   앱을 시작하기 위해서는 첫 렌더링을 촉발 시켜야 한다.
   대상 DOM 노드로 `createRoot` 호출 후 다음 컴포넌트로 render 메서드를 호출한다.

```
// Image.js
export default function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

```
// index.js
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Image />);
```

2. 컴포넌트의 state 또는 상위 요소 중 하나가 업데이트 된 경우

컴포넌트가 처음 렌더링 되면 set 설정자 함수로 state를 업데이트 해 추가 렌더링을 촉발 할 수 있다.
컴포넌트의 state가 업데이트 되면 자동으로 렌더링이 대기열에 추가된다.

### 2. 컴포넌트 렌더링

렌더링이 발생하면 React는 컴포넌트를 호출해 화면에 표시될 내용을 파악한다.
`렌더링 === React에서 컴포넌트를 호출하는 것`

첫 렌더링에서는 루트 컴포넌트를 호출하고, 이후 렌더링에서는 state 업데이트에 의해 렌더링이 발동된 함수 컴포넌트를 호출한다.

이 과정은 `재귀적`이다.
업데이트된 컴포넌트가 다른 컴포넌트 반환 -> 그 컴포넌트를 렌더링하고 또 컴포넌트 반환 -> ...
중첩된 컴포넌트가 더 이상 없고 React도 화면에 더 표시해야 할 내용을 다 알 때까지 반복된다.

### 3. DOM에 커밋

컴포넌트를 호출한 뒤 React는 DOM을 수정한다.

첫 렌더링의 경우 React는 appendChild() DOM API를 사용해 생성한 모든 DOM 노드를 화면에 표시한다.
이후 렌더링(리렌더링)의 경우 React는 필요한 최소한의 작업을 적용해 DOM이 최신 렌더링 출력과 일치하도록 한다.
React는 렌더링 간에 차이가 있는 경우에만 DOM 노드를 변경한다.

### 그외

렌더링이 완료되고 React가 DOM을 업데이트 한 후, 브라우저는 다시 화면을 그린다.
이 경우를 브라우저 렌더링이라고 하지만 공식 문서에서는 혼동을 피하고자 `페인팅`이라고 한다.

---

출처: https://react-ko.dev/learn/render-and-commit
