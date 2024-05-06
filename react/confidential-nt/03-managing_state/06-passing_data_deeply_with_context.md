# Passing Data Deeply with Context

## 들어가면서

일반적으로 부모 컴포넌트에서 자식 컴포넌트로 props를 통해 정보를 전달하지만, 중간에 여러 컴포넌트를 거쳐야 하거나 앱의 여러 컴포넌트가 동일한 정보를 필요로 하는 경우 과도한 Props Drilling 문제가 발생할 수 있음. Context를 사용하면 부모 컴포넌트가 props를 통해 명시적으로 전달하지 않고도 깊이 여부와 무관하게 그 아래 트리의 모든 컴포넌트에서 일부 정보를 사용할 수 있음.

## props 전달의 문제

props 전달은 UI 트리를 통해 데이터를 사용하는 컴포넌트로 명시적으로 연결할 수 있는 좋은 방법. 그러나 트리 깊숙이 prop을 전달해야 하거나 많은 컴포넌트에 동일한 prop이 필요한 경우 prop 전달이 장황하고 불편해질 수 있음. 가장 가까운 공통 조상이 데이터가 필요한 컴포넌트에서 멀리 떨어져 있을 수 있으며, state를 그렇게 높이 끌어올리면 “prop drilling” 이라고 불리는 상황이 발생할 수 있음.

props를 전달하지 않고도 트리에서 데이터를 필요한 컴포넌트로 “텔레포트”할 수 있는 방법: context

## Context: props 전달의 대안

Context를 사용하면 상위 컴포넌트가 그 아래 전체 트리에 데이터를 제공할 수 있음. context는 다양한 용도로 사용됨.

예시 코드: 크기에 대한 level을 받아들이는 Heading컴포넌트가 있음. 동일한 Section 내의 여러 제목이 항상 같은 크기를 갖도록 하려고 한다고 가정.

```javascript
import Heading from "./Heading.js";
import Section from "./Section.js";

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

현재 level prop을 각 \<Heading>에 개별적으로 전달.

```javascript
<Section>
  <Heading level={3}>About</Heading>
  <Heading level={3}>Photos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

이렇게 하는 대신 level prop을 \<Section>컴포넌트로 전달하고 \<Heading>에서 제거할 수 있다면 좋을 것. **하지만 \<Heading>컴포넌트가 가장 가까운 \<Section>의 level을 어떻게 알 수 있을까?** **그러기 위해서는 자식이 트리 위 어딘가에서 데이터를 “요청”할 수 있는 방법이 필요.**

props 만으로는 부족. 이때 context가 중요한 역할을 함. 세 단계로 진행:

1. context를 생성. (제목 level을 위한 것이므로 LevelContext라고 부를 수 있음)
2. 데이터가 필요한 컴포넌트에서 해당 context를 사용. (Heading은 LevelContext를 사용.)
3. 데이터를 지정하는 컴포넌트에서 해당 context를 제공. (Section은 LevelContext를 제공).

context는 멀리 떨어져 있는 상위 트리라도 그 안에 있는 전체 트리에 일부 데이터를 제공할 수 있게 해줌.

## Step1: Context 만들기

```javascript
import { createContext } from "react";

export const LevelContext = createContext(1);
```

createContext의 유일한 인수는 기본값. 모든 종류의 값(객체 포함)을 기본값으로 전달할 수 있음. 여기서 1은 가장 큰 제목 수준을 의미.

## Step 2: context 사용하기

React와 context에서 useContext Hook을 가져옴.

```javascript
import { useContext } from "react";
import { LevelContext } from "./LevelContext.js";
```

현재 Heading 컴포넌트는 props 에서 level을 읽음.

```jsx
export default function Heading({ level, children }) {
  // ...
}
```

대신 level prop을 제거하고 방금 import한 context인 LevelContext에서 값을 읽기.

```javascript
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

useContext는 Hook. useState 및 useReducer와 마찬가지로, React 컴포넌트의 최상단에서만 Hook을 호출할 수 있음. **useContext는 React에게 Heading 컴포넌트가 LevelContext를 읽기를 원한다고 알려줌.**

이제 Heading 컴포넌트에는 level prop 이 없으므로 더 이상 JSX에서 Heading에 level prop 을 전달할 필요가 없음. 대신 Section이 level을 받도록 JSX를 수정.

```javascript
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

context를 사용하고 있지만 아직 context를 제공하지 않았기 때문에 아직 작동 안함. React는 어디서 그것을 가져와야 할지 아직 모름.

context를 제공하지 않으면 React는 **이전 단계에서 지정한 기본값을 사용.** 이 예제에서는 createContext의 인수로 1을 지정했기 때문에, useContext(LevelContext)는 1을 반환하고 모든 제목을 \<h1>으로 설정. 각 Section이 자체 context를 제공하도록 하여 이 문제를 해결해보자.

## Step 3: context 제공하기

Section 컴포넌트는 현재 children을 렌더링. context provider로 감싸 LevelContext를 제공하라.

```javascript
import { LevelContext } from "./LevelContext.js";

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>{children}</LevelContext.Provider>
    </section>
  );
}
```

이는 React에게 “이 \<Section> 안에 있는 컴포넌트가 LevelContext를 요청하면 이 level을 제공하라.”고 지시. 컴포넌트는 그 위에 있는 UI 트리에서 가장 가까운 \<LevelContext.Provider>의 값을 사용.

원래 코드와 동일한 결과이지만, 각 Heading 컴포넌트에 level prop을 전달할 필요가 없다! 대신, 위의 가장 가까운 Section에 요청하여 제목 level을 “파악”.

1. level prop을 \<Section>에 전달.
2. Section은 section의 children을 \<LevelContext.Provider value={level}>로 감쌈.
3. Heading은 useContext(LevelContext)를 사용하여 위의 LevelContext값에 가장 가까운 값을 요청.

## 동일한 컴포넌트에서 context 사용 및 제공

현재는 여전히 각 section의 level을 수동으로 지정해야 함.

```javascript
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

context를 사용하면 위의 컴포넌트에서 정보를 읽을 수 있으므로 각 Section은 위의 Section에서 level을 읽고 level + 1을 자동으로 아래로 전달할 수 있음.

```javascript
import { useContext } from "react";
import { LevelContext } from "./LevelContext.js";

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

이렇게 변경하면 level prop을 \<Section>이나 \<Heading>모두에 전달할 필요가 없음.

```javascript
import Heading from "./Heading.js";
import Section from "./Section.js";

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

이제 Heading과 Section은 모두 LevelContext를 읽어 얼마나 “깊은” 수준인지 파악. 그리고 Section은 그 children을 LevelContext로 감싸 그 안에 있는 모든 것이 “더 깊은” level에 있음을 지정.

- 참고: 이 예제에서는 중첩된 컴포넌트가 context를 재정의하는 방법을 시각적으로 보여주기 위해 제목 level을 사용하지만 context는 다른 많은 사용 사례에도 유용. context를 사용하여 현재 색상 테마, 현재 로그인한 사용자 등 전체 하위 트리에 필요한 모든 정보를 전달할 수 있음.

## Context는 중간 컴포넌트들을 통과합니다

context를 제공하는 컴포넌트와 context를 사용하는 컴포넌트 사이에 원하는 만큼의 컴포넌트를 삽입할 수 있음. 여기에는 \<div>와 같은 기본 제공 컴포넌트와 사용자가 직접 빌드할 수 있는 컴포넌트가 모두 포함됨.

이 예시에서는 동일한 Post 컴포넌트(점선 테두리 포함)가 두 개의 서로 다른 중첩 단계에서 렌더링됨. 그 안의 \<Heading>이 가장 가까운 \<Section>에서 자동으로 level을 가져오는 것을 볼 수 있음:

```javascript
import Heading from "./Heading.js";
import Section from "./Section.js";

export default function ProfilePage() {
  return (
    <Section>
      <Heading>My Profile</Heading>
      <Post title="Hello traveller!" body="Read about my adventures." />
      <AllPosts />
    </Section>
  );
}

function AllPosts() {
  return (
    <Section>
      <Heading>Posts</Heading>
      <RecentPosts />
    </Section>
  );
}

function RecentPosts() {
  return (
    <Section>
      <Heading>Recent Posts</Heading>
      <Post title="Flavors of Lisbon" body="...those pastéis de nata!" />
      <Post title="Buenos Aires in the rhythm of tango" body="I loved it!" />
    </Section>
  );
}

function Post({ title, body }) {
  return (
    <Section isFancy={true}>
      <Heading>{title}</Heading>
      <p>
        <i>{body}</i>
      </p>
    </Section>
  );
}
```

이 기능이 작동하기 위해 특별한 작업을 수행하지 않았음. Section은 그 안에 있는 트리의 context를 지정하므로 아무 곳에나 \<Heading>을 삽입할 수 있으며 올바른 크기를 가짐.

**Context를 사용하면 “주변 환경에 적응”하고 렌더링되는 위치 (context)에 따라 다르게 표시되는 컴포넌트를 작성할 수 있음**

context가 작동하는 방식은 CSS 속성 상속을 떠올리게 할 수 있음. CSS에서는 \<div>에 color: blue을 지정할 수 있으며, 중간에 다른 DOM 노드가 color: green으로 재정의하지 않는 한 그 안에 있는 모든 DOM 노드는 아무리 깊어도 그 색을 상속받음. **마찬가지로 React에서 위에서 오는 context를 재정의하는 유일한 방법은 children을 다른 값으로 context provider로 감싸는 것.**

CSS에서는 color 및 background-color와 같은 서로 다른 속성이 서로 재정의되지 않음. background-color에 영향을 주지 않고 모든 \<div>의 color를 빨간색으로 설정할 수 있음. **마찬가지로 서로 다른 React context도 서로 재정의하지 않음.** createContext()로 만드는 각 context는 다른 context와 완전히 분리되어 있으며, 특정 context를 사용하거나 제공하는 컴포넌트를 함께 묶음. 하나의 컴포넌트가 문제없이 다양한 context를 사용하거나 제공할 수 있음.

## context를 사용하기 전에

context는 사용하기 매우 유혹적! 그러나 이는 또한 너무 쉽게 남용될 수 있다는 의미이기도 함. props를 몇 단계 깊이 전달해야 한다고 해서 해당 정보를 context에 넣어야 한다는 의미는 아님.

다음은 context를 사용하기 전에 고려해야 할 몇 가지 대안

1. props 전달로 시작하라. 컴포넌트가 사소하지 않다면,(중요한 컴포넌트라면) 수십 개의 props를 수십 개의 컴포넌트에 전달해야 하는 경우가 드물지 않음.(있을 수 있는 일임.) 지루하게 느껴질 수도 있지만, 어떤 컴포넌트가 어떤 데이터를 사용하는지 매우 명확해짐! 코드를 유지 관리하는 사람은 props를 사용하여 데이터 흐름을 명확하게 만든 것에 만족할 것. -> 오버 엔지니어링 하지 말아라.

   - **props 전달은 컴포넌트가 사용하는 데이터를 명확히 이해할 수 있게 하여, 코드의 유지보수에 유리한 점을 강조.** 많은 수의 props를 여러 컴포넌트에 전달하는 것이 번거롭고 지루할 수 있지만, 이 방식은 각 컴포넌트의 의존성과 데이터 사용을 매우 투명하게 만듦. Context API를 사용하기 전에 이러한 props 전달 방식의 이점을 고려하는 것이 중요. **Context API는 데이터를 전역적으로 공유하여 props의 반복적인 전달을 피할 수 있지만, 컴포넌트 간의 명확한 데이터 의존성이 흐려질 수 있기 때문.** 따라서 props를 사용할 때는 각 컴포넌트가 필요로 하는 정확한 데이터만을 전달하고, 이러한 명확성이 유지 보수에 어떻게 도움이 되는지를 평가해야.

2. 컴포넌트를 추출하고 JSX를 children으로 전달하라. 일부 데이터를 해당 데이터를 사용하지 않는 중간 컴포넌트의 여러 레이어를 거쳐 전달한다면(그리고 더 아래로만 전달한다면), 이는 종종 그 과정에서 일부 컴포넌트를 추출하는 것을 잊었다는 것을 의미. 예를 들어, posts과 같은 데이터 props를 직접 사용하지 않는 시각적 컴포넌트에 \<Layout posts={posts} /> 와 같은 방법 대신, Layout이 children을 prop으로 사용하도록 만들고 \<Layout>\<Posts posts={posts} />\</Layout>를 렌더링하라. 이렇게 하면 데이터를 지정하는 컴포넌트와 데이터를 필요로 하는 컴포넌트 사이의 레이어 수가 줄어듦.

이 두 가지 접근 방식이 모두 적합하지 않은 경우 context를 고려.

## context 사용 사례

테마: 앱에서 사용자가 앱의 모양을 변경할 수 있는 경우(예: 다크 모드), 앱 상단에 context provider를 배치하고 시각적 모양을 조정해야 하는 컴포넌트에서 해당 context를 사용할 수 있음.

현재 계정: 많은 컴포넌트에서 현재 로그인한 사용자를 알아야 할 수 있음. 이 정보를 context에 넣으면 트리의 어느 곳에서나 편리하게 읽을 수 있음. 또한 일부 앱에서는 여러 계정을 동시에 조작할 수 있음(예: 다른 사용자로 댓글을 남기는 경우). 이러한 경우 UI의 일부를 다른 현재 계정 값으로 중첩된 provider로 감싸는 것이 편리할 수 있음.

라우팅: 대부분의 라우팅 솔루션은 내부적으로 context를 사용하여 현재 경로를 유지. 이것이 모든 링크가 활성 상태인지 아닌지를 “아는” 방식. 자체 라우터를 구축하는 경우에도 이러한 방식을 사용할 수 있음.

state 관리: 앱이 성장함에 따라 많은 state가 앱 상단에 가까워질 수 있음.(공통적으로 사용해야할 state라면 특히...) 아래에 있는 많은 멀리 떨어진 컴포넌트에서 이를 변경하고 싶을 수 있음. context와 함께 reducer를 사용하여 복잡한 state를 관리하고 번거로움 없이 멀리 떨어진 컴포넌트에 전달하는 것이 일반적.

Context는 정적 값에만 국한되지 않음. 다음 렌더링에서 다른 값을 전달하면 React는 아래에서 이를 읽는 모든 컴포넌트를 업데이트함! 이것이 context가 state와 함께 자주 사용되는 이유.

일반적으로 트리의 다른 부분에 있는 멀리 떨어진 컴포넌트에서 일부 정보가 필요한 경우 context가 도움이 될 수 있다는 좋은 신호.
