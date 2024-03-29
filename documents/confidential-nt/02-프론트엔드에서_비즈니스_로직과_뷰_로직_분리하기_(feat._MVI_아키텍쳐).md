[원문](https://velog.io/@teo/MVI-Architecture)

## 프론트엔드에서 비즈니스 로직과 뷰 로직 분리하기 (feat. MVI 아키텍쳐)

### 프롤로그

- 많은 사람들이 의문을 가지는 주제: 프론트엔드에서 아키텍쳐를 어떻게 가져가면 좋을까요? 어떤게 좋은 아키텍쳐일까요?
- 많은 사람들이 여러가지 방법을 생각해내며 여러가지 라이브러리와 프레임워크들이 등장.
- 특정 벤더가 존재하지 않는 웹 + 요구사항의 변화와 빠른 대응이 중요한 프론트엔드라는 직군의 특성의 만남 -> 트렌드 변화가 빠른 곳이 됨.

최근의 프론트엔드 아키텍쳐 트렌드(MVI)에 대해서 살펴보는 글.

단, 2022년에 쓰여진 글이라는 점에 유의.

---

### 프론트엔드 트렌드 변천사

#### 프론트엔드 트렌드 변화

1. HTML, CSS, JS의 탄생 - 관심의 분리와 느슨한 결합

   - 컨텐츠는 HTML, 스타일은 CSS, 간단한 인터랙션은 JS
   - 고대 시절에는 서버에서 HTML을 만들어서 반환.

2. jQuery까지의 시대 - 일단 DOM을 쉽게 쓰자.

   - AJAX 등장 -> 서버는 이제 HTML 말고 JSON을 반환
   - 프엔은 이걸 받아서, HTML + CSS + jQuery를 통해 이쁘게 가공해서 보여줌.

3. HTML + JS를 합치는게 더 낫던데? MVC(Model - View - Controller) 컴포넌트 방식의 탄생

   - 웹 어플리케이션의 복잡성 증가 -> 정적인 화면보다 인터랙션이 많이 필요해짐 -> 컨텐츠의 구성이 자바스크립트에서 더 많이 일어나더라 -> HTML과 JS를 함께 다루자
   - MVC 아키텍쳐 표방 : 데이터는 데이터끼리, DOM 조작 로직은 DOM 조작 로직끼리...하나로 관리하려는 움직임 -> 자연스럽게 화면 단위가 아니라 컴포넌트 단위로 발전
   - backbone.js..

4. 선언적으로 만들자. 데이터 바인딩 + 템플릿 = MVVM 웹 프레임워크 춘추전국시대

   - 데이터 변경 때마다 DOM을 조작해야했던 방식을 서버에서 하듯이 템플릿 방식으로 HTML을 작성하는 방안에 대해 연구 -> knockoutjs와 angaulrjs등을 바탕으로, 템플릿을 만들고 데이터를 바인딩하고 변경감지 전략을 통해 웹 서비스를 개발하는 MVVM 아키텍쳐 부상
   - 이것들은 훗날 앵귤러 새 버전, react, vue의 기반이 되었음

5. 컴포넌트간 데이터 교환이 너무 복잡해요. Container - Presenter 방식

   - 리액트의 발전 -> 컴포넌트에 대한 발전 -> 재사용가능한 컴포넌트에 대한 연구 시작
   - 비즈니스 로직을 포함하면 컴포넌트가 재사용이 쉽지 않더라 -> Container라는 상위 컴포넌트에 비즈니스 로직을 넣고 하위 컴포넌트는 Props를 통해 데이터만 받게하자.

6. Props Drill 문제 어떻게 할거임? - FLUX와 Redux

   - 위에서 언급한 방법 -> Props Driiling 문제..
   - 또한, 여러 컴포넌트가 같은 상태를 공유할 때, 어떻게 이 상태를 변경할지도 문제고 이 상태가 필요한 컴포넌트에게 상태를 전달하는 것도 문제다!
   - FLUX의 등장
   - 비즈니스 로직을 굳이 컴포넌트 계층구조로 만들 필요가 없다! MVC..뭐 이렇게 보지말고, 그냥 크게 View Layer와 비즈니스 Layer로 구분하자.
   - 데이터를 조작하는 로직을 별개로 두고 Action이라는 매개체를 통해서 데이터를 변경을 하면 이를 컴포넌트에 직접 연결해서 사용하는..
   - 이를 구현한 Redux가 주류 아키텍쳐가 됨.
   - 이때 부터 **비즈니스 로직을 컴포넌트에서 분리를 하고 별도로 관리하는 도구들이 주류**가 되면서 이러한 개념을 **상태관리(State Management)**라고 부르게 됩니다.
   - 이에 영감 받아 여러가지 상태관리 도구들이 나타난다.

7. 너무 복잡한데? - hooks와 context, Recoil, Zustand, jotai

   - 아 근데 Redux 너무 복잡하다. 너무 많은 보일러 플레이트가 필요하다.
   - 체계적으로 잘 기획된 대형 프로젝트면 ㄱㅊ. 그런데 소형 프로젝트, 그리고 빨리 만들어보고 결과를 봐야하는 그런 프로젝트에는 안맞고 오버 엔지니어링이다...
   - 이를 위해 React: hooks와 ContextAPI 제공
   - 그러나 hook으로는 전역 상태 관리를 하기엔 부족.(hook은 애초에 로컬 상태 관리 + 로직의 재사용을 위해 태어난 아이이니..) ContextAPI도 사용하기에도 좀 복잡하고, 복잡한 상태 관리를 하기에는 그리 알맞는 도구는 아님.(미미하나 성능적으로도 좋지 않음. 해당 상태가 필요없는 컴포넌트까지 리렌더링이 발생해야하니까..)
   - 그래서 좀 더 간결한 형태인 Recoil, Zudstands, Jotai들이 부상

8. 어차피 대부분 서버 api를 관리 하려고 쓰는 거 아님? React Query, SWR, Redux Query

   - 프론트엔드 대부분의 전역적인 상태관리가 필요한 이유는 서버와의 API에 있음.
   - 전역 상태관리를 통해 비즈니스 로직을 관리하기보다 다이렉트로 백엔드와 직접 연동을 하면서 필요한 로딩, 캐싱, 무효화, 업데이트 등 기존 상태관리에서 복잡하게 진행해야했던 로직들을 단순하게 만들자!
   - API를 통한 전역 상태관리가 단순하게 되는 결과

9. 그 밖에 소개하지 않은 눈여겨 볼만한 상태관리도구
   - 우리가 기존에 사용하던 문법, 혹은 복잡하지 않은 문법을 통해 상태관리를 하고 Nested Object의 상태관리도 쉽게해주는 그런 방향
   - valtio, Hookstate 등...

---

### (MVC → MVP → MVVM → ) 그리고 MVI 아키텍쳐

#### User Interface

- 보통 사람들은 컴퓨터 자료구조 지식에 대해 잘 모름
- 그런 사용자들이 쉽게 컴퓨터와 상호작용 하도록 만든게 UI
- 데이터와 사용자 사이에 있는 가상의 매개체
- 기존 프로그램의 Input와 Ouput 사이에 **화면과 행동**을 추가함으로써 새로운 Input과 Ouput을 사용자로 부터 받도록 만드는 매개체의 역할
- 두 가지의 흐름
  - 데이터 → (화면) → 사용자 : 데이터의 시각화
  - 사용자 → (행동) → 데이터: 사용자의 행동으로 데이터를 조작
    - 전달받은 화면을 통해서 데이터를 보면서 외부의 입력장치를 통해서 데이터를 조작 하기 위해 특정한 **행동(Behavior)**(ex 버튼을 클릭)을 함. 그러면 프로그램은 사용자의 발생시킨 **이벤트(Event)**를 통해서 사용자 행동의 **의도(Intent)**에 맞는 **동작(Action)**을 전달하게 되면 이에 맞게 적절한 데이터의 변화가 발생하게 되고 이는 다시 첫번째로 돌아가게 되면서 순환구조를 이루게 됨.

#### MVI? Model - View - Intent

- 사용자가 **다른 동작을 해도 실제로 같은 데이터의 변화**를 의미하는 경우가 있음.
- 키보드의 위쪽 방향키를 누르는 것, + 버튼을 누르는 것 -> 다른 동작이지만 같은 데이터의 count를 1 증가 시킨다고 가정.
- 이를 통해서 비즈니스로직을 2가지 레이어로 나눌 수 있다는 걸 알 수 있음.
  - 사용자가 View를 통해서 전달한 UI Event를 **어떠한 데이터 변화를 하게 할지 전달하는 역할** (**사용자의 의도를 파악**한다 -> Intent)
  - 전달받은 **요청에 따라서 적절히 데이터를 변화하는 역할** (데이터를 다루므로 Model)
- 이래서 MVI다.

#### MVI의 특징

- 기존의 MVC나 MVVM과는 다른 점은 이 구성이 하나의 컴포넌트가 아니라 앱 전체에 적용되는 흐름이라는 것.
- 전체적으로 데이터의 방향성이 단방향으로 연결이 됨.
  - 데이터의 흐름을 이해하고 디버깅을 하기 쉬워짐.
- 데이터가 전역적으로 구성이 됨
- 뷰는 모델에 의존적이지만 비즈니스 로직은 뷰와의 의존성이 없기 때문에 UI 요구사항에 대한 변화에 유연하며, 별도로 테스트를 하기 쉬움.
- 반대로 뷰는 비즈니스 로직에 의존적이지만 뷰끼리는 느슨하게 결합되어 UI 요구사항 변화에 긴밀하게 대응할 수 있게됨.
- 일관성있는 상태를 유지하기에 용이.
- View의 생명주기와 무관하게 일관성 있는 상태를 가짐. 컴포넌트 생명주기에 따른 상태 동기화 문제를 해결
  - 참고: 컴포넌트 생명주기에 따른 상태 동기화 문제?
    - 비동기 작업: 컴포넌트가 언마운트된 후 비동기 작업(예: 네트워크 요청)이 완료되고 그 결과를 컴포넌트 상태에 반영하려 할 때, 이미 존재하지 않는 컴포넌트를 업데이트하려고 하면 문제가 발생합니다.
    - 의존성 변경: 컴포넌트가 의존하는 외부 데이터나 상태가 변경되었을 때, 이 변경사항이 컴포넌트의 상태와 제대로 동기화되지 않으면 예상치 못한 동작이 발생할 수 있습니다.
    - 부적절한 상태 관리: 상태가 여러 컴포넌트에 걸쳐 있거나, 복잡한 상태 관리 로직으로 인해 생명주기 메소드가 예상대로 작동하지 않을 수 있습니다.
    - 즉, 잘못된 상태 관리로 인해 오류가 발생하는 문제들..

그렇다. 사실 FLUX 패턴, Redux 패턴, 상태관리등 그 동안 프론트엔드에서 줄곧 제시되었던 방법들을 잘 정리하여 아키텍쳐로써 설명을 하고 있는 모양새다.

#### 프론트엔드 아키텍쳐의 방향성 요약정리

- 재사용이 가능한 독립적인 컴포넌트를 조립하여 서비스를 구성한다는 기존 구조와 달리 뷰와 비즈니스로직을 완전히 분리하여 생각
- 비즈니스 로직은 1) 데이터를 관장하는 Model과 2)사용자의 행동을 데이터 변화로 매핑하는 Intent영역으로 분리
- Model은 다시 1) 변화를 감지하고 변경사항을 전파하는 영역과 2) 데이터를 변화하는 로직으로 구성
- 이를 통해 View에서는 Model로 부터 데이터를 조회하는 Query와 데이터를 변화시키는 Command를 언제든 조립해서 사용(CQRS 패턴) 할 수 있게 되어 뷰는 비즈니스 로직에 의존적이지만 뷰끼리는 느슨하게 결합되어 UI 요구사항 변화에 긴밀하게 대응할 수 있게됨.
- 비즈니스 로직도 마찬가지. 뷰와의 의존성이 없기 때문에 UI 요구사항에 대한 변화에 유연하며, 별도로 테스트를 하기 쉬움.

#### 그렇다면 기존 방법은 안 좋은 것일까?

- 그렇다고 전통적인 독립적인 UI 컴포넌트가 필요없다는 얘기는 x
- **'컴포넌트 계층의 맨 아래에 있는' 재사용이 가능한 독립적인 UI 컴포넌트를 조립하여 화면을 만드는 전략은 여전히 유효**
- 그러나 Props Drilling을 유발하며 경직된 구조를 만들어내는 중간 계층의 컴포넌트들은 사실 독립적일 필요도 없고 재사용도 되지 않는 컴포넌트인 경우가 많음.
- 이러한 컴포넌트들은 대개 비즈니스 로직를 담당하기 때문에 거대한 컴포넌트(Massive-View-Controller) 되기 마련
- 컴포넌트는 독립성을 유지하고 의존성을 최소화 해야한다는 원칙에 따라 최대한 props를 통해서 컴포넌트 계층 구조를 유지하던 방식이 되려 경직된 구조를 만들어냈기에 **어차피 재사용하지 못할 컴포넌트라면 비즈니스 모듈에 의존적이더라도 언제든 교체가능한 방식으로 만들어내는 것이 변화에 대응하기 좋다**는 뜻.
- 즉, Props Drilling 유발할바엔 비즈니스 로직을 다른 곳으로 빼내고, 뷰든 비즈니스로직 쪽이든 둘다 유연하게 대응할 수 있도록 만들어라라는 뜻.

#### One more thing: Intent와 Reducer

- Flux 패턴에서 Reducer는 왜 이렇게 작성을 해야하는지에 대해서 의문을 품는 사람들이 있음. 일반적으로는 보통 이벤트 핸들러에서 직접 모델의 여러가지 데이터들의 변화를 기술하는 식으로 작성을 하는 경우가 많음. 다음과 같이.

```javascript

// View에서 직접 여러가지 값들의 변화를 기술한다.
const onClick = () => {
  setVisible(!visible)
  setCount(count + 1)
  setTodos([...todos, {title: "할일추가"})
}
```

- 문제는 되진 않지만, 나중에 디버깅할때 하나하나 추적을 해야하기 때문에, 어디서 문제가 발생한지도 파악하기도 힘들어질 수 있기 때문에 좀 힘들어질 수가 있음..
- 뷰에서 직접 데이터를 수정하도록 작성을 하게 되면,(즉, 뷰에서 비즈니스 로직을 작성하게되면) 뷰가 모델에 의존하는 건 어쩔 수 없다지만 위와 같이 작성하게 되면 모델도 뷰에 의존하게 되어서 뷰에 변화에 따라 비즈니스로직도 바꿔서 작성해야할수도 있음. ex) onClick의 구조가 바뀌어야하는 상황이 생긴다면...충분히 이런 상황이 발생할 수 있음.
- 뷰에서는 의도만을 전달하고 의도에 맞는 데이터 변환은 모델에서만 처리할 수 있도록 만들어 데이터를 변화하는 코드를 Model 모듈에 모으게 되면 응집도가 높아지고 추후 디버깅에 용이하며 특히 뷰가 비즈니스로직으로 부터 느슨한 결합을 할 수 있도록 만들어 줆.

#### 이해를 돕기 위한 의사코드

```javascript
// Action
export const _INCREASE = action("_INCREASE");
export const _DECREASE = action("_DECREASE");
export const _RESET = action("_RESET");
```

```javascript
// Model
export const store = createStore({ count: 0 }, (state) => {
  on(_INCREASE, (value) => (state.count += value));

  on(_DECREASE, (value) => (state.count -= value));

  on(_RESET, () => (state.count = 0));
});
```

```javascript
// Component
export const App = (props) => {
  // Query
  const count = SELECT(store.count);

  // Computed Value
  const doubledCount = count * 2;

  // Intent
  const onClick = () => dispatch(_INCREASE(+1));

  // View
  return <button onCLick={onClick}>{count}</button>;
};
```

이런식으로 작성하게 되면 아무튼 유연하게 화면을 구성할 수 있다.

#### 못다한 이야기들

(MVI라는 용어에 집착하지 말고) 그냥 이러한 트렌드 흐름이다..정도로 읽으면 될 것 같다.

---

### 마무리

대강, 뷰와 비즈니스 로직을 거시적으로 분리하고 단방향 흐름을 만들어내고 전역적인 스토어에서 변경사항을 전파한다는 흐름이다.

패러다임과 아키텍쳐는 실제가 아니라 방향성. 내가 쓰고 있는 라이브러리가 정확히 이러한 형태에 맞지 않을 수 있지만 이러한 대략적인 방향성과 흐름을 이해한다면 여러가지 상태관리 라이브러리를 선택하거나 컴포넌트의 구조를 설계할때 더 넒은 시각으로 바라볼 수 있을 거라고 생각하신다고..(동의)
