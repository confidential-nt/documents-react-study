# Understanding Your UI as a Tree

## 들어가면서

React 앱은 많은 컴포넌트들이 서로 중첩된 형태로 구성되어 있음. React는 어떻게 앱의 컴포넌트 구조를 추적할까?

React와 다른 많은 UI 라이브러리들은 **UI를 트리로 모델링함.** **앱을 트리로 생각하면 컴포넌트 간의 관계를 이해하는 데 유용.** 이러한 이해는 성능 및 상태 관리와 같이 앞으로 배울 개념을 디버깅하는 데 도움이 될 것

## 트리로 보는 UI

트리는 아이템 간의 관계 모델이며 UI는 트리 구조를 사용하여 표현되는 경우가 많. 예를 들어 브라우저는 트리 구조를 사용하여 HTML(DOM) 및 CSS(CSSOM)를 모델링. 모바일 플랫폼 또한 트리를 사용하여 뷰 계층 구조를 나타냄.

![](https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fpreserving_state_dom_tree.dark.png&w=1920&q=75)
< React는 컴포넌트로 UI 트리를 생성함. 이 예시에서, UI 트리를 사용하여 DOM에 렌더링이 됨. >

브라우저 및 모바일 플랫폼과 마찬가지로 React 역시 트리 구조를 사용하여 React 앱의 컴포넌트 간의 관계를 관리하고 모델링.

이 트리들은 React 앱을 통해 데이터가 어떻게 흘러가는지, 렌더링 및 앱의 크기를 최적화하는 방법을 이해하는 데 유용한 도구.

## The Render Tree

컴포넌트의 주요 특징은 다른 컴포넌트의 컴포넌트를 합성할 수 있다는 것.

컴포넌트를 중첩하면 부모 컴포넌트와 자식 컴포넌트라는 개념이 생기며, 각 부모 컴포넌트는 그 자체로 다른 컴포넌트의 자식이 될 수 있음.

React 앱을 렌더링할 때 렌더 트리라고 알려진 트리에서 이 관계를 모델링할 수 있음.

아래는 영감을 주는 인용문을 만들어 주는 React 앱

```javascript
import FancyText from "./FancyText";
import InspirationGenerator from "./InspirationGenerator";
import Copyright from "./Copyright";

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}
```

![](https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Frender_tree.dark.png&w=1080&q=75)
<React는 렌더링된 컴포넌트로 구성된 UI 트리인 렌더 트리를 생성>

트리는 노드들로 구성되어 있으며, 각 노드는 컴포넌트를 나타냄. App, FancyText, Copyright 등은 모두 트리의 노드

React 렌더 트리의 루트 노드는 앱의 루트 컴포넌트. 이 경우 루트 컴포넌트는 App이며 React가 렌더링하는 첫 번째 컴포넌트. 트리의 각 화살표는 상위 컴포넌트에서 하위 컴포넌트를 가리킴.

참고: 렌더 트리에서 HTML 태그는 어디에 있을까요?

- 위 렌더 트리에서는 각 컴포넌트가 렌더링하는 HTML 태그에 대한 언급이 없음. 렌더 트리는 React 컴포넌트로만 구성되어 있기 때문.
- UI 프레임워크인 React는 플랫폼에 구애받지 않음.
- React.dev에서는 HTML 마크업을 UI 기본 요소로 사용하는 웹으로 렌더링하는 예제를 보여줌.
- 그러나 React 앱은 UIView 또는 FrameworkElement와 같은 다양한 UI 기본 요소를 사용할 수 있는 모바일, 데스크톱 플랫폼로도 렌더링할 수 있음.
- 이러한 플랫폼 UI 기본 요소는 React의 일부가 아님. React 렌더 트리는 앱이 렌더링되는 플랫폼에 관계없이 React 앱을 제공할 수 있음.
- 그니까, 결론은 특정 플랫폼에 구애 받지 않는 프레임워크이기 때문에, html과 같은 특정 플랫폼을 언급하지 않는다는 것. '모델'이니까...

렌더 트리는 React 애플리케이션의 싱글 렌더 패스를 나타냄. 조건부 렌더링을 사용하면 상위 컴포넌트는 전달된 데이터에 따라 다른 하위 컴포넌트를 렌더링할 수 있음. 영감을 주는 인용문이나 색상을 조건부로 렌더링하도록 앱을 업데이트할 수 있음.

```javascript
import FancyText from "./FancyText";
import InspirationGenerator from "./InspirationGenerator";
import Copyright from "./Copyright";

export default function App() {
  return (
    <>
      <FancyText title text="Get Inspired App" />
      <InspirationGenerator>
        <Copyright year={2004} />
      </InspirationGenerator>
    </>
  );
}
```

![](https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fconditional_render_tree.dark.png&w=1200&q=75)
<조건부 렌더링을 사용하면 다양한 렌더링에 따라, 렌더 트리가 다른 컴포넌트를 렌더링할 수 있습니다.>

이 예에서는 Inspiration.type이 무엇인지에 따라 \<FancyText> 또는 \<Color>를 렌더링할 수 있음. 렌더 트리는 각 렌더 패스마다 다를 수 있음.

렌더 트리는 렌더 패스마다 다를 수 있지만 일반적으로 이러한 트리는 React 앱의 최상위 컴포넌트와 리프 컴포넌트가 무엇인지 식별하는 데 도움이 됨.

최상위 컴포넌트는 루트 컴포넌트에 가장 가까운 컴포넌트로, 그 아래에 있는 모든 컴포넌트의 렌더링 성능에 영향을 미치며 종종 가장 복잡한 컴포넌트를 포함.

리프 컴포넌트는 트리 하단에 있으며 하위 컴포넌트가 없고 자주 리렌더링됨.

이러한 컴포넌트들를 식별하는 것은 앱의 데이터 흐름과 성능을 이해하는 데 유용함.

## 모듈 의존성 트리

트리로 모델링할 수 있는 React 앱의 또 다른 관계는 앱의 모듈 종속성

컴포넌트와 로직을 별도의 파일로 분할하면서 컴포넌트, 함수 또는 상수를 내보낼 수 있는 JS 모듈로 만듦.

모듈 종속성 트리의 각 노드는 모듈이고 각 분기는 해당 모듈의 import 문을 나타냄.

이전 Inspirations 앱을 사용하면 모듈 종속성 트리, 줄여서 종속성 트리를 구축할 수 있음.

![](https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fmodule_dependency_tree.dark.png&w=1920&q=75)

트리의 루트 노드는 entrypoint 파일이라고도 알려진 루트 모듈. 루트 컴포넌트를 포함하는 모듈인 경우가 많음.

동일한 앱의 렌더 트리와 비교해 보면 구조는 비슷하지만 몇 가지 눈에 띄는 차이점이 있음:

1. 트리를 구성하는 노드는 컴포넌트가 아닌 모듈을 나타냄
2. Inspirations.js와 같은 컴포넌트가 아닌 모듈도 트리에 표시됩니다. 반면에 렌더 트리는 컴포넌트만 트리로 캡슐화함.
3. Copyright.js는 App.js의 하위 노드로 나타나지만 렌더 트리에서는 Copyright 컴포넌트는 InspirationGenerator의 하위 컴포넌트로 나타남. 이는 InspirationGenerator가 JSX를 Child Prop으로 전달하므로 Copyright을 자식 컴포넌트로 렌더링하지만 모듈을 가져오지 않기 때문

종속성 트리는 React 앱을 실행하는 데 필요한 모듈들을 결정하는 데 유용.

프로덕션용 React 앱을 빌드할 때 일반적으로 클라이언트에 제공하는 데 필요한 모든 JavaScript를 번들로 묶는 빌드 단계가 있음. 이를 담당하는 도구를 번들러라고 하며, 번들러는 종속성 트리를 사용하여 어떤 모듈을 포함할지 결정.

앱이 성장함에 따라 번들 크기도 커지는 경우가 많음. 큰 번들 크기는 클라이언트가 다운로드하고 실행하는 데 비용이 많이 듬. 번들 크기가 크면 UI가 그려지는 시간이 지연될 수 있음. 앱의 종속성 트리를 이해하면 이러한 문제를 디버깅하는 데 도움이 될 수 있음.
