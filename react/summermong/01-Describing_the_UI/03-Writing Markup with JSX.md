# ✏️ Describing the UI : Writing Markup with JSX

### JSX: Putting markup into JavaScript

웹은 HTML, CSS, JS 기반으로 만들어져왔다.
HTML로 콘텐츠, CSS로 디자인, JS로 로직을 작성해 각각의 분리된 파일을 관리했다.
하지만 웹의 상호작용이 많아지면서 '로직이 콘텐츠를 결정하는 경우'가 많아졌다.
그래서 JS가 HTML을 담당하게 되었고, 리액트에서 렌더링 로직과 마크업이 같은 위치의 컴포넌트에 존재하게 되었다.

각 React 컴포넌트는 리액트가 브라우저에 마크업을 렌더링할 수 있는 JS 함수다.
React 컴포넌트는 JSX라는 구문 확장자로 마크업을 표현한다.
JSX은 HTML과 유사해보이지만 보다 엄격하며 동적으로 정보를 표시할 수 있다.

---

### Converting HTML to JSX

JSX의 규칙은 아래와 같다.

1. 단일 루트 엘리먼트를 반환해야 한다.

   컴포넌트에서 여러 엘리먼트를 반환하려면 부모 태그로 랩핑해야 한다.

```
<>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</>
```

이렇게 <></>와 같이 빈 태그를 Fragment라고 한다.
이를 통해 브라우저 상의 HTML 트리 구조에서 흔적 없이 그룹화를 할 수 있다.

📌 단일 루트 엘리먼트를 반환해야 하는 이유

JSX는 HTML처럼 보이지만 내부적으로는 JS 객체로 변환된다.
하나의 배열로 감싸지 않은 하나의 함수에서 두 개의 객체를 반환할 수 없다.
따라서 또 다른 태그나 Fragment로 감싸지 않으면 두 개의 JSX 태그를 반환할 수 없다.

2. 모든 태그를 닫아야 한다.

   JSX에서는 태그를 명시적으로 닫아야 한다.
   자체적으로 닫는 img 태그도 반드시 <img /> 형식으로 닫아줘야 한다.

```
<>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
   />
  <ul>
    <li>Invent new traffic lights</li>
    <li>Rehearse a movie scene</li>
    <li>Improve the spectrum technology</li>
  </ul>
</>
```

3. 대부분 카멜 케이스를 사용한다.

   JSX는 JS로 바뀌고, JSX로 작성된 어트리뷰트는 JS 객체의 키가 된다.
   컴포넌트 안에서 어트리뷰트를 변수로 읽고 싶은 경우가 있을 수 있다.
   이 때 JS의 변수명에는 제한이 있어 대시를 포함하거나 예약어를 사용할 수 없다.

다만 역사적인 이유로 aria-*, data-*의 어트리뷰트는 HTML에서와 동일하게 사용할 수 있다.
화이트리스트란 특정 어트리뷰트나 속성을 명시적으로 허용하는 목록을 가리키는데,
React 16에서 화이트리스트가 완화됨에 따라 사용자 정의 어트리뷰트가 허용되었다.
따라서 사용자는 자유롭게 aria나 data 같은 사용자 정의 어트리뷰트를 사용할 수 있게 됐다.

---

출처: https://react-ko.dev/learn/writing-markup-with-jsx
