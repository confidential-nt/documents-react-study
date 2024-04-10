# Sharing State Between Components

## 들어가면서

때로는 두 컴포넌트의 state가 항상 함께 변경되기를 원할 때가 있음. 그렇게 하려면 두 컴포넌트에서 state를 제거하고 가장 가까운 공통 부모로 이동한 다음 props를 통해 전달하면 됨. 이를 **state 끌어올리기**라고 하며, React 코드를 작성할 때 가장 흔히 하는 작업 중 하나

## 예제로 알아보는 state 끌어올리기

이 예제에서는 부모 컴포넌트인 Accordion 컴포넌트가 두 개의 Panel 컴포넌트를 렌더링

각 Panel 컴포넌트는 콘텐츠 표시 여부를 결정하는 불리언 타입 isActive state를 가짐.

```javascript
import { useState } from "react";

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>Show</button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        With a population of about 2 million, Almaty is Kazakhstan's largest
        city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology">
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for
        "apple" and is often translated as "full of apples". In fact, the region
        surrounding Almaty is thought to be the ancestral home of the apple, and
        the wild <i lang="la">Malus sieversii</i> is considered a likely
        candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}
```

한 패널의 버튼을 눌러도 다른 패널에 영향을 주지 않고 독립적으로 동작.

그러나 이제 한 번에 하나의 패널만 열리도록 변경하려고 함. 설계에 따르면, 두 번째 패널을 열기 위해선 첫 번째 패널을 닫아야. 어떻게 해야할까?

이 두 패널을 조정하려면 세 단계에 걸쳐 부모 컴포넌트로 “state를 끌어올려야:

1. 자식 컴포넌트에서 state를 제거
2. 공통 부모 컴포넌트에 하드 코딩된 데이터를 전달
3. 공통 부모 컴포넌트에 state를 추가하고 이벤트 핸들러와 함께 전달

이렇게 하면 Accordion 컴포넌트가 두 Panel 컴포넌트를 조정하고 한 번에 하나씩만 열리도록 할 수 있음.

## Step 1: 자식 컴포넌트에서 state 제거

부모 컴포넌트에 Panel의 isActive를 제어할 수 있는 권한을 부여.(prop으로) 즉, 부모 컴포넌트가 isActive를 Panel에 prop으로 대신 전달하게 됨.

자식 컴포넌트에서 state 제거 후 해당 스테이트를 부모로부터 자식으로 prop으로 전달.

이제 Panel 컴포넌트는 isActive 값을 제어할 수 없음.

## Step 2: 공통 부모에 하드 코딩된 데이터 전달하기

state를 끌어올리려면 조정하려는 두 자식 컴포넌트의 가장 가까운 공통 부모 컴포넌트를 찾아야.

예제에서 가장 가까운 공통 부모는 Accordion 컴포넌트.

두 패널 위에 있고 props를 제어할 수 있으므로 현재 어떤 패널이 활성화되어 있는지에 대한 **“진실 공급원(source of truth)”**이 됨. Accordion 컴포넌트가 두 패널 모두에 하드 코딩된 isActive 값(예: true)을 전달하도록 함.

```javascript
import { useState } from "react";

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About" isActive={true}>
        With a population of about 2 million, Almaty is Kazakhstan's largest
        city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology" isActive={true}>
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for
        "apple" and is often translated as "full of apples". In fact, the region
        surrounding Almaty is thought to be the ancestral home of the apple, and
        the wild <i lang="la">Malus sieversii</i> is considered a likely
        candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>Show</button>
      )}
    </section>
  );
}
```

## Step 3: 공통 부모에 state 추가

state를 끌어올리면 state로 저장하는 항목의 특성이 변경되는 경우가 많음.

이 경우 한 번에 하나의 패널만 활성화되어야. 즉, 공통 부모 컴포넌트인 Accordion는 어떤 패널이 활성화된 패널인지 추적해야. boolean 값 대신, 활성화된 Panel 의 인덱스를 나타내는 숫자를 state 변수로 사용할 수 있음:

```javascript
const [activeIndex, setActiveIndex] = useState(0);
```

각 Panel에서 “Show” 버튼을 클릭하면 Accordian의 활성화된 인덱스를 변경해야. activeIndex state가 Accordian 내부에 정의되어 있기 때문에 Panel은 값을 직접 설정할 수 없음. Accordion 컴포넌트는 이벤트 핸들러를 prop으로 전달하여 Panel 컴포넌트가 state를 변경할 수 있도록 명시적으로 허용해야.

```javascript
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)}
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)}
  >
    ...
  </Panel>
```

이제 Panel 안에 있는 \<button>은 클릭 이벤트 핸들러로 onShow prop을 사용할 수 있음

```javascript
import { useState } from "react";

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        With a population of about 2 million, Almaty is Kazakhstan's largest
        city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for
        "apple" and is often translated as "full of apples". In fact, the region
        surrounding Almaty is thought to be the ancestral home of the apple, and
        the wild <i lang="la">Malus sieversii</i> is considered a likely
        candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive, onShow }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? <p>{children}</p> : <button onClick={onShow}>Show</button>}
    </section>
  );
}
```

이렇게 state 끌어올리기가 완성! **state를 공통 부모 컴포넌트로 옮기면 두 패널을 조정할 수 있게 됨.**

두 개의 “is shown” 플래그 대신 활성화된 인덱스를 사용하면 한번에 하나의 패널만 활성화되게 할 수 있었음. 그리고 이벤트 핸들러를 자식에게 전달하면 자식이 부모의 state를 변경할 수 있었음.

참고 : 제어 및 비제어 컴포넌트

- **일반적으로 일부 로컬 state를 가진 컴포넌트를 “비제어 컴포넌트”라고 부름.** 예를 들어, isActive state 변수가 있는 원래 Panel 컴포넌트는 **부모가 패널의 활성화 여부에 영향을 줄 수 없기 때문에 제어되지 않음.**
- 반대로 **컴포넌트의 중요한 정보가 자체 로컬 state가 아닌 props에 의해 구동되는 경우 컴포넌트가 “제어”된다고 말할 수 있음.** 이렇게 하면 **부모 컴포넌트가 그 동작을 완전히 지정할 수 있음.** 최종 Panel 컴포넌트에는 isActive props가 있으며, Accordion 컴포넌트에 의해 제어됨.
- **비제어 컴포넌트는 구성이 덜 필요하기 때문에 상위 컴포넌트 내에서 사용하기가 더 쉽다. 하지만 함께 통합하려는 경우 유연성이 떨어짐. 제어 컴포넌트는 최대한의 유연성을 제공하지만 부모 컴포넌트가 props를 사용하여 완전히 구성해야.**
- 실제로 “제어”와 “비제어”는 엄격한 기술 용어가 아니며, 각 컴포넌트에는 **일반적으로 로컬 state와 props가 혼합되어 있음.** 하지만 컴포넌트가 어떻게 설계되고 어떤 기능을 제공하는지에 대해 이야기할 때 유용한 용어.
- 컴포넌트를 작성할 때는 (props를 통해) 컴포넌트에서 어떤 정보를 제어해야 하는지, (state를 통해) 어떤 정보를 제어하지 않아야 하는지 고려하라.
- 하지만 나중에 언제든지 마음을 바꾸고 리팩토링할 수 있음.

## 각 state의 단일 진실 공급원(SSOT)

React 애플리케이션(이하 앱)에서 많은 컴포넌트는 고유한 state를 가지고 있음. 일부 state는 입력값과 같이 leaf 컴포넌트(트리의 맨 아래에 있는 컴포넌트)에 가깝게 “위치” 할 수 있음. 다른 state는 앱의 상단에 더 가깝게 “위치” 할 수 있음.

**각 고유한 state들에 대해 해당 state를 “소유”하는 컴포넌트를 선택하게 됨.** 이 원칙은 “단일 진실 공급원”이라고도 함. 이는 모든 state가 한 곳에 있다는 뜻이 아니라, 각 state마다 해당 정보를 소유하는 특정 컴포넌트가 있다는 뜻.
**컴포넌트 간에 공유하는 state를 복제하는 대신 공통으로 공유하는 부모로 끌어올려서 필요한 자식에게 전달함.**

앱은 작업하면서 계속 변경됨. 각 state의 “위치”를 파악하는 동안 state를 아래로 이동하거나 백업하는 것이 일반적. 이 모든 것이 과정의 일부.

몇 가지 컴포넌트를 사용해 실제로 어떤 느낌인지 알아보려면 [React로 사고하기](https://react-ko.dev/learn/thinking-in-react)를 읽어봐라.
