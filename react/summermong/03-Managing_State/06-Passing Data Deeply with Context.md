# ✏ Managing State : Passing Data Deeply with Context

일반적으로 부모 > 자식 컴포넌트로 props를 통해 정보를 전달하는데, 이렇게 하면 중간에 여러 컴포넌트를 거쳐야 하거나 동일한 props를 필요로 하는 컴포넌트에 전달하는 과정을 겪어야 한다. 이는 개발 시 유지보수에 어려움을 야기하고 효율적이지 않으므로 Context를 사용해 props를 전달해보자.

### props 전달의 문제

props로 데이터를 전달하는 과정이 복잡해지면 prop drilling이 발생한다.
![image](https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fpassing_data_prop_drilling.png&w=640&q=75)

### Context는 props 전달의 대안이다.

```
// Heading.js

export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```
// App.js

import Heading from './Heading.js';
import Section from './Section.js';

import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

크기에 대한 level을 받아들이는 Heading 컴포넌트가 있다.
이 때 동일한 Section 내의 여러 제목이 항상 같은 크기를 갖도록 하려고 한다.

현재 level prop를 각 Heading에 개별적으로 전달하고 있는데, 대신에 level prop을 Section 컴포넌트로 전달하고 Heading에서 제거하면 보다 코드가 간결해질 수 있다.

```
<Section level={3}>
  <Heading>About</Heading>
  <Heading>Photos</Heading>
  <Heading>Videos</Heading>
</Section>
```

위와 같이 하면 좋을텐데, 현재 Heading 컴포넌트는 가장 가까운 Section의 level을 알 수 없다.
이 때 context를 사용한다.

1. context 생성
2. 데이터가 필요한 컴포넌트에 해당 context 사용
3. 데이터를 지정하는 컴포넌트에서 해당 context 제공

#### 1. context 생성

```
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

#### 2. 데이터가 필요한 컴포넌트에 해당 context 사용

React와 context에서 useContext/LevelContext를 가져온다.

```
export default function Heading({ level, children }) {
  // ...
}
```

```
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

위에서 아래 코드처럼 level prop을 삭제하고 LevelContext에서 level 값을 사용한다.
이제 Heading 컴포넌트에는 level이 없으므로 level prop를 삭제하고 Section 컴포넌트에 level을 전달한다.

```
<Section>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
</Section>
```

```
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

하지만 작동시켜보면 아직 h4가 나오지 않는다.
context를 사용만 했지 아직 제공하지 않기 때문에 모든 제목이 h1로 동일하다. (LevelContext가 1이므로)

#### 3. 데이터를 지정하는 컴포넌트에서 해당 context 제공

```
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

Section 컴포넌트를 context provider로 감싸 LevelContext를 제공한다.
Section 안에 있는 컴포넌트가 LevelContext를 요청하면 level을 제공하라고 지시하고 있다.
컴포넌트는 그 위에 있는 UI 트리에서 가장 가까운 <LevelContext.Provider>의 값을 사용하게 된다.

```
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

현재 이런 식으로 Section의 level을 수동으로 지정해주고 있는데, context를 사용하면 위 컴포넌트에서 정보를 읽을 수 있으므로 level을 읽어 level + 1을 아래로 전달하는 식으로 코드를 수정해보자.

```
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

### Context 특징

1. context를 제공하는 컴포넌트와 사용하는 컴포넌트 사이에 원하는 만큼 컴포넌트를 삽입할 수 있다.
2. CSS에서는 부모 요소의 속성이 자식 요소에게 상속되는 것처럼, React에서도 상위에서 내려오는 context를 하위 컴포넌트가 그대로 사용할 수 있다. 이를 재정의하려면 새로운 값을 갖는 context provider로 해당 컴포넌트를 감싸야 한다. 3. CSS에서는 서로 다른 속성이 서로를 덮어쓰지 않는 것처럼, React의 서로 다른 context도 서로를 덮어쓰지 않는다. createContext()를 사용하여 만든 각각의 context는 완전히 분리되어 있으며, 컴포넌트 간의 연결고리 역할을 한다. 이를 통해 하나의 컴포넌트가 여러 가지 다른 context를 자유롭게 사용하거나 제공할 수 있다.

### Context 사용 전

1. props 전달로 시작해 데이터 흐름을 명확하게 만들 것
2. 컴포넌트를 추출하고 JSX를 children으로 전달할 것

이 2가지 방식이 모두 적합하지 않을 때 context를 고려하자!
context는 간단해 보이지만 오히려 남용할 경우 복잡해지고, 의존성이 명확하게 드러나지 않으며(결합도 증가), 성능이 저하될 수 있는 위험 요소가 존재한다.

### Context 사용 예제

다크모드 테마 변경, 현재 로그인한 계정 정보, state 관리 및 라우팅 등

---

https://react-ko.dev/learn/passing-data-deeply-with-context#context-passes-through-intermediate-components
