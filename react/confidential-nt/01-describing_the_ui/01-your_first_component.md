## Your First Component

### 들어가면서

컴포넌트는 리액트에서 매우 중요한 컨셉이고 UI의 근간이 되는 요소다. 함께 알아가보도록 하자. 컴포넌트가 뭔지 어떻게 정의하고 사용하는지.

### 컴포넌트는 UI를 구성하는 블록이다.

웹에서, html을 예로 든다면 다음과 같은 형태의 마크업을 본적이 있을 것이다.

```html
<article>
  <h1>My First Component</h1>
  <ol>
    <li>Components: UI Building Blocks</li>
    <li>Defining a Component</li>
    <li>Using a Component</li>
  </ol>
</article>
```

저런식으로 마크업을 하고, css 파일에서 스타일을 정의하고, 자바스크립트 파일에서 상호작용 로직을 작성하고... 보통은 이런 식으로 흘러간다.

**그런데 리액트는 마크업, 스타일, 상호작용에 관한 정의 (즉, UI에 관한 모든 정의)를 컴포넌트라는 하나의 공간에서 할 수 있게 해준다.**

그리고 이 컴포넌트는 정의만 해놓으면 어디서든지 재사용 가능하다. html을 태그를 사용하는 것처럼, 페이지를 디자인 하기 위해 컴포넌트를 중첩하고 조합하고 나열할 수 있다.

- 재사용 가능성이 중요한 이유: 한번만 정의해놓고 필요한 곳에 가져다 씀으로서 개발 속도를 향상 시켜줌.

### 컴포넌트 정의

**리액트 컴포넌트는 마크업을 뿌려주는 자바스크립트 함수다.**

컴포넌트를 어떻게 정의하는지는 생략.

다만 몇 가지 유의점에 관하여 정리.

1. 컴포넌트 자체는 그냥 자바스크립트 함수 문법이지만, 반드시 이름을 **대문자로 지정**해야함. 안그러면 작동 안함
2. 컴포넌트는 마크업을 리턴하는데, **html과 똑같이 생겼지만 사실 기저에는 자바스크립트가 깔린 기술**임. JSX라고 불림.
3. JSX를 리턴할 때는, return 문과 같은 라인에 jsx를 두거나, 같은 라인에 두지 않는다면 JSX를 괄호로 감싸야 함. (그렇지 않으면 return만 인식해서 그대로 함수를 종료해버리고 JSX를 렌더링하지 않음)
4. **컴포넌트는 다른 컴포넌트를 렌더할 수 있지만, 컴포넌트 정의 안에 다른 컴포넌트를 정의하면 안됨.** -> 버그를 일으키고 성능저하를 일으킬 수 있음. 반드시 컴포넌트는 top level에 정의하라.

### 컴포넌트를 사용했을 때 리액트는...

JSX를 보고,

1. 첫글자가 소문자다 -> HTML tag라는 것을 인식.
2. 첫글자가 대문자다 -> Component라는 것을 인식.

### Deep Dive

**컴포넌트를 오직 버튼과 같이 재사용 가능한 UI 조각만을 위해 작성하지는 않음.** 사이드 바, 리스트, 더 나아가 페이지와 같이 더 큰 구성요소에도 사용. 컴포넌트는 UI를 구성하는 좋고 간단한 방법이기에, 해당 컴포넌트가 한번만 사용된다고 해도 컴포넌트로 정의함.

React-based frameworks take this a step further. Instead of using an empty HTML file and letting React “take over” managing the page with JavaScript, they also generate the HTML automatically from your React components. This allows your app to show some content before the JavaScript code loads. -> Next.js가 첫 페이지 로드를 빠르게 하기 위해 수행하는 기능 중 하나.
