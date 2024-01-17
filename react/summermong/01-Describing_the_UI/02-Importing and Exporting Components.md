# ✏️ Describing the UI : Importing and Exporting Components

### The root component file

CRA에서는 앱 전체가 src/App.js에서 실행된다.
설정에 따라 루트 컴포넌트가 다른 파일에 위치할 수도 있고, Next.js처럼 파일 기반으로 라우팅하는 프레임워크의 경우 페이지별로 루트 컴포넌트가 다를 수 있다.

리액트에서 루트 컴포넌트는 애플리케이션 내의 최상위 엘리먼트를 의미한다.
일반적으로 리액트 앱에 진입점 역할을 하는 public/index.html 파일 내의 DOM 노드로 표시된다.

```
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App'; // 예시로 사용할 애플리케이션의 최상위 컴포넌트

// 루트 컴포넌트를 실제 DOM에 렌더링
ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

이 때 <App />이 루트 컴포넌트다.

```
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

React 18의 경우 Concurrent Mode를 도입해 ReactDOM.render 대신 createRoote().render()를 사용한다.
Root를 생성하고 그 Root를 통해 비동기적으로 렌더링을 진행한다.
이로써 더 부드러운 UX와 렌더링 성능을 향상 시킬 수 있다.

---

### Exporting and importing a component

export, import로 모듈성을 강화하고 컴포넌트를 재사용할 수 있다.

1. 컴포넌트를 넣을 JS 파일 생성
2. 새로 만든 파일에서 함수 컴포넌트를 export
3. 컴포넌트를 사용할 파일에서 import

이 때 export, import 하는 방식은 default / named 2가지가 있다.

default export는 파일 내 1개만 존재하고, named export는 여러 개가 존재할 수 있다.
default import를 사용할 경우 원한다면 import 키워드 뒤에 다른 이름으로 값을 가져올 수 있다.
반면 named import는 import/export하는 쪽 모두 사용하는 이름이 같아야 하기에 named로 불린다.

| 문법    | export                              | import                                |
| ------- | ----------------------------------- | ------------------------------------- |
| default | export default function Button() {} | import Button from './Button.js';     |
| named   | export function Button() {}         | import { Button } from './Button.js'; |

보편적으로 한 파일에서 하나의 컴포넌트를 export할 때 default export를,
여러 컴포넌트를 export할 때 named export 방식을 선호한다.
또한 일반적으로 코드 내 혼선을 줄이기 위해 한 가지 스타일로 통일하는 것을 지향한다.

```
// Gallery.js
export function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
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

```
// App.js
import Gallery from './Gallery.js';
import { Profile } from './Gallery.js';

export default function App() {
  return (
    <Profile />
  );
}
```

파일 확장자 없이 import된 코드도 존재한다.

```
import Gallery from './Gallery';
```

리액트에서 './Gallery.js', './Gallery' 둘 다 사용할 수 있지만 전자의 경우가 ESM 사용 방법에 더 가깝다.
ESM은 JS 모듈 시스템의 표준이며 파일 확장자를 명시적으로 쓰지 않고도 모듈을 가져오거나 내보낼 수 있다.
따라서 위와 같이 생략할 수 있으나, 파일 확장자를 명시하는 것을 권장한다.

1. 명시성과 가독성 (동명을 가지는 경우)
2. 브라우저의 표준 동작
   MIME 타입 인식의 어려움
   파일 확장자가 없으면 서버가 정확한 MIME 타입을 제공해야 함 (그렇지 않으면 파일 해석 불가)
   각 브라우저별 ESM 구현의 차이 존재

---

출처: https://react-ko.dev/learn/importing-and-exporting-components
