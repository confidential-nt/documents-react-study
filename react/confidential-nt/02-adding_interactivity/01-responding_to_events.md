## Responding to Events

### 들어가면서

리액트는 JSX에 이벤트 핸들러를 부착할 수 있게 해준다. 이벤트 핸들러는 사용자가 UI를 통해 상호작용을 했을 때, 그에 대한 반응으로 호출되는 함수이다.

### 이벤트 핸들러 부착하기

이벤트 핸들러를 정의한 다음, 올바른 JSX 태그에다가 프롭으로 전달해줘야함.

이벤트 핸들러는 보통 컴포넌트 안에서 정의되며, 'handle + 이벤트이름'으로 이름을 지음.

- 주의사항
  ```javascript
  <button onClick={alert('You clicked me!')}>
  ```
  위와 같이 핸들러를 정의하게 되면, 컴포넌트가 렌더 될 때 핸들러가 호출이 됨. (의도한 대로) 클릭했을 때 호출이 안됨.

### 이벤트 핸들러를 프롭으로 전달하기

사용자 정의 컴포넌트가 이밴트 핸들러를 프롭으로 받을 수 있도록 정의 할 수 있음.

이렇게 하면 때에 따라 컴포넌트를 좀 더 유연하게 재사용할 수 있음

If you use a design system, it’s common for components like buttons to contain **styling** but not specify behavior. Instead, components like PlayButton and UploadButton will pass event handlers down. -> 디자인 시스템처럼, 여기저기서 컴포넌트를 재사용해야하는 경우, 컴포넌트 안에서, 이벤트에 대하여 어떻게 행동할 지 정의하기 보다는 사용하는 측에서 프롭을 통해 커스터마이징 할 수 있도록 한다는 의미.

### 이벤트 핸들러를 전달받는 프롭의 이름

빌트인 컴포넌트(div, button)은 오직 정해진 이름만을 프롭으로 전달 받을 수 있지만, 사용자 정의 컴포넌트는 마음대로 정할 수 있음.

일반적으로 'on..'으로 시작함.

이것을 이용해서, 상황에 따라 해당 컴포넌트가 하는 일에 잘 맞는 이름을 프롭으로 지어주자. -> 이러면 컴포넌트의 내부 구현 사항을 몰라도 해당 컴포넌트가 무엇을 하는지 알 수 있으니까..

- 주의사항

  이벤트 핸들러를 부착할 때, 더 적절한 태그를 사용할 수 있도록 해라. 예를 들어 onClick 이벤트 핸들러는 div에도 붙일 수 있지만 button에 붙이는 게 더 적절함. 그리고 button으로 구현해야 키보드 네비게이션 같은 브라우저의 기본 동작을 할 수가 있으며 이는 웹 접근성에 영향 미침.

### 이벤트 전파

당연히 버블링됨. 이 전파를 막으려면 이벤트를 발생시킬 자식 컴포넌트에서 e.stopPropagation()을 호출해주면 된다.

사용자의 혼란을 방지하려면 이벤트 전파에 관해서도 잘 고려해야한다.

- 캡쳐링에 관하여

  캡쳐링을 사용해야 할때가 간혹 있음. stop propagation을 사용했더라도, 자식 컴포넌트에 발생하는 이벤트를 전부 알고 싶을 때..

  예를 들면, 전파 로직에 상관없이, 특정 컴포넌트에 발생한 클릭을 분석하기 위해 로깅을 하거나 그럴 때...

  그러고 싶으면 이벤트 이름 뒤에 'Capture'를 붙이면 캡쳐링을 할 수 있음

  ```javascript
  <div onClickCapture={() => { /* this runs first */ }}>
  // ...
  ```

  Capture events are useful for code like routers or analytics, but **you probably won’t use them in app code.**

### 이벤드 전파 대신 핸들러를 패스하세요.

이벤트가 발생했을 때 부모 컴포넌트에서 추가적으로 해야할 일이 있다면, 전파를 사용하지 말고, 자식 컴포넌트로 prop을 통해 전달해라. 다음과 같이. 그래야 추적이 쉽고 혼란을 방지할 수 있다.

```javascript
function Button({ onClick, children }) {
  return (
    <button
      onClick={(e) => {
        e.stopPropagation();
        onClick(); // *
      }}
    >
      {children}
    </button>
  );
}
```

### 브라우저 기본 이벤트 막는 방법

생략

### 이벤트 핸들러는 사이드 이펙트를 가질 수 있나요?

이벤트 핸들러는 사이드 이펙트를 구성하기에 알맞다. 이벤트가 발생했을 때, 그에 따라 '무언가를 변경하기'에 알맞은 곳이다. 그런데 무언가를 바꾸려면 그 무언가를 어디엔가 저장해야하는데, 그 store가 바로 state다. (컴포넌트의 메모리라고도 할 수 있다)
