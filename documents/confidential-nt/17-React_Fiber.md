## 들어가면서

현 리액트는 내부적으로 React Fiber라는 알고리즘을 사용하고 있다. 이것이 무엇인지, 왜 사용하는지, 왜 등장했는지, 어떤 식으로 작동을 하는지에 관해 가볍게 짚고 넘어가볼 것이다.

리액트 버전 18을 릴리즈하면서 리액트 팀이 노력을 기울였던 부분은 concurrent feature이다. 리액트 팀은 concurrent 렌더링을 가능하게 만들기 위해서 지난 몇 년 동안 꾸준히 리액트가 내부적으로 작동하는 방식을 조율하고 있었으며, 그 핵심에 있는 것이 React Fiber이다.

Fiber에 관해 얘기하기 전에 concurrency와 reconciler의 의미를 가볍게 짚어보자.

## Concurrency 동시성

Concurrency (동시성)이란 두 개 이상의 테스크를 동시에 지원함을 뜻하고, parallelism (평행성) 은 두 개 이상의 테스크를 동시에 실행할 수 있음을 뜻함. Parallelism이 가능하게 하려면 물리적으로 여러 개의 thread를 사용할 수 있어야 하고, JavaScript 같은 single-threaded 환경에서 parallelism은 불가능함. 그렇다면 단일 thread에서는 어떻게 concurrency를 지원할까?

![](https://miro.medium.com/v2/resize:fit:1384/format:webp/1*lMg6EIxo8nGYsuwKPVngfw.png)
<하루에 여러가지 일을 concurrent 하게 처리하는 방법>

나라는 한 명의 노동자가 있음.

- 서비스의 새로운 피쳐를 개발합니다.
- 50% 정도 개발했습니다.
- 갑자기 서비스에 치명적인 버그가 발견되었습니다, 잠시 피쳐 개발을 멈춥니다.
- 버그를 잡으러 갑니다.
- 버그를 100% 잡았습니다.
- 돌아와서, 50% 작업 된 시점부터 다시 피쳐 개발을 계속합니다.
- 피쳐 개발이 100% 완료되었습니다.
- 회의록을 정리합니다.
- 80% 작성했습니다. 잠시 멈춥니다.
- 커피를 내립니다.
- 커피를 100% 내렸습니다.
- 회의록을 다시 작성합니다.
- 회의록이 100% 완료되었습니다.
- 더 급한 업무가 없다면 미루고 미루던 블로그 글을 작성합니다.
- 10% 작성했습니다.…

이렇게 여러가지 일을 처리. 그러나 3–4개가 넘는 다양한 업무를 동시다발적으로 처리하는 parallel 한 능력은 없음. 이 예시에서 평행성을 그리려면 노동자가 두 명은 있어야 동시간에 버그도 처리하고 피쳐도 개발할 수 있다. 고로 여러 작업에 대해 일시 정지와 재가동을 반복하면서 하나의 타임라인 위에서 여러 업무를 처리하며 concurrency를 구현하는 것.

### Incremental Rendering 증분 렌더링

이렇게 여러 가지 일을 한 작업자가 처리하려면 “일시 정지”, “재가동”, “우선순위”는 필수 기능. 애플리케이션의 concurrent 렌더링이 가능하다고 함은 화면 렌더링 테스크에 우선순위를 매겨서 중요한 것을 먼저 처리하고 덜 중요한 것을 나중에 처리하는 것이 가능하다를 의미. 리액트 공식 문서에서는 이것을 **incremental rendering (증분 렌더링)** 이라고 부름.

렌더링에 이러한 우선순위는 왜 필요할까? 예를 들어서 우리 애플리케이션의 JavaScript 스레드에서 지금 당장 다음과 같은 테스크들을 처리해야 한다면, 무엇을 우선순위 높은 렌더링 테스크로 정의해야 할까?

1. 방금 돌아온 API 호출의 응답 처리
1. 현재 진행 중인 스크롤 애니메이션 업데이트
1. 사용자의 버튼 클릭 처리

1번 테스크가 몇백 millisecond 지체된다고 해도 사용자는 큰 차이를 느낄 수 없을 것. 2번 테스크에서 돌아가던 애니메이션이 16ms 이상 지체된다면 사용자는 눈에 띄는 jank를 (삐걱) 목격하게 됨. 3번 테스크의 버튼 클릭 핸들링이 100ms 이상 지체된다면 사용자는 애플리케이션이 직관적으로 반응하지 못하다고 느끼게 됨. 부드럽고 reactive 한 UI를 제공하기 원한다면, 위 테스크 2–3–1 번 순으로 렌더 우선순위를 매겨야 함.

우리는 리액트를 쓰면서 ‘지금 이걸 렌더하고 다음에 이걸 렌더 해’ 같이 DOM 업데이트에 대한 명령을 직접 주입하지 않음. ‘이 컴포넌트는 이런 경우에는 이런 걸 보여주는 요소야’라고 선언하고 화면을 업데이트시키는 과정과 작업의 순서를 리액트에 위임함. 이는 리액트 Design Principle에서의 ‘pull’ scheduling 접근을 대변함.

- 참고: Push 기반 접근은 애플리케이션 (당신, 프로그래머)으로 하여금 스케쥴링이 어떻게 작동할 것인지 결정하도록 요구한다. Pull 기반 접근은 프레임워크 (이 경우 React)로 하여금 똑똑하게 그 결정사항들을 대신 결정할 수 있도록 해준다.

UI 라이브러리 특성상, 리액트는 어떤 렌더 작업들이 우선되어야 하는지 전제할 수 있음. 이 때문에 리액트 라이브러리는 내부적인 구조 조정으로 성능을 위한 최적화를 하고 있고, 이를 가능하게 하기 위해 Fiber라는 것을 사용함.

## Reconciliation 재조정

### Reconciler and Renderer 재조정과 렌더

리액트 앱에 상태 변화가 일어난 시점부터 이에 대응하는 UI 변화가 화면에 비칠 때까지 리액트 내부적으로 여러 단계를 거치는데, 여기서 명확히 분리해야 하는 두 단계는 reconciliation (재조정)과 render (렌더) 임.

Reconciliation은 virtual DOM 트리의 업데이트 된 버전과 실제로 화면에 렌더 돼 있는 버전을 비교하고 바뀐 부분을 감지하는 작업을 뜻함.

### Reconcilation이란?

React에서 어떤 부분들이 변해야하는지 서로 다른 두 개의 트리를 비교하는 데 사용하는 알고리즘

아래 코드를 보자.

React 코드를 작성하다 보면 한번쯤은 반드시 만나봤을 코드

```js
ReactDOM.render(<App />, document.getElementById('root'));
```

\<App/>은 우리가 작성한 엘리먼트일텐데, 이 "엘리먼트" 라는건 정확하게 무엇일까?

우리가 만든 이 엘리먼트는 단순하게 어떤 속성을 갖고 있는지, 어떤 children을 갖고 있는지 등에 대한 내용을 나타내는 하나의 객체인 것. 우리가 \<div> 라는 태그를 쓰던, \<Div>라는 React 컴포넌트(함수컴포넌트 or 클래스 컴포넌트)를 만들어 쓰던, 이는 모두 단순한 객체라는것이 핵심. 결국 화면에 어떻게 렌더링을 해야 하는지에 대한 설명들일 뿐. 이렇게 객체로 구성한 뒤에 React에서 Tree를 구축 하고 파싱을 해서 "어떤 부분을 수정해야 하는구나" 판단이 된 뒤에야 실제 렌더링이 일어나게 되는 것.

그럼 "어떤 부분이 어떻게 다른가"를 판별하기 위해 꼭 필요한 과정이 무엇일까? 당연히 모든 엘리먼트들을 돌고 돌아 기본적인 DOM태그를 얻어내는 과정이 필요할 것.

아래와 같은 코드가 있다고 해보자.

```js
// App.js

// ...
<Hello>
  <Button>I am button!</Button>
</Hello>;
// ...

const Hello = (props) => {
  return <div className="hello">{props.children}</div>;
};
```

당연하게도, React는 어떤 엘리먼트들이 렌더링되어야 하는지 알아야 할 것. 따라서 **React는 render()호출 뒤에 최종적인 자식 요소가 무엇인지 알아내기 위해 재귀적으로 React 내의 트리를 탐색하며 기본적인 DOM Tag를 얻어낸다.** 그게 위 코드에서와 같이 div 태그같은 경우를 말하는 것. 이렇게 가장 기본적인 DOM Tag를 얻어낸 뒤 기존 뷰와 다른점을 찾아내는 과정을 React에서는 "Reconcilation" 이라고 부른다.

우리가 사용하는 React에서는 ReactDOM.render()나 setState()가 호출되면 이 reconcilation 과정을 React가 수행하게 되는데, 이렇게 얻어낸 기본적인 DOM Tag들을 기반으로 기존에 렌더링되었던 트리와 새 트리를 비교하여 변경된 사항들을 확인하게 된다. (그 뒤로는 당연하게도 변경사항을 트리에 반영할 것이다.)

이렇게 Reconcilation이 진행된 뒤에, 실제 DOM에 변경되어야 할 최소 변화들을 적용하게 되는 것이다.

간단하게 전체적으로 정리하고 넘어가자면, React 애플리케이션을 렌더링하면, 앱을 나타내는 노드들이 달린 트리가 생성되고 메모리에 저장된다. 이 트리는 렌더링 환경에 플러시된다. (예를들어, 브라우저 앱의 경우, DOM 작업의 형태로 번역된다). setState와 같은 방법을 통해 앱이 업데이트 되면, 새로운 트리가 생성된다. 새로운 트리는 렌더된 앱에 있어 변화가 필요한 부분들을 발견하기 위해 이전 트리와 비교된다.

n개의 엘리먼트가 있는 트리를 다른 트리로 변환하는 알고리즘의 복잡도는 최소 O(n^3)로 알려져 있다고 한다. 이는 간단치 않은 알고리즘이므로 React는 아래 두 조건을 전제로 복잡도가 O(n)인 휴리스틱 알고리즘을 재조정에 사용.

1. 서로 다른 타입의 두 엘리먼트는 서로 다른 트리를 만들어낸다.
2. 개발자가 key prop을 통해, 여러 렌더링 사이에서 어떤 자식 엘리먼트가 변경되지 않아야 할지 표시해 줄 수 있다.

즉, 다음과 같은 가정을 통해 O(n)의 휴리스틱을 달성했다.

- 이전과 다른 타입의 React 엘리먼트로 교체되었다면 하위 트리는 더 이상 비교하지 않고 전체를 교체한다.
- key가 동일한 React 엘리먼트는 이전과 동일한 엘리먼트로 취급한다.

### Render

Render는 비교 후 변경된 사항을 실제 DOM 요소에 반영하는 작업을 뜻함. 이 두 가지 작업을 해주는 reconciler와 renderer의 모듈이 분리되어있기 때문에 리액트와 리액트 네이티브는 같은 reconciler 코어를 사용하고 다른 renderer 모듈을 사용할 수 있음.

reconciler에 집중해볼 것. 현 상태의 트리와 업데이트될 상태의 트리를 비교하기 위해서, 리액트 v16 이전에 사용되던 Stack reconciler와 이후에 사용되는 Fiber reconciler가 트리를 탐색하는 방식을 비교해보자.

### Before : Stack Reconciler 스택 재조정

앞서 얘기했던 concurrency의 핵심 단어 “일시 정지”, “재가동”, “우선순위”가 기존 시스템에서 가능하지 않았던 이유는 여기에 있음. Stack reconciler는 virtual DOM 트리를 비교하고 화면에 변경 사항을 푸시하는 이 모든 작업을 동기적으로, 하나의 큰 테스크로 실행함.

왜 스택인가? 순차적으로 만나는 컴포넌트들을 돌며 스택, 콜스택에 담아두고 render를 호출하는 방식의 FILO(First In Last Out) 방식이기 때문에 "Stack reconcilation"인 것.

이는 현 상태의 트리와 작업 중인 트리를 DFS 패턴으로 재귀적으로 탐색하며 굉장히 깊은 콜 스택을 만들곤 함. 이런 작업은 일시 중지되거나 취소될 수 없어서, 이 콜 스택이 전부 처리되기 전까지는 메인 스레드는 다른 작업을 할 수 없고, 앱은 일시적으로 무반응 상태가 되거나 버벅거리게 됨.

변경사항이 발생할 때마다 이렇게 업데이트를 하는 것은 리소스 낭비이기에, 프레임이 드랍이 일어날 수 있음. 또한, 애니메이션 업데이트는 데이터 저장소 업데이트보다 빨리 완료되어야 함. 업데이트 종류마다 우선 순위가 달라야 한다는 이야기.

프레임 드랍이 대체 어떤 문제가 있길래 그런 것일까?

요즘의 일반적인 모니터들은 초당 60프레임(60 FPS)정도로로 화면을 재생한다.

이 말은 즉 1/60(16.67) ms 간격으로 새 프레임이 나타난다는 이야기이다. 즉, React는 16ms보다 빠르게 리렌더를 하지 않는다면 프레임 드랍이 일어날 수 밖에 없다는 이야기이다. 프레임 드랍이 되면 당연히 사용자는 버벅거리는 화면을 보게 될 것이다.

단순한 텍스트를 리렌더하는거라면 별 문제가 없다. 그러나 애니메이션이라면 이야기가 달라진다. reconciliation 알고리즘이 업데이트가 있을 때마다 React 내 트리를 다 순회하고 렌더링을 하는데 16ms가 넘어버린다면? 자연스럽게 프레임이 드랍되고 애니메이션이 버벅거리는 화면을 사용자는 보게 될 것이다.

[stack reconciler 가 재귀적으로 virtual DOM을 탐색하는 순서를 시각화한 애니메이션](https://codepen.io/ejilee/pen/ZExPQRE)

### After : Fiber Reconciler 파이버 재조정

Fiber는 이러한 기존 reconciler의 취약점을 보완하기 위해서 리액트 팀에서 2년에 걸쳐 재작성한 리액트 코어의 reconciler 알고리즘. 위에서 얘기한 incremental rendering이 가능해지고, 렌더링 작업을 잘게 쪼개어 여러 프레임에 걸쳐 실행할 수 있고, 특정 작업에 “우선순위”를 매겨 작업의 작은 조각들을 concurrent하게 “일시 정지”, “재가동” 할 수 있게 해줌.

주목할 점은 Fiber 트리에서는 각 노드가 return, sibling, child 포인터 값을 사용하여 체인 형태의 singly linked list를 이룬다는 점. -> 따라서 어디서 작업을 중지했고, 어디서부터 다시 시작해야하는지 파악 가능함.

이 트리에서는 단순히 깊이 우선으로 탐색하는 것이 아니라, 각 노드의 return, sibling, child 포인터를 사용해서 child가 있으면 child, child가 없으면 sibling, sibling이 없으면 return… 의 순으로 다음 파이버로 이동함.

각 fiber는 다음으로 처리해야 할 fiber를 가리키고 있기 때문에, 이 긴 일련의 작업이 중간에 멈춰도, 지금 작업 중인 fiber만 알고 있다면 돌아와서 같은 위치에서 작업을 이어가는 것이 가능.

각 fiber는 이 과정에서 각자의 ‘변경 사항에 대한 정보 (effect)'를 들고 있고, 이를 DOM에 바로바로 반영하지 않고, 모아뒀다가 모든 fiber 탐색이 끝난 후, 마지막 commit 단계에서 한 방에 반영하기 때문에, reconciliation 작업이 commit 단계 전에 중단되어도 실제 렌더 된 화면에는 영향을 미치지 않음.

[fiber reconciler 가 각 fiber의 return child sibling 을 참조하여 트리를 탐색하는 순서를 시각화한 애니메이션](https://codepen.io/ejilee/pen/eYMXJPN)

이렇게 Fiber는 기존의 virtual DOM 트리의 화면 요소에 대응하는 역할도 하고, ‘일’을 관리하는 가상 스택 프레임의 역할도 함. 요약하자면, Stack reconciler는 하나의 콜 스택에서 모든 작업을 한 방에 실행하고, Fiber reconciler는 제어 가능한 가상의 스택 프레임 구조를 만들어서 작업을 잘게 나누어서 실행한다고 볼 수 있음.

- 참고: 스택 재조정은 재귀적으로, 현재 탐색하고있는 노드/일감에 대한 레퍼런스 없이 한방에 처리되었던 것에 반면,
  파이버 재조정은 현재 탐색하고있는 노드/일감(workInProgress fiber) 에 대한 레퍼런스를 들고있고기 때문에, 그리고 재조정 작업을 fiber라는 일감 단위로 분리하기 때문에, 재조정 작업이 중간에 중단되어도, 중단했던 fiber에 대한 래퍼런스를 들고있다면, 해당 fiber에서부터 특정 알고리즘을 따라서 재조정 작업을 다시 실행할 수 있다

## Fiber란?

이제 그럼 기존 아키텍처의 Stack reconcilation에서 Fiber 아키텍처의 reconcilation 알고리즘으로 개선하기 위해 어떤 요구사항이 있는지를 알아보자.

4가지의 요구사항이 존재한다.

1. 작업 일시 중지 후 나중에 다시 시작
2. 작업별 우선 순위 지정
3. 이전에 완료된 작업 재사용
4. 더이상 필요하지 않은 경우 작업 중단

이 작업을 수행하기 위해서는 작업을 단위로 세분화 할 수 있는 방법이 필요하다. 여기서 말하는 이 "작업의 단위"를 Fiber라고 부른다.

그런데, 조금 이상한 점을 느낄 수 있다. React는 결국 JS코드를 실행시키게 된다. 작업 우선순위를 정하고 하려면 이는 곧 JS엔진을 거쳐 무언가를 해야하는데, 뭔가 이상함.

JavaScript의 구조를 조금만 생각해보자. 당장 Call Stack과 Execution Context만 생각해봐도 JS엔진 특성상 스케줄링 순서를 직접 정하는건 불가능한 이야기처럼 보인다.

순서는 고정될 수 밖에 없는 것 같은데, 어떻게 이를 구현할 수 있는 것일까?

그걸 위해 React팀은 context switch와 JS의 Call Stack을 JS로 구현했다.

그게 바로 Fiber 아키텍처이다.

이 Fiber 아키텍처의 모든 Fiber 노드들은 모두 "workLoop()" 라는 함수에서 동작한다. 이 말인 즉, Call Stack에 올라가있는 workLoop Execution Context에 JS로 스케줄링 알고리즘과 Call Stack이 구현되어 관리된다는 이야기이다.

요약하자면, Fiber 아키텍처는 JS로 Scheduling 기능이 포함된 가상의 비동기적인 stack 프레임 모델을 구축한 것이라고 보면 된다.

이 때, 이 Fiber에 Reconcilation을 처리하기 위해 존재하는 Fiber Tree의 경우 Stack Reconcilation을 할 때 처럼 Tree가 recursion으로 구현된 것이 아닌, LinkedList로 구현되어 있다.(정확히는 LCRS트리라고 불리며, Left Children Right Sibiling의 약자이다.)

Fiber Tree를 구성하는 LCRS트리의 각각의 노드들에는 type, key, child, sibiling, memoized같이 각 컴포넌트에 대한 정보들이 들어가있다.

(여기서, memoized된걸 확인했다면 리렌더 시 기존에 메모이제이션 해뒀던 걸 다시 가져오게 되는 것이다.)

## Fiber Structure 파이버 구조

CS 영역에서 ‘Fiber’라는 명칭은 ‘가벼운 실행 스레드’를 일컫는 단어. React Fiber 는 UI 라이브러리에 이 개념을 도입한 경우임.

더 나아가기 전에 명칭에 대한 혼란을 방지하기 위해서… 아래 내용부터는 ‘reconciler 알고리즘으로서의 파이버’와 이를 구현하기 위해서 사용하는 ‘개별 작업의 단위인 파이버 노드’를 분리해서 부를 필요가 있는데, ‘Fiber’로 reconciler 알고리즘을 칭하고, ‘fiber’로 노드를 칭하도록 함.

각 fiber는 컴포넌트 인스턴스의 인풋과 아웃풋에 대한 정보를 들고 있음. 리액트 앱 내 모든 요소에는 이에 대응하는 fiber가 존재함. fiber는 인스턴스에 대한 정보뿐만 아니라 다음 fiber로 향하는 포인터, 변경 사항에 대한 정보도 갖고 있음. 그럼, fiber 가 어떻게 생겼는지 눈으로 확인해 보자. 아무 리액트 앱이나 켜서DOM 요소를 선택한 다음, \_\_reactFiber… 라는 키값을 콘솔로 찍어보자.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*aFtH62Hp2gol-AMnG_PDEg.png)

콘솔에서 fiber의 개별 속성들을 펼쳐보면 많은 부분이 또 다른 fiber를 가리키고 있음을 볼 수 있음.

간결한 설명을 목적으로 소스 코드에 있는 본 타입을 조금 간소화해서 보자.

```

[📕 src : type Fiber]
type Fiber = {
// 인스턴스 관련 항목
tag: WorkTag,
key: null | string,
type: any,

// 가상 스택 관련 항목
return: Fiber | null,
child: Fiber | null,
sibling: Fiber | null,

// 이펙트 관련 항목
flags: Flags,
nextEffect: Fiber | null,
firstEffect: Fiber | null,
lastEffect: Fiber | null,
alternate: Fiber | null,
|};

```

스택과 이펙트 관련 항목들의 타입이 전부 Fiber 인 것을 확인할 수 있는데, 이전 fiber, 다음 fiber 에 대한 포인터인 셈. 이들이 어떻게 사용되는지 설명하기 전에 각 키의 의미를 가볍게 훑어보자.

### 인스턴스 관련 항목

- tag: WorkTag - 이 fiber가 대변하는 컴포넌트/요소 인스턴스의 유형, 0~25의 정수. 각 숫자는 FunctionComponent, ClassComponent, HostComponent, Fragment, ContextProvider, MemoComponent… 같은 이름에 할당되어 있음. (여기서 HostComponent는 \<div /> 같은 html 요소를 의미함)
- key: null | string - 이 fiber가 가리키는 인스턴스의 고유한 식별자. 배열을 맵핑할 때 개별 요소의 정체를 보존하기 위해 사용하는 키와 같은 값.
- type: any - 컴포넌트의 경우 이 인스턴스를 만드는 함수, 혹은 클래스. Html 요소의 경우 ‘div’ 와 같은 DOM 요소를 나타내는 문자열. 루트 fiber의 경우 null.

key와 type 은 reconciliation 과정에서 해당 fiber의 identity (오브젝트 정체성)을 보존할지 혹은 새로운 fiber를 만들지 식별하기 위해 사용됨. 이는 새로운 fiber 노드가 만들어지는 시점에 해당 인스턴스의 실제 key, type 값을 그대로 복사해와서 사용함.

### 가상 스택 관련 항목

- return: Fiber | null - 부모 fiber, 컴포넌트/요소에는 부모가 한 개 이상 있을 수 있는데, 여기의 return 은 현 fiber의 작업 처리가 끝나면 돌아갈 fiber를 가리킴. DOM의 부모 개념보다는 스택 프레임에서의 return 주소와 더 유사.
- child: Fiber | null - 첫 자식 fiber, 마찬가지로 컴포넌트/요소에는 자식이 여러 개가 있을 수 있지만, 여기의 child는 첫 자식 fiber에 대한 포인터.
- sibling: Fiber | null - 현 fiber 옆 형제 fiber를 가리킵니다. child fiber 가 없다면 sibling fiber를 다음으로 탐색을 이어감.

return, chlid, sibling 포인터들을 사용하면 위 애니메이션처럼 리액트의 트리 구조를 하나의 긴 linked list로 탐색하는 게 가능해지고, 이를 하나의 가상 스택 프레임처럼 사용하게 됨.

### 이펙트 관련 항목

이펙트는 두 트리를 비교한 후 감지된 ‘바뀐 점, 바뀌어야 할 작업’을 뜻함.

- flags: Flags - 이펙트, 변화의 유형, 정수. 각 숫자는 Placement, Update, Deletion, Content Reset… 같은 이름에 할당되어 있음. 바이너리로 숫자로 표기되어있고, bitwise OR assignment (|=)를 사용해서 여러 변화의 조합을 workInProgress.flags |= Placement 이런 식으로 축적.
- nextEffect: Fiber | null - reconciliation 과정에서 새로운 변경 사항 혹은 이펙트가 생겼을 때, 이전 이펙트가 발생했던 fiber의 nextEffect 가, 이 새로운 이펙트를 가진 fiber를 가리키도록 함. 전체 트리 비교가 끝나면 루트 fiber의 firstEffect부터 시작해서 nextEffect 포인터를 따라가면 변화가 생긴 fiber만 빠르게 탐색할 수 있음.
- firstEffect: Fiber | null, lastEffect: Fiber | null - 이 fiber 하위에 있는 서브 트리에서의 첫 이펙트를 가진 fiber 와 마지막 이펙트를 가진 fiber 를 가리킴. 특정 fiber에서 이루어진 작업만 일부 재사용하기 위해 사용할 수 있음.
- alternate: Fiber | null - 해당 fiber의 교대 fiber - alternate 속성은 이전 작업 current Fiber와 workInProgress Fiber가 연결되어있는 한쌍으로 Fiber가 재사용되기 위해서 매우 중요한 속성

### workInProgress

트리 비교 작업이 끝나고 변경된 사항이 render commit phase를 통해서 화면까지 반영이 되면, 현재의 앱 상태를 대변하는 fiber 를 ‘flushed fiber’라고 부름.

이와 반대로 아직 작업 중인 상태, 화면까지 반영되지 않은 fiber를 ‘workInProgress fiber’라고 부르는데, 이 말은 즉 특정 컴포넌트에 대응하는 fiber가 최대 두 버전이 존재한다는 뜻. flushed fiber의 alternate 는 workInProgress fiber를 가리키고, workInProgress fiber 의 alternate 는 flushed fiber를 가리킴. (순환참조)

즉, fiber들이 작업에 들어가게 되는 Tree가 workInProgress. fiber가 작업에 들어가면 workInProgress Tree로 들어가게 된다.

기존의 화면에 떠있는 정보들을 담고 있는 tree를 current Tree(flushed tree)라고 부르는데, work-in-progress Tree는 이를 복제해서 만든다.

fiber node들은 자바스크립트 객체들이라서 Commit단계가 수행되기 전까지 비동기적으로 계속 변동이 가능하다.

그렇다면 리액트는 몇 개의 트리를 유지하는가?

createWorkInProgress를 보면 어떤 주석이 있는데 그 주석에서 다음과 같은 내용을 확인할 수 있다.

"react에서는 어차피 두 개 버전의 트리만 필요하기 때문에, 이 workInProgress Tree에 대해 버퍼링 풀이 2개가 있다"

React element와 fiber는 매우 유사하지만 중요한 차이점은 React element는 매번 다시 생성 되지만 React Fiber는 가능한 자주 재사용된다.(초기 마운트 시 Fiber가 생성되며 그 후에는 대부분 재사용된다.)

## Fiber Algorithm 파이버 알고리즘

Fiber 는 reconciliation 작업을 2단계로 나눠서 실행함.

- Phase 1 Render — 실제로 두 fiber 트리를 비교하고 변경된 이펙트들을 수집하는 작업을 함. 이 단계는 concurrent 하게 일시 정지되고 재가동될 수 있음. 리액트 scheduler로 인해 허용되는 시간 동안 작업하고 수시로 멈춰서 메인 스레드에 user input, animation 같은 더 급한 작업이 있는 확인 해가며 실행되기 때문에 아무리 트리가 커도 비교 작업이 메인 스레드를 막을 걱정이 없음. Phase 1의 목적은 이펙트 정보를 포함한 새로운 fiber 트리를 만들어내는 것.
- Phase 2 Commit — Phase 1에서 만든 트리에 표시된 이펙트들을 모아 실제 DOM에 반영하는 작업을 함. 이 단계는 synchronous하게 한 타에 이루어지기 때문에 일시 정지하거나 취소할 수 없음.

Phase 1는 concurrent하게, Phase 2는 synchronous하게 이루어지기 때문에, Phase 1에서의 작업이 잘게 쪼개져 여러 프레임에 걸쳐서 실행되어도 사용자 눈에 보이는 Phase 2의 업데이트는 한 타에 이루어지기 때문에, fiber 관련 작업이 일시 정지되거나 취소되어도 화면에 영향을 미치지 않음.

그럼 fiber 트리의 생성과 비교 과정을 앱이 처음 문서에 마운팅되는 시점부터 한 단계, 한 단계 나눠서 살펴보도록 하자.

- 참고: 중요한 건 전반적인 작업의 분리와 흐름을 이해하는 것으로, 개별 함수의 작동 방식에 너무 얽매일 필요는 없을 것 같다.

### Initial Render and Update 최초 렌더와 업데이트

- 앱을 html 루트 요소에 마운팅할 때, renderer는 reconciler 모듈에서 제공하는 createContainer() 함수를 호출함. 여기서 root이라는 new FiberRootNode(...) 를 만들고, 새로운 HostRoot 태그를 가진 fiber를 만들어서 root의 current 값이 이 HostRoot fiber를 가리키도록 함.
- 앱에 상태 변화/업데이트가 일어남.
- 지금 root.current에 있는 fiber를 복제해서 workInProgress 파이버로 지정함.
- workLoop을 호출함.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*9sghSAFGSB4KlSzJOB5cXA.png)
<workLoop 최초 호출 시 fiber 트리 상태>

setState로 인한 변경사항 발생 시 리액트 내부에서 해당 피버의 flag에 변경이 발생했다고 표시하는 것 같음. 이후에 render phase가 트리거됨.

#### workLoopConcurrent() 모든 일의 반복

현재 작업이 남아있는 fiber가 존재하고, scheduler가 작업을 허락하는 동안 반복적으로 작업을 수행하는 룹을 돌림.

```

[📕 src: workLoopConcurrent]
function workLoopConcurrent() {
while (workInProgress !== null && !shouldYield()) {
performUnitOfWork(workInProgress);
}
}

```

#### performUnitOfWork() 들어가는 일의 흐름 제어

performUnitWork에서는 다음으로 처리할 fiber가 남아있는 이상 계속 beginWork()를 호출하고 다음 작업할 fiber를 workInProgress 에 할당함.

(리액트는 트리를 순회하면서 어디서부터 렌더가 필요한지를 체크해야 한다. 체크가 완료되었다면 이제 거기서부터 beginWork()가 호출)

```

[📕 src: performUnitOfWork]
function performUnitOfWork(unitOfWork: Fiber): void {
const current = unitOfWork.alternate;
const next = beginWork(current, unitOfWork, renderLanes);
if (next === null) {
completeUnitOfWork(unitOfWork);
} else {
workInProgress = next;
}
}

```

#### beginWork() 들어가서 하는 일

해당 fiber의 인풋 값이 current 트리에 있는 fiber의 프롭과 비교했을 때 변경되었나를 확인하고, (변경된 것이 없으면 fiber를 업데이트해줄 필요가 없으므로) workInProgress fiber의 tag 값에 따라 해당 fiber 를 업데이트해주고 다음 작업 fiber 를 반환하는 함수를 호출함. 대략 이렇게 생김

```

[📕 src: beginWork]
function beginWork(
current: Fiber | null,
workInProgress: Fiber,
renderLanes: Lanes,
): Fiber | null {
if (current !== null) {
const oldProps = current.memoizedProps;
const newProps = workInProgress.pendingProps;
if (oldProps === newProps) {
// bail out
}
}
switch (workInProgress.tag) {
case FunctionComponent:
return updateFunctionComponent(...);
case ClassComponent:
return updateClassComponent(...);
case HostRoot:
return updateHostRoot(...);
case HostComponent:
return updateHostComponent(...);
case HostText:
return updateHostText(...);
case Fragment:
return updateFragment(...);
...
}
throw new Error(`Unknown unit of work tag (${workInProgress.tag})...`);
}

```

업데이트 함수는 fiber의 tag에 따라 다양하고 하는 일이 다른데, 그중 가장 단순해 보이는 Fragment 업데이트 함수를 확인해보면 대략 이렇게 생김.

```

function updateFragment(
current: Fiber | null,
workInProgress: Fiber,
renderLanes: Lanes,
) {
const nextChildren = workInProgress.pendingProps;
reconcileChildren(current, workInProgress, nextChildren, renderLanes);
return workInProgress.child;
}

```

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*VMCWi6DsKlZYHSeMadv6yg.png)
<A1 fiber의 child 값에 있는 B1 fiber 가 다음 wordInProgress 가 됨>

이렇게 beginWork() 가 반환하는 child fiber가 다음 workInProgress가 되고, workLoop은 다음 fiber 가 없을 때까지 계속 beginWork() 작업을 반복하며 workInProgress fiber를 업데이트시켜줌. 위 updateFragment의 예를 보면, 마지막 줄의 workInProgress.child 가 null 인 경우에, performUnitOfWork에서의 next 값이 null이 될 것이고, 비로소 completeUnitOfWork() 가 실행됨.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*5XEO4DZ6tqsHK9uFch3IzA.png)
<A1, B1, C1의 beginWork를 실행 후 child 가 없는 D1 fiber에 다다름>

사실은 이러한 반복적으로 타고들어가는 짓을 하는 이유는 우리는 가장 기본적인 DOM element를 얻어야 하기 때문에..

#### completeUnitOfWork() 돌아 나오는 일의 흐름 제어

completeUnitOfWork가 호출되었다는 것은 지금 workInProgress fiber의 하위로 더 이상 처리할 fiber가 없다는 의미. 우선 해당 fiber를 완료 처리하고 completeWork를 호출함 (이 작업은 아래에서 더 설명) 만약에 completeWork를 호출하면서 다음으로 처리할 fiber가 생겨버렸다면 (예. 상태 변화로 인해 해당 요소 아래로 child가 생성된다면) 또다시 workInProgress 값에 할당해주고 completeUnitOfWork를 그대로 종료함. 흐름의 제어는 다시 performUnitOfWork로 돌아감. 혹은 sibling fiber가 있다면 이를 workInProgress 값에 할당해주고, 마찬가지로 performUnitOfWork 에게 제어가 넘어감.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*jQ-7fNpbENsd1_kfqiSrFA.png)
<child가 없어 D1을 completeWork 처리 후 sibling 을 발견해 D2가 workInProgress가 됩니다>

더 이상 처리할 child 도 sibling 도 없다면 해당 fiber의 return fiber로 돌아가서 마찬가지로 방금 completeUnitOfWork 의 do whlie 룹에서 했던 작업을 반복. 이 작업은 completedWork이 없을 때까지, 그러니까 return fiber가 없을 때까지 계속함. 마지막으로 루트에 완료 표시를 해줌.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*4zMk7hgGRdmQk3XuqP86MA.png)
<전체 fiber 트리를 반정도 reconcile 한 모습. E1, E2 노드에 변경사항이 있다는 걸 표시만 한 상황>

```

[📕 src: completeUnitOfWork]
function completeUnitOfWork(unitOfWork: Fiber): void {
let completedWork = unitOfWork;
do {
const current = completedWork.alternate;
const returnFiber = completedWork.return;
if ((completedWork.flags & Incomplete) === NoFlags) {
const next = completeWork(current, completedWork, renderLanes);
if (next !== null) {
workInProgress = next;
return;
}
} else {
// something threw
// ...
}
const siblingFiber = completedWork.sibling;
if (siblingFiber !== null) {
workInProgress = siblingFiber;
return;
}
completedWork = returnFiber;
workInProgress = completedWork;
} while (completedWork !== null);
if (workInProgressRootExitStatus === RootInProgress) {
workInProgressRootExitStatus = RootCompleted;
}
}

```

#### completeWork() 돌아 나오면서 하는 일

beginWork와 마찬가지로, 해당 fiber의 tag 값에 따라 조금씩 다른 작업을 하게 됨. 아래 코드에서 보이는 bubbleProperties는 해당 fiber의 모든 children의 flag 와 lane 정보(이건 몰라도 되는데 암튼 업데이트 관련해서 중요한 것)를 머징해주는 작업이라고 볼 수 있을 것 같음.

즉, 작업이 끝났다는걸 확정하는 함수이다.

무언가 변경점이 있다면, 여기서 이를 하나 하나 모아준다.

이를 "Effect" 라고 하는데, 이는 나중에 EffectList에 담겨 한번에 처리한다.

(우리가 원하는 기본 dom element 얻은 시점이겠지..)

근데 만약 메모이제이션 된 컴포넌트라면 React가 비교 후 판단하여 새로운 Component를 부를지 말지를 결정하게 된다.

- 참고: pendingProps와 memoizedProps
  - 개념상으로 props는 함수의 인수이다. Fiber의 pendingProps는 그것의 실행 처음에 설정되고, memoizedProps는 실행 마지막에 설정된다.
  - 들어오는 pendingProps가 memoizedProps와 같으면, 이걸 통해 fiber에게 이전 출력물이 재사용될 수 있음을 알리고, 불필요한 작업을 방지할 수 있다.

비교 결과가 true이면 기존에 만들었던 fiber를 그대로 사용하게 되는 것

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*QBOaoAdfprVQy_UraUS38A.png)
<completeWork가 개별 fiber의 이펙트를 표시하며 돌아나오는 과정>

```

[📕 src: completeWork]
function completeWork(
current: Fiber | null,
workInProgress: Fiber,
renderLanes: Lanes,
): Fiber | null {
switch (workInProgress.tag) {
case IndeterminateComponent:
case LazyComponent:
case SimpleMemoComponent:
case FunctionComponent:
case ForwardRef:
case Fragment:
case Mode:
case Profiler:
case ContextConsumer:
case MemoComponent:
bubbleProperties(workInProgress);
return null;
...
}
throw new Error(`Unknown unit of work tag (${workInProgress.tag})...`);
}

```

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*wioVWNWHvH1fl-825vcLmg.png)
<RootCompleted 상태>

completeUnitOfWork 에서 마지막으로 루트에 완료 표시까지 했다면, Phase 1의 목적인 이펙트 정보를 포함한 새로운 fiber 트리가 완성된 셈. 이제 Phase 2로 넘어감.

### commitWork() 모든 작업 반영

work-in-progress 루트의 exitStatus 가 RootCompleted가 되면, 비로소 commitWork가 호출됨. 많은 일이 일어나는 구간이지만, 요약하자면 Phase 1에서 완성된 fiber 트리에 유효한 effect flag 값이 있다면, commitMutationEffects, resetAfterCommit, commitLayoutEffects, requestPaint 같은 작업을 실행해서 화면에 fiber 트리를 flush 해주고 루트의 current 포인터를 바꿔줌. 기존에 있던 current 트리는 새로 지정된 current의 alternate 로 재활용됨. 이 작업은 workLoopConcurrent의 일환이 아니므로 동기적으로 한 큐에 실행됨.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Y0BAyBnDWKHbQNFGFhIhHA.png)
<workInProgress 트리에 있는 effect 정보들을 모아 DOM에 반영해줌. 이제 우측 트리가 최신 DOM 상태를 반영한 flushed 트리가 되어야 함>

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*FnBHthbgckrqrawx0tqv3w.png)
<root의 current 포인터가 바뀌어 작업하던 트리가 flushed fiber tree가 되고, 기존의 flushed fiber 가 반대로 workInProgress가 될 차례>

```

[📕 src: commitRoot]
function commitRoot(
root: FiberRoot,
recoverableErrors: null | Array<CapturedValue<mixed>>,
transitions: Array<Transition> | null,
) {
try {
commitRootImpl(
root,
recoverableErrors,
transitions,
previousUpdateLanePriority,
);
} finally {
// commit 후처리 작업 (내용 생략)
}
return null;
}
function commitRootImpl(
root: FiberRoot,
recoverableErrors: null | Array<CapturedValue<mixed>>,
transitions: Array<Transition> | null,
renderPriorityLevel: EventPriority,
) {
do {
flushPassiveEffects();
} while (rootWithPendingPassiveEffects !== null);
const finishedWork = root.finishedWork;
const lanes = root.finishedLanes;
// passive effects, lane 관련 작업 (내용 생략)
if (root === workInProgressRoot) {
workInProgressRoot = null;
workInProgress = null;
workInProgressRootRenderLanes = NoLanes;
}
// 남은 pending effect가 있다면 schedule a callback (내용 생략)
const subtreeHasEffects =
(finishedWork.subtreeFlags &
(BeforeMutationMask | MutationMask | LayoutMask | PassiveMask)) !==
NoFlags;
const rootHasEffect =
(finishedWork.flags &
(BeforeMutationMask | MutationMask | LayoutMask | PassiveMask)) !==
NoFlags;
if (subtreeHasEffects || rootHasEffect) {
// priority, transition, context 관련 작업 (내용 생략)
commitMutationEffects(root, finishedWork, lanes);
resetAfterCommit(root.containerInfo);
root.current = finishedWork;

    commitLayoutEffects(finishedWork, root, lanes);

    requestPaint();
    // ... (이하 내용 생략)

} else {
// No effects.
root.current = finishedWork;
}
// passive effects, remaining work, errors check (내용 생략)
return null;
}

```

## 요약

최초 업데이트 직후에 일어나는 일들을 다시 한번 요약해보자. 우선 루트에서 시작함. 루트의 current 포인터가 가리키는 최상단 루트 파이버를 복제해서 새로운 트리를 만들어줌. 이 트리가 current 의 alternate.

여기서부터 concurrent하게 실행되는 Phase 1. alternate 트리의 최상단 fiber를 workInProgress fiber로 설정하고 beginWork를 실행함. 바뀐 input에 따라 해당 컴포넌트/요소에 변경 사항이 생기는지 확인함. 만약 이 과정에서 변경 사항이 있다면 이펙트의 유형을 표시해줌. 해당 fiber에 child fiber가 있다면 chlid를 workInProgress fiber로 설정하고 또다시 beginWork를 실행함. 이렇게 child 가 없을 때까지 반복하다가 child가 없는 fiber에 다다르면 completeWork를 실행함. completeWork fiber 하위에 있는 모든 이펙트에 대한 정보를 모아줌. 해당 fiber에 sibling 이 있다면 해당 sibling을 workInProgress fiber로 설정하고 다시 beginWork를 실행함. 이렇게 작업을 반복하다 보면 마지막 completeWork를 실행하고 루트로 돌아오게 됨. 루트에 완료 표시를 해줌.

여기서부터 synchronous하게 실행되는 Phase 2. Phase 1에서 완료한 workInProgress 트리에 화면에 반영되어야 하는 모든 이펙트에 대한 정보가 담겨있음. workInProgress 트리를 화면에 flush 해줌. 이 작업은 취소되거나 멈출 수 없음. 완료 후 루트의 current 포인터가 방금 flush한 트리를 가리키도록 하고 기존에 있던 current 트리를 새로운 workInProgress 트리로 지정함.

## 참고: Virtual DOM에 관하여

> “virtual DOM”은 특정 기술이라기보다는 패턴에 가깝기 때문에 사람들마다 의미하는 바가 다릅니다. React의 세계에서 “virtual DOM”이라는 용어는 보통 사용자 인터페이스를 나타내는 객체이기 때문에 React elements와 연관됩니다. 그러나 React는 컴포넌트 트리에 대한 추가 정보를 포함하기 위해 “fibers”라는 내부 객체를 사용합니다. 또한 React에서 “virtual DOM” 구현의 일부로 간주할 수 있습니다.

위 글에 따르면 virtual DOM은 React 내부에 실존하는 어떤 구체적인 인스턴스나 value라기보다는 React가 React 엘리먼트와 native DOM을 동기화하기 위한 패턴의 이름을 의미하며 문맥에 따라 지칭하는 실체가 다르다. React 내부를 충분히 파악하지 않은 개발자들은 이 부분을 헷갈리기 쉬운데 React 내부에선 실제로 virtual DOM이라는 이름을 가진 개념이 존재하지 않는다고함.

그렇다면 React의 핵심 개념이라고 할 수 있는 virtual DOM이 결국 무엇을 칭하는 것인가? React 엘리먼트인가? FiberNode인가? 아니면 Commit 단계의 updateQueue인가?

여러 분석 자료를 살펴보았지만 통일된 의견이 없고 마찬가지로 재조정 과정을 자세하게 분석할수록 virtual DOM에 대해 혼란스러워 하는 사람들이 많았다. 심지어 React 팀 내부에서도 다르게 호칭하는 경우도 있었다.

이미 React 개발자인 Dan Abramov가 virtual DOM이라는 용어 자체를 폐기할 것을 권고하고 있었다. virtual DOM은 React가 렌더링할 때마다 DOM을 생성하지 않는다는 점을 명확히 하기 위해 도입했지만 현재는 사용자들이 더 이상 그러한 오해를 하지 않으며 오히려 DOM의 이슈를 극복하기 위해 virtual DOM을 도입했다는 오해를 하고 있어 React의 의도와 다르게 해석되고 있기 때문이라고 한다. 대신에 UI를 값으로 다룬다는 의미에서 “value UI”라 표현하기를 권고한다.

그리고 DOM은 React가 렌더할 수 있는 환경들 중 하나이며, 이외에도 네이티브 iOS나 Android view를 대상으로 하는 React Native도 있다. 그렇기 때문에 “virtual DOM”이란 표현은 다소 잘못된 호칭이다.

## 참고: 우선순위 조정

어떤 작업이 애니메이션과 같은 높은 우선순위를 가지고 있다면 requestAnimationFrame 함수를 통해 우선순위가 높은 함수를 예약할 수 있다.
또는 requestIdleCallback 함수를 통해 유휴 기간동안 호출한 우선순위가 낮은 함수를 예약할 수 있다.(두 함수를 지원하지 않는 브라우저의 경우 폴리필을 제공한다.)

## 출처:

1. [React 파이버 아키텍쳐 분석](https://d2.naver.com/helloworld/2690975)
2. [React Deep Dive - Fiber](https://blog.mathpresso.com/react-deep-dive-fiber-88860f6edbd0)
3. [[React] Fiber 아키텍처의 개념과 Fiber Reconcilation 이해하기](https://m.blog.naver.com/dlaxodud2388/223195103660)
4. [React Fiber Architecture](https://immigration9.github.io/react/2021/05/29/react-fiber-architecture.html)
5. [React Fiber에 대해서](https://velog.io/@tnghgks/React-Fiber에-대해서)
6. [react-fiber-architecture](https://github.com/acdlite/react-fiber-architecture)
