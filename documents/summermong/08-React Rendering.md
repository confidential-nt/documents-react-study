# React.js의 렌더링 방식

React의 렌더링 방식을 살피기 전, 웹 브라우저가 어떻게 동작하는지 먼저 알아보자.

### 웹 브라우저의 동작

브라우저는 HTML, CSS로 작성한 웹 페이지를 화면에 렌더링한다.
이 렌더링 과정은 **Critical Rendering Path**를 이뤄진다.

<img width="795" alt="Pasted Graphic" src="https://github.com/summermong/another-world-test/assets/124887974/52060ca4-5d2c-4f0d-a137-4ac1201cf967">

1단계: HTML -> DOM, CSS -> CSSOM으로 변환한다.
이 때 DOM은 문서 객체 모델의 약자로, HTML을 브라우저가 해석하기 편한 방식으로 변환한 '객체 트리'를 말한다.
CSSOM도 동일하다.

2단계: Render Tree
DOM + CSSOM = Render Tree
웹 페이지의 청사진으로, DOM에는 HTML로 작성한 요소들의 배치 및 모양을 기술한 정보가 있고 CSSOM에는 CSS로 작성한 요소들의 스타일링에 대한 정보가 있는데 이런 정보들을 결합한 것이 Render Tree이기 때문이다.

3단계: Layout
Render Tree를 기반으로 실제 웹 페이지에 요소들의 배치를 결정하는 작업이다. 웹 페이지 안에 HTML 요소들이 어떤 위치에 어떤 사이즈로 들어갈 것인지 계산하는 과정을 말한다.

4단계: Painting
웹 페이지에 요소들을 실제로 화면에 그리는 과정을 말한다.

#### 그럼 업데이트는 어떻게 이뤄질까?

여기서 업데이트란 이벤트에 따라 화면이 실시간으로 변하는 것을 말한다.

이러한 업데이트는 JS가 DOM을 수정하면서 발생하게 된다.
DOM API를 사용해 DOM을 수정하게 되면 브라우저가 이러한 변화를 감지, Render Tree를 다시 만들고 Layout, Painting을 다시 해 변화된 UI를 화면에 그리는 과정을 거친다.

<img width="816" alt="Pasted Graphic 1" src="https://github.com/summermong/another-world-test/assets/124887974/09fae505-aa5e-49e2-85f1-d4445eb954c5">

이렇게 JS로 DOM을 조작할 때는 Reflow, Repaint를 주의해야 한다.
**두 과정은 연산이 오래 소요되는 비싼 작업이므로 DOM의 수정 횟수를 최소화 해야 한다.**

만약 DOM에 300번의 수정이 필요하다면? 300번의 Reflow, Repaint가 발생하게 되어 웹 서비스의 성능 저하를 야기할 수도 있다. 🥲

![image](https://github.com/summermong/another-world-test/assets/124887974/c585eae1-4b6c-4da9-b2c1-2933a5221f49)
![image](https://github.com/summermong/another-world-test/assets/124887974/1ad73e40-20ad-404b-bbbf-d03584aa5714)

DOM을 딱 한 번만 수정하게끔 코드를 바꾸자 성능이 22배 개선되었다!

즉 다양한 업데이트를 모아 한 번에 수정하는 것이 좋은데, 서비스의 규모가 커지면 이를 지키기 어려워진다.

하지만 React에서는 이것이 자동으로 이뤄진다.
내부적으로 동시에 발생한 업데이트를 모아 한 번에 처리하는 기능이 있다.
React는 별도의 렌더링 프로세스를 갖고 있기 때문이다.

### React의 렌더링 프로세스

React는 2가지 단계를 거쳐 화면에 UI를 렌더링한다.

1단계: Render Phase (컴포넌트를 계산하고 업데이트 사항을 파악하는 단계)

2단계: Commit Phase (변경사항을 실제 DOM에 반영하는 단계)

![image](https://github.com/summermong/another-world-test/assets/124887974/c802dc91-2a52-44e1-ab56-d638d9810ff3)

1단계부터 알아보자.

function App() {...} 과 같은 컴포넌트를 호출해 결과값을 계산한다.
단순히 html 태그가 반환되는 것이 아니라, **React Element라는 객체가 반환된다.**

그리고 이 React Element들을 모아 가상 DOM을 생성한다. React Element라고 부르는 객체 값의 모음으로, 실제 DOM이 아닌 복제판이다.
(따라서 값으로 표현된 UI라고 이해하는 것이 더 쉽다.)

즉, **컴포넌트 호출 -> 객체 React Element 모으기 -> 가상 DOM 만들기**와 같은 과정을 겪게 된다.

2단계에서는 가상 DOM을 실제 DOM에 반영한다.
2단계까지 반영하면 Critical Rendering Path를 진행한다.

요약하자면, 업데이트 발생 시 1) Render Phase를 처음부터 다시 실행해 새로운 가상 DOM을 만들고 이전까지 사용했던 가상 DOM과 비교해(diffing) 무엇이 달라졌는지 찾아낸다. 2) 그리고 어디가 어떻게 달라졌는지 찾아낸 후 Commit Phase로 넘어가 실제 DOM에 다시 조정(Reconciliation)하는 프로세스를 갖는다.

+) 가상 DOM을 사용하는 React, 반드시 최고의 속도를 보장할까?

이미 가상 DOM을 생성하고 비교하는 데에도 연산이 소요되기 때문에 Svelt와 같이 가상 DOM을 사용하지 않는 프레임워크가 더 빠를 수 있다. 반드시 '최고의 속도'를 보장하는 것은 아니고, 대부분의 상황에서 충분히 빠른 업데이트를 보장하는 것이다.

---

참고
https://www.youtube.com/watch?v=N7qlk_GQRJU
