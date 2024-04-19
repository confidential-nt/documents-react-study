# ✏ Managing State : Sharing State Between Components

두 컴포넌트의 state가 항상 함께 변경되길 원한다면,
두 컴포넌트에서 state를 제거하고 가장 가까운 공통 부모로 이동해 props로 전달한다.
이를 state 끌어올리기 라고 한다.

### 예제로 알아보는 state 끌어 올리기

```
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        With a population of about 2 million, Almaty is Kazakhstan's largest city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology">
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples". In fact, the region surrounding Almaty is thought to be the ancestral home of the apple, and the wild <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}
```

부모 컴포넌트인 Accordian 컴포넌트가 두 개의 Panel 컴포넌트를 렌더링한다.
각 Panel 컴포넌트는 콘텐츠 표시 여부를 결정하는 불리언 타입 isActive state를 갖는다.

현재는 한 패널의 버튼을 눌러도 다른 패널에 영향을 주지 않고 독립적으로 동작하지만,
한 번에 하나의 패널만 열리도록 변경하려고 한다.

이 때 state를 끌어 올린다.

1. 자식 컴포넌트에서 state를 제거합니다.
2. 공통 부모 컴포넌트에 하드 코딩된 데이터를 전달합니다.
3. 공통 부모 컴포넌트에 state를 추가하고 이벤트 핸들러와 함께 전달합니다.

3단계를 거쳐 Accordian 컴포넌트가 두 Panel 컴포넌트를 조정해 한 번에 하나씩만 열리게 한다.

#### 1. 자식 컴포넌트에서 state를 제거합니다.

부모 컴포넌트에 Panel의 isActive를 제어할 권한을 부여한다. (=부모 컴포넌트가 isActive를 Panel에 prop으로 대신 전달)
이를 위해 Panel 컴포넌트에서 _const [isActive, setIsActive] = useState(false);_ 를 제거한다.
대신 Panel의 props 목록에 *function Panel({ title, children, isActive }) {*를 추가한다.

#### 2. 공통 부모 컴포넌트에 하드 코딩된 데이터를 전달합니다.

가장 가까운 공통 부모 컴포넌트인 Accordian은 source of truth가 된다.
(두 패널 위에 있고 props를 제어할 수 있으므로 현재 어떤 패널이 활성화 되어 있는지에 대한)

```
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About" isActive={true}>
        With a population of about 2 million, Almaty is Kazakhstan's largest city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology" isActive={true}>
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples". In fact, the region surrounding Almaty is thought to be the ancestral home of the apple, and the wild <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor of the modern domestic apple.
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
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}
```

Accordian 컴포넌트에서 하드코딩된 isActive 값을 편집한다.

#### 3. 공통 부모 컴포넌트에 state를 추가하고 이벤트 핸들러와 함께 전달합니다.

state를 끌어 올려 항목의 특성이 변경되었다.
기존에는 불리언 값으로 true, false만 보여줬다면 이제는 어떤 패널이 활성화 되어 있는지 Panel의 인덱스를 나타내는 숫자 값으로 바뀌어야 한다.

_const [activeIndex, setActiveIndex] = useState(0);_
이 값이 0이면 첫 번째 패널이, 1이면 두 번째 패널이 활성화 된 것으로 정했다.

```
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
</>
```

위와 같이 Accordian 컴포넌트는 이벤트 핸들러를 prop으로 전달해 Panel 컴포넌트가 state를 변경할 수 있도록 명시적으로 허용한다.
이는 각 Panel에서 Show 버튼을 누르면 인덱스를 변경해야 하는데 state가 Accordian 내부에 정의되어 있어 직접 변경할 수 없기 때문이다.

```
import { useState } from 'react';

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
        With a population of about 2 million, Almaty is Kazakhstan's largest city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples". In fact, the region surrounding Almaty is thought to be the ancestral home of the apple, and the wild <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          Show
        </button>
      )}
    </section>
  );
}
```

_끌올 성공_

_참고: 제어 및 비제어 컴포넌트_

일반적으로 일부 로컬 state를 가진 컴포넌트를 비제어 컴포넌트라고 한다.
isActive state 변수가 있는 원래 Panel 컴포넌트는 부모가 패널의 활성화 여부에 영향을 줄 수 없기에 제어되지 않는다.

반대로 컴포넌트의 중요 정보가 props에 의해 구동되는 경우 제어 컴포넌트라고 한다.
이렇게 하면 부모 컴포넌트가 그 동작을 완전히 지정할 수 있다. (수정 후 최종 Panel 컴포넌트는 props가 있고 부모 컴포넌트에 의해 제어된다.)

비제어 컴포넌트는 구성이 덜 필요하기에 상위 컴포넌트 내에서 사용하기 쉽지만 통합 시 유연성이 떨어진다.
제어 컴포넌트는 비교적 더 유연하지만 부모 컴포넌트가 props를 사용해 구성해야 한다.

컴포넌트를 작성할 때는 props를 통해 어떤 정보를 제어해야 하는지,
state를 통해 어떤 정보를 제어하지 않아야 하는지를 고려해 개발하면 된다.

### 각 state의 단일 진실 공급원(SSOT)

React에서 많은 컴포넌트는 고유한 state를 갖고 있다.
일부 state는 입력값과 같이 트리의 맨 아래에 있는 컴포넌트에 가깝게, 또 어떤 것은 상단에 더 가깝게 위치할 수 있다.
각 고유한 state들에 대해 해당 state를 소유하는 컴포넌트를 선택하게 된다.

이를 단일 진실 공급원이라고 한다.
각 state마다 해당 정보를 소유하는 특정 컴포넌트가 있다는 뜻으로,
컴포넌트 간에 공유하는 state의 복제 대신 공통으로 공유하는 부모로 끌어 올려 자식에게 전달한다.

---

참고
https://react-ko.dev/learn/sharing-state-between-components
