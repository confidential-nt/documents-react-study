# ✏️ Describing the UI : Keeping Components Pure

일부 JS 함수는 '순수'하다.
순수 함수란 동일한 매개변수(입력)가 주어질 때 항상 동일한 결과를 반환하는 함수를 말한다.
더 자세히 말하면 함수의 입력 외에 함수의 결과에 영향을 미치는 사이드 이펙트가 없는 함수로,
순수 함수로만 코드를 작성하면 의도하지 않은 동작이나 버그를 줄일 수 있기 때문에 JS에서는 이러한 순수함수를 만들기 위해 데이터 불변성을 유지하는 것이 중요하다.

순수함수는 다음과 같은 특징이 있다.

1. 동일 입력 동일 출력
2. 호출되기 전 존재했던 객체나 변수를 변경하지 않는다.

2번에 위배되는 코드로 아래 예시를 들 수 있다.

```
let guest = 0;

function Cup() {
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup /> // Tea cup for guest #2
      <Cup /> // Tea cup for guest #4
      <Cup /> // Tea cup for guest #6
    </>
  );
}
```

위 코드는 함수 내부에서 guest라는 외부 변수를 변경하고 있다.
순수 함수는 오직 매개변수에 의해서만 결과가 결정되어야 하는데, 외부 변수 guest를 변경함에 따라 함수의 결과가 달라진다.

이를 아래와 같이 수정할 수 있다.
guest를 prop을 전달한 것으로 수정한다.

```
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} /> // Tea cup for guest #1
      <Cup guest={2} /> // Tea cup for guest #2
      <Cup guest={3} /> // Tea cup for guest #3
    </>
  );
}
```

---

### StrictMode

React에서는 렌더링하는 동안 읽을 수 있는 입력이 3가지 있다. (props, state, context)
이 입력들은 항상 읽기 전용으로 취급되어야 한다.

사용자 입력에 대한 응답으로 어떤 것을 변경하려면 변수 대신 state로 변경해야 한다.
컴포넌트가 렌더링되는 동안 기존의 변수나 객체를 변경하면 안된다.

Strict Mode는 각 컴포넌트의 함수를 2번 호출함으로서 이런 규칙을 위반하는 컴포넌트를 찾아낸다.

---

### 지역 변이

2번 특징을 위배한 경우를 '변이'라고 한다.
순수 함수는 함수 이전의 변수나 객체를 변이시키지 않는다.
하지만 렌더링하는 동안 방금 생성한 변수나 객체를 변경하는 것은 가능하다.

```
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

let cups를 TeaGathering 함수의 외부에 선언하고, strict mode를 실행하면 콘솔에 에러가 발생한다.

---

함수형 프로그래밍은 순수성에 크게 의존한다. (이에 따라 함수 결과값이 달라질 수 있으므로)
하지만 언젠가는 어디에서 무언가 변경되어야 하는데, 이러한 변경(사이드 이펙트)은 '부수적으로' 일어나야 한다.
React에서 사이드 이펙트는 보통 이벤트 핸들러에 속한다.
이벤트 핸들러는 유저가 어떤 동작을 수행할 때 React가 실행하는 함수로, 이 역시 컴포넌트 내부에 정의되어 있지만 렌더링 중에는 실행되지 않는다.

만약 사이드 이펙트에 적합한 이벤트 핸들러를 찾을 수 없다면 useEffect 호출로 사용할 수 있다.
다만 이는 최후의 수단으로 사용해야 하며 가능한 렌더링으로 표현하는 것이 좋다.

왜?

1. 가독성과 유지보수: 일반적으로 컴포넌트 내부에 이벤트 핸들러를 직접 선언하는 것이 가독성과 유지보수에 도움이 된다.
2. React의 철학: React는 상태와 속성을 통한 선언적 프로그래밍을 강조한다. 상태나 속성이 변경될 때 컴포넌트의 렌더링이 트리거 되고, 예측 가능한 방식으로 UI를 업데이트 한다. useEffect는 이러한 React의 철학과 다른 맥락으로 동작한다.
3. 성능: 매 렌더링마다 useEffect로 이벤트 핸들러를 첨부하는 것은 메모리 누수 및 성능 저하의 가능성이 있다.

---

참고: https://react-ko.dev/learn/keeping-components-pure
