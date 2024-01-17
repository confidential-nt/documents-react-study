# ✏️ Describing the UI : Your First Components

### Components: UI building blocks

컴포넌트는 리액트의 핵심 개념 중 하나로 UI를 구축하는 기반이다.
웹에서는 HTML 태그로 풍부한 구조의 문서를 만들 수 있다.

리액트를 사용하면 마크업, CSS, JS를 컴포넌트로 결합해 재사용할 수 있다.
이 때 컴포넌트란 앱에서 재사용 할 수 있는 UI 요소라고 이해하면 된다.
리액트 앱에서 모든 UI는 컴포넌트로 구성된다.

컴포넌트를 재사용하는 것은 여러 가지 이점이 있지만, 가장 대표적으로 빠르게 다양한 화면을 개발할 수 있다는 점이다.
n개의 동일한 UI를 만드는 것과 컴포넌트를 사용해 동일한 UI를 n번 사용하는 것을 상상하면 당연히 후자가 빠를 것이다.

---

### Defining a components

컴포넌트를 만드는 방법은 3가지로 나뉜다.

1. 컴포넌트 export하기
   default export / named export
2. 함수 정의하기
   컴포넌트의 이름은 반드시 대문자로 시작해야 한다.
3. 마크업 추가하기
   return 키워드와 같은 라인이 아닐 경우 소괄호로 묶어야 한다.
   HTML처럼 작성되었지만 실제로는 JS인 구문을 JSX라고 한다.

```
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

---

### Nesting and organizing components

컴포넌트는 일반적인 JS 함수이기 때문에 같은 파일에 여러 컴포넌트를 포함할 수 있다.
파일이 복잡해진다면 언제는 import/export로 파일을 나누고 불러올 수 있다.
이처럼 한 번 컴포넌트를 정의한 후에 원하는 곳에 원하는 만큼 재사용할 수 있다.

단, 컴포넌트의 정의 자체를 중첩하는 것은 속도 저하와 버그를 유발할 수 있어 지양해야 한다.
대신 최상위 레벨에서 모든 컴포넌트를 정의하는 것을 권장한다.
만약 자식 컴포넌트에 부모 컴포넌트의 특정 데이터가 필요하다면 props로 전달해야 한다.

```
export default function Gallery() {
  // 🔴 Never define a component inside another component!
  function Profile() {
    // ...
  }
  // ...
}
export default function Gallery() {
  // ...
}

// ✅ Declare components at the top level
function Profile() {
  // ...
}
```

---

출처: https://react-ko.dev/learn/your-first-component
