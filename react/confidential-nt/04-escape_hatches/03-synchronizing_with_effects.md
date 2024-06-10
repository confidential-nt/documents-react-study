# Synchronizing with Effects

## 들어가면서

일부 컴포넌트는 외부 시스템과 동기화해야. 예를 들어, React state를 기반으로 React가 아닌 컴포넌트를 제어하거나 서버 연결을 설정하거나 컴포넌트가 화면에 나타날 때 분석 로그를 보낼 수 있음. Effect를 사용하면 렌더링 이후 일부 코드를 실행할 수 있으므로 컴포넌트를 React 외부의 시스템과 동기화할 수 있음.

## Effect란 무엇이며 이벤트와는 어떤게 다른가요?

Effect에 도달하기 전에 React 컴포넌트 내부의 두가지 유형의 논리에 익숙해져야.

- 렌더링 코드(UI 구성하기에서 소개됨)는 컴포넌트의 최상위 레벨에 있음. 여기서 props와 state를 가져와 변환하고 화면에 표시할 JSX를 반환함. 렌더링 코드는 순수해야. 수학 공식처럼 결과만 계산할 뿐 다른 작업은 수행하지 않음.
- 이벤트 핸들러(상호작용 추가하기에서 소개됨)는 컴포넌트 내부에 있는 중첩된 함수로, 계산만 하는 것이 아니라 별도의 작업도 수행함. 이벤트 핸들러에서는 입력 필드를 업데이트하거나, HTTP POST요청을 제출하여 제품을 구매하거나, 사용자를 다른 화면으로 이동할 수 있음. 이벤트 핸들러에는 특정 사용자 작업(예:버튼 클릭 또는 입력)으로 인해 발생하는 “사이드 이펙트”(프로그램의 state를 변경함)가 포함되어 있음.

**때로는 이것만으로는 충분하지 않을 수 있음.** 화면에 표시될 때마다 채팅 서버에 연결해야 하는 ChatRoom 컴포넌트를 고려해보자. 서버에 연결하는 것은 순수한 계산이 아니므로(사이드 이펙트) 렌더링 중에 발생할 수 없음. 그러나 ChatRoom 표시를 촉발하는 클릭과 같은 특정한 단일 이벤트는 없음.

Effect를 사용하면 특정 이벤트가 아닌 **렌더링 자체로 인해 발생하는 사이드 이펙트**를 명시할 수 있음. 채팅에서 메시지를 보내는 것은 사용자가 특정 버튼을 클릭함으로써 직접적으로 발생하기 때문에 이벤트임. 그러나 서버 연결을 설정하는 것은 상호작용에 관계없이 발생해야 하기 때문에 하나의 Effect임. **Effect는 화면 업데이트 후 커밋이 끝날 때 실행됨.** 이 때가 React 컴포넌트를 일부 외부 시스템(네트워크 또는 서드파티 라이브러리와 같은)과 동기화하기에 좋은 시기.

- 참고: 이 글에서 대문자로 시작하는 “Effect”는 위의 React에 한정된 정의, 즉,렌더링으로 인해 발생하는 사이드 이펙트를 나타냄. 더 넓은 프로그래밍 개념을 언급할 때는 “사이드 이펙트”라고 지칭.

## Effect가 필요하지 않을 수도 있습니다

컴포넌트에 Effect를 추가하고자 서두르지 마라. **Effect는 일반적으로 React 코드에서 벗어나 일부 외부 시스템과 동기화하는 데에 사용된다는 점을 명심하라.** 여기에는 브라우저 API, 서드파티 위젯, 네트워크 등이 포함됨. **Effect가 다른 state를 기반으로 일부 state만을 조정하는 경우,** Effect가 필요하지 않을 수도 있음.

## Effect 작성 방법

3가지 단계.

1. Effect를 선언한다. 기본적으로 Effect는 모든 렌더링 후에 실행됨.
2. Effect의 의존성을 명시. 대부분의 Effect는 렌더링 할 때마다가 아니라 필요할 때만 다시 실행해야 함. 예를 들어, 페이드 인 애니메이션은 컴포넌트가 나타날 때만 발동되어야 함. 대화방 연결 및 해제는 컴포넌트가 나타났다가 사라지거나 대화방이 변경될 때만 발생해야함. 의존성을 지정하여 이를 제어하는 방법은 이 글 아래에서 다룸.
3. 필요한 경우 클린업을 추가. 일부 Effect는 수행중이던 작업을 중지, 취소 또는 정리하는 방법을 명시해야함. 예를 들어, “connect”에는 “disconnect”가 필요하고 “subscribe”에는 “unsubscribe”가 필요하며 “fetch”에는 “cancel”또는 “ignore”가 필요함. 클린업 함수를 반환하여 이를 수행하는 방법은 이 글 아래에서 다룸.

## Step 1: Effect를 선언하세요

어떻게 선언하는지는 알 것. 스킵.

컴포넌트가 렌더링될 때마다 React는 화면을 업데이트하고 useEffect 내부의 코드를 실행함. **즉, useEffect는 해당 렌더링이 화면에 반영이 될 때까지 코드 조각의 실행을 “지연”.**

Effect를 사용하여 외부 시스템과 동기화하는 방법을 살펴보자. \<VideoPlayer> React 컴포넌트를 생각해봐라. isPlaying prop을 전달하여 재생 또는 일시 중지 여부를 제어하면 좋을 것 같음.

커스텀 컴포넌트 VideoPlayer는 브라우저 빌트인 \<video>태그를 렌더링.

```javascript
function VideoPlayer({ src, isPlaying }) {
  // TODO: do something with isPlaying
  return <video src={src} />;
}
```

그러나, 브라우저 \<video>태그에는 isPlaying prop이 없음. 이를 제어하는 유일한 방법은 DOM요소에서 play() 와 pause() 메서드를 수동으로 호출하는 것. **비디오가 현재 재생되어야 하는지 여부를 알려주는 isPlaying prop값을 play() 및 pause()와 같은 함수를 호출과 동기화해야.**

먼저 \<video> DOM 노드에 대한 ref를 가져오기.

렌더링 중에 play() 또는 pause() 함수를 호출하고 싶을 수 있지만, 이는 올바르지 않음.

```javascript
unction VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause(); // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />;
}
```

**이 코드가 올바르지 않은 이유는 렌더링 중에 DOM 노드로 무언가를 시도하기 때문. React에서 렌더링은 JSX의 순수한 계산이어야 하며 DOM 수정과 같은 사이드 이펙트를 포함해서는 안됨.**

**더구나 VideoPlayer가 처음 호출될 때 DOM은 아직 존재하지 않는다!** React는 JSX를 반환하기 전까지는 어떤 DOM을 생성할지 모르기 때문. play()나 pause()를 호출할 DOM 노드가 아직 없는 상태.

해결책은 사이드 이펙트를 useEffect로 감싸 렌더링 계산 밖으로 옮기는 것.

```javascript
import { useEffect, useRef } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

DOM 업데이트를 Effect로 감싸면, React가 먼저 화면을 업데이트하도록 할 수 있음. 그런 다음 Effect가 실행됨.

이제 원하는 대로 작동함.

이 예제에서 React state에 동기화한 “외부 시스템”은 브라우저 미디어 API였음. 비슷한 접근 방식으로 React가 아닌 레거시 코드(예: jQuery 플러그인)를 선언적인 React 컴포넌트로 감쌀 수 있음.

동영상 플레이어 제어는 실제로는 훨씬 더 복잡하다는 점에 유의. play() 호출이 실패할 수도 있고, 사용자가 브라우저 빌트인 컨트롤을 사용하여 재생하거나 일시정지할 수도 있음. 이 예시는 매우 단순하고 불완전.

- 참고: 기본적으로 Effect는 매번 렌더링 후에 실행됨. 그렇기 때문에 무한 루프를 생성할 수가 있음.
  - 특히 useEffect 내부에서 상태를 set하는 경우..
  - Effect는 렌더링의 결과로 실행됨. state를 설정하면 렌더링을 촉발. Effect에서 즉시 state를 설정하는 것은 전원 콘센트를 꽂는 것과 같음. Effect가 실행되고, state를 설정하면 다시 렌더링이 발생하고, 다시 렌더링이 발생하면 Effect가 실행되고, 다시 state를 설정하면 또 다시 렌더링이 발생하는 식
  - Effect는 보통 컴포넌트를 외부 시스템과 동기화해야 합니다. 외부 시스템이 없고 다른 state를 기반으로 일부 state만 조정하려는 경우 Effect가 필요하지 않을 수도 있습니다.
  - Effect는 보통 컴포넌트를 외부 시스템과 동기화해야 할때 사용하는 것. **외부 시스템이 없고 다른 state를 기반으로 일부 state만 조정하려는 경우 Effect가 필요하지 않을 수도 있음.**

## Step 2 : Effect 의존성을 지정하세요

기본적으로 Effect는 매번 렌더링 후에 실행됩니다. 하지만 이를 원하지 않는 경우가 있음.

- 때로는 속도가 느릴 수 있음. 외부 시스템과의 동기화가 항상 즉각적인 것은 아니므로 꼭 필요한 경우가 아니라면 동기화를 건너뛰는 것이 좋음. 예를 들어, 키 입력 시마다 채팅 서버에 다시 연결하고 싶지 않을 수 있음.
- 때로는 잘못된 경우가 있음. 예를 들어, 키 입력 시마다 컴포넌트 페이드인 애니메이션을 발동시키고 싶지 않을 수 있음. 애니메이션은 컴포넌트가 처음 나타날 때 한 번만 재생되어야 함.

useEffect 호출의 두 번째 인자로 의존성 배열을 지정하여 React가 불필요하게 Effect를 다시 실행하지 않도록 지시할 수 있음. 일단 빈 [] 배열을 추가하는 것으로 시작.

React useEffect 훅에 누락된 의존성이 있다면 오류가 표시될 것.

> React Hook useEffect has a missing dependency: 'isPlaying'. Either include it or remove the dependency array.

문제는 Effect 내부의 코드가 isPlaying prop에 의존하여 수행할 작업을 결정하는데(예시에서는 if 문의 조건으로서), 이 의존성이 명시적으로 선언되지 않았다는 것. 이 문제를 해결하려면 의존성 배열에 isPlaying을 추가해야.

의존성 배열로 [isPlaying]을 지정하면 isPlaying이 이전 렌더링 때와 같으면 Effect를 다시 실행하지 않아도 된다고 React에 알려줄 수 있음.

의존성 배열은 여러 개의 의존성을 포함할 수 있음. React는 지정한 모든 의존성의 값이 이전 렌더링 때와 정확히 동일한 경우에만 Effect의 재실행을 건너뜀. React는 Object.is 비교를 사용해 의존성 값을 비교함. 자세한 내용은 useEffect를 참고.

**의존성을 “선택”할 수 없다는 점에 유의.** 지정한 의존성들이 Effect 내부의 코드를 기반으로 React가 예상하는 것과 일치하지 않으면 lint 오류가 발생. 이는 코드에서 많은 버그를 잡는 데 도움이 됨. 일부 코드가 다시 실행되는 것을 원하지 않는다면, 해당 의존성을 “필요”하지 않도록 Effect 코드 자체를 편집하라.

- 참고: 의존성 배열이 없는 경우와 비어 있는 [] 의존성 배열이 있는 경우의 동작은 다름.

  ```javascript
  useEffect(() => {
    // This runs after every render
    // 렌더시마다 실행됩니다.
  });

  useEffect(() => {
    // This runs only on mount (when the component appears)
    // 오직 마운트시(컴포넌트가 나타날 때)에만 실행됩니다.
  }, []);

  useEffect(() => {
    // This runs on mount *and also* if either a or b have changed since the last render
    // 마운트시 뿐만 아니라 a 또는 b가 직전 렌더와 달라졌을 때에도 실행됩니다.
  }, [a, b]);
  ```

- 참고: 의존성 배열에서 ref가 생략된 이유는 무엇인가요?
  - 이 Effect는 ref와 isPlaying을 모두 사용하지만, 의존성으로 선언된 것은 isPlaying뿐
    ```javascript
    function VideoPlayer({ src, isPlaying }) {
    const ref = useRef(null);
    useEffect(() => {
    if (isPlaying) {
        ref.current.play();
    } else {
        ref.current.pause();
    }
    }, [isPlaying]); // [isPlayinf, ref] ??
    ```
  - **이는 ref 객체가 안정적인 정체성을 가지고 있기 때문. React는 렌더링할 때마다 동일한 useRef 호출에서 항상 동일한 객체를 얻을 수 있도록 보장.** **절대 변하지 않으므로 그 자체로 Effect가 다시 실행되지 않음.** 따라서 포함 여부는 중요하지 않음. 포함해도 괜찮.
  - useState가 반환하는 설정자 함수도 안정된 정체성을 가지므로 의존성에서 생략되는 경우가 많음. linter를 통해 오류 없이 의존성을 생략할 수 있다면 그렇게 해도 안전.
  - **언제나 안정적인 의존성을 배제하는 것은 linter가 객체가 안정적이라는 것을 “확인할” 수 있을 때에만 잘 동작함.** 예를 들어, **부모 컴포넌트에서 ref가 전달된 경우 의존성 배열에 이를 지정해야 함. 부모 컴포넌트가 항상 동일한 ref를 전달하는지, 아니면 여러 ref 중 하나를 조건부로 전달하는지 알 수 없기 때문에** 이 방법이 좋음. 이 경우 Effect는 어떤 ref가 전달되는지에 따라 달라지게 됨.

## Step 3 : 필요한 경우 클린업을 추가하세요

다른 예시를 살펴보자. 채팅 서버가 나타날 때 채팅 서버에 연결해야 하는 ChatRoom 컴포넌트를 작성하고 있다고 가정. connect() 와 disconnect() 메서드가 있는 객체를 반환하는 createConnection() API가 주어짐. 컴포넌트가 사용자에게 표시되는 동안 어떻게 연결 상태를 유지할 수 있을까?

Effect 로직 작성부터 시작

```javascript
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

다시 렌더링할 때마다 채팅에 연결하는 속도가 느려지므로, 의존성 배열을 추가.

Effect 내부의 코드는 props나 state를 사용하지 않으므로 의존성 배열은 [](비어 있음). 이는 컴포넌트가 “마운트”될 때, 즉,화면에 처음 나타날 때만 이 코드를 실행하도록 React에 지시.

코드를 실행해보면..이 Effect는 마운트 시에만 실행되므로 콘솔에서 "✅ Connecting..."이 한 번 인쇄될 것으로 예상할 수 있음. 하지만 콘솔을 확인해보면 "✅ Connecting..."이 두 번 출력됨. 왜?

ChatRoom 컴포넌트가 다양한 화면으로 구성된 큰 앱의 일부라고 상상해 보자. 사용자는 ChatRoom 페이지에서 여정을 시작. 컴포넌트가 마운트되고 connection.connect()를 호출. 그런 다음 사용자가 다른 화면(예: 설정 페이지)으로 이동한다고 하자. ChatRoom 컴포넌트가 마운트 해제됨. 마지막으로 사용자가 뒤로 가기(Back)를 클릭하면 ChatRoom이 다시 마운트됨. 이렇게 하면 두 번째 연결이 설정되지만, 여전히 첫 번째 연결은 해제되지 않음! 사용자가 앱을 돌아다닐 때마다 연결은 계속 쌓이게 될 것.

이와 같은 버그는 광범위한 수동 테스트 없이는 놓치기 쉬움. **버그를 빠르게 발견할 수 있도록 개발 모드에서 React는 모든 컴포넌트를 최초 마운트 직후에 한 번씩 다시 마운트함.**(마운트 -> 마운트 해제 -> 마운트)

"✅ Connecting..." 로그가 두 번 보이면, 컴포넌트가 마운트 해제될 때 연결을 닫지 않고 있다는 실제 문제를 파악하는 데 도움이 됨.

문제를 해결하려면 Effect에서 클린업 함수를 반환.

```javascript
useEffect(() => {
  const connection = createConnection();
  connection.connect();
  return () => {
    connection.disconnect();
  };
}, []);
```

React는 Effect가 다시 실행되기 전에 매번 클린업 함수를 호출하고, 컴포넌트가 마운트 해제(제거)될 때 마지막으로 한 번 더 호출함.

이제 3개의 콘솔 로그를 확인할 수 있음.

"✅ Connecting..."
"❌ Disconnected."
"✅ Connecting..."

이는 개발 단계에서 올바른 동작. 컴포넌트를 다시 마운트하면 React는 멀리 이동했다가 다시 돌아와도 코드가 깨지지 않는지 확인. 연결을 끊었다가 다시 연결하는 것은 정확히 일어나야 하는 일! 클린업을 잘 구현하면, Effect를 한 번 실행하는 것과 실행하고 정리한 후 다시 실행하는 것 사이에 사용자가 체감할 수 있는 차이가 없어야. React가 개발 과정에서 코드에 버그가 있는지 검사하기 때문에 연결/해제 호출 쌍이 추가됨. 이것은 정상적인 현상이니 없애려고 하지 말길...

상용 환경에서는 "✅ Connecting..."이 한 번만 인쇄됨. **컴포넌트를 다시 마운트하는 동작은 클린업이 필요한 Effect를 찾는 것을 돕기 위해 오직 개발 환경에서만 수행됨.** Strict Mode를 해제하면 이같은 개발 동작을 없앨 수 있지만, 가능한 계속 켜두는 것을 추천. 이를 통해 위와 같은 많은 버그를 찾을 수 있기 때문.

## 개발 환경에서 두 번씩 실행되는 Effect를 처리하는 방법은 무엇인가요?

지난 예제에서와 같이 React는 개발 환경에서 버그를 찾기 위해 컴포넌트를 의도적으로 다시 마운트함. 올바른 질문은 “어떻게 하면 Effect를 한 번만 실행할 수 있는가”가 아니라 **“어떻게 다시 마운트한 후에도 Effect가 잘 작동하도록 수정하는가” 임.**

일반적으로 정답은 클린업 함수를 구현하는 것. 클린업 함수는 Effect가 수행 중이던 작업을 중지하거나 취소해야 함. 경험상 (상용 환경에서) 한 번만 실행되는 Effect와 (개발 환경에서의) 설정 → 정리 → 설정 시퀀스를 사용자가 구분할 수 없어야 함.

작성하게 될 대부분의 Effect는 아래의 일반적인 패턴 중 하나에 해당.

## React가 아닌 위젯 제어하기

때론 React로 작성하지 않은 UI 위젯을 추가해야 하는 경우가 있음. 예를 들어, 페이지에 지도 컴포넌트를 추가한다고 가정. 여기에는 setZoomLevel() 메서드가 있으며, 확대/축소 수준을 React 코드의 zoomLevel state 변수와 동기화하고 싶음. Effect는 다음과 비슷할 것.

```javascript
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

이 경우 **클린업이 필요하지 않음.** 개발 환경에서 React는 Effect를 두 번 호출하지만 동일한 값으로 setZoomLevel을 두 번 호출해도 아무 작업도 수행하지 않기 때문에 문제가 되지 않음. **약간 느릴 수는 있지만 상용 환경에서는 불필요하게 다시 마운트되지 않으므로 문제가 되지 않음.**

일부 API는 연속으로 두 번 호출하는 것을 허용하지 않을 수 있음. 예를 들어, 브라우저의 빌트인 요소인 \<dialog>의 showModal 메서드는 두 번 호출하면 에러를 던짐. 클린업 함수를 구현하고 대화 상자를 닫도록 하자.

```javascript
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

개발 중에 Effect는 showModal()을 호출한 다음 즉시 close()를 호출하고, 다시 showModal()을 호출. **이는 상용 환경에서 볼 수 있는 것처럼 showModal()을 한 번 호출하는 것과 체감상 동일**

## 이벤트 구독하기

Effect가 무언가를 구독하는 경우, 클린업 함수는 구독을 취소해야

```javascript
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener("scroll", handleScroll);
  return () => window.removeEventListener("scroll", handleScroll);
}, []);
```

개발 중에 Effect는 addEventListener()를 호출한 다음 즉시 removeEventListener()를 호출함. 그런 다음 동일한 핸들러를 사용하여 다시 addEventListener()를 사용함으로써, 한 번에 하나의 구독만 활성화 되도록 함. 이는 상용 환경에서 addEventListener()를 한 번만 호출하는 것과 체감상 동일.

## 애니메이션 촉발하기

Effect가 무언가를 애니메이션하는 경우 클린업 함수는 애니메이션을 초기값으로 재설정해야

```javascript
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  // 애니메이션 촉발
  return () => {
    node.style.opacity = 0; // Reset to the initial value
    // 초기값으로 재설정
  };
}, []);
```

개발 과정에서 불투명도는 1이었다가, 0이었다가, 다시 1로 설정됨. 이것은 상용 환경에서 1을 한 번만 설정하는 것과 체감상 동일. 만약 트위닝을 지원하는 서드파티 애니메이션 라이브러리를 사용하는 경우라면, 클린업 함수에서 타임라인을 초기 state로 재설정해줘야 함.

## 데이터 페칭하기

Effect가 무언가를 페치하면 클린업 함수는 페치를 중단하거나 그 결과를 무시해야 함.

```javascript
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

이미 발생한 네트워크 요청을 “실행 취소”할 수는 없으므로, 대신 클린업 함수에서 더 이상 관련이 없는 페치가 애플리케이션에 계속 영향을 미치지 않도록 해야. 만약 userId가 'Alice'에서 'Bob'으로 변경되면 클린업은 'Alice' 응답이 'Bob' 이후에 도착하더라도 이를 무시하도록 함.

개발 환경에서는 네트워크 탭에 두 개의 페치가 표시됨. 이는 아무 문제가 없음. 위의 접근 방식을 사용하면 첫 번째 Effect가 즉시 정리되므로, ignore 변수의 복사본이 true로 설정됨. 따라서 추가 요청이 있더라도 if (!ignore) 검사 덕분에 state에 영향을 미치지 않음.

상용 환경에서는 요청이 하나만 있음. 개발 환경에서 두 번째 요청이 귀찮은 경우 가장 좋은 방법은 요청을 중복 제거하고 컴포넌트 간에 응답을 **캐시하는 솔루션을 사용하는 것.**

```javascript
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

이렇게 하면 개발 경험이 향상될 뿐만 아니라 애플리케이션이 더 빠르게 느껴짐. 예를 들어, Back 버튼을 누르는 사용자는 일부 데이터가 캐시되기 때문에 다시 로드될 때까지 기다릴 필요가 없음. 이러한 캐시는 직접 구축할 수도 있고, Effect에서 수동으로 페칭하는 기능을 대체하는 많은 대안 중 하나를 사용할 수도 있음.

- 참고: Effect에서 데이터를 페칭하는 것의 대안은 무엇입니까?
  - Effect 내에 fetch 호출을 작성하는 것은 특히 클라이언트 측에서만 작성된 앱에서 데이터를 페치하는 인기 있는 방법. 그러나 이것은 매우 수동적인 접근 방식이며 상당한 단점이 있음.
  - Effects는 서버에서 실행되지 않음. 즉,초기 서버에서 렌더링되는 HTML에는 데이터가 없는 로딩 state만 포함됨. 클라이언트 컴퓨터는 모든 JavaScript를 다운로드하고 앱을 렌더링하고 나서야 비로소 데이터를 로드해야 한다는 사실을 발견해 냄. 이것은 그다지 효율적이지 않음.
  - Effect에서 직접 페치하면 “네트워크 워터폴”이 만들어지기 쉬움. 상위 컴포넌트를 렌더링하면, 상위 컴포넌트가 일부 데이터를 페치하고, 하위 컴포넌트를 렌더링한 다음, 다시 하위 컴포넌트의 데이터를 페치하기 시작함. 네트워크가 매우 빠르지 않다면, 모든 데이터를 병렬로 페치하는 것보다 훨씬 느림.
  - Effect에서 직접 페치하는 것은 일반적으로 데이터를 미리 로드하거나 캐시하지 않음을 의미. 예를 들어, 컴포넌트가 마운트 해제되었다가 다시 마운트되면, 데이터를 다시 페치할 것.
  - 이것은 전혀 인체공학적이지 않음. race condition과 같은 버그가 발생하지 않는 방식으로 fetch를 작성하기 위해서는 꽤 많은 상용구 코드가 필요함.
  - 이러한 단점들은 React에만 국한된 것이 아님. 어떤 라이브러리를 사용하든 마운트시 데이터를 페치하는 경우 동일하게 적용됨. 라우팅과 마찬가지로 데이터 페칭 역시 제대로 수행하기는 쉽지 않으므로, 다음과 같은 접근 방식을 권장.
  - 프레임워크를 사용하는 경우 빌트인 데이터 페칭 메커니즘을 사용. 최신 React 프레임워크에는 위와 같은 함정이 발생하지 않으면서 효율적인 데이터 페칭 메커니즘이 통합되어 있음.
  - 그렇지 않으면 클라이언트측 캐시를 사용하거나 직접 구축하는 것을 고려. 인기 있는 오픈소스 솔루션으로는 React Query, useSWR, React Router 6.4+. 등이 있음. 자체 솔루션을 구축할 수도 있는데, 이 경우 내부적으로는 Effect를 사용하되, 요청 중복 제거, 응답 캐싱, 네트워크 워터폴 방지(데이터를 미리 로드하거나 라우트에 데이터 요구 사항을 호이스팅해서) 논리를 추가하는게 좋음.
  - 이러한 접근 방식 중 어느 것도 적합하지 않은 경우 계속해서 Effect에서 직접 데이터를 페칭할 수도.

## 분석 보내기

페이지 방문 시 분석 이벤트를 보내는 다음 코드를 살펴보자.

```javascript
useEffect(() => {
  logVisit(url); // Sends a POST request
}, [url]);
```

개발 환경에서는 모든 URL에 대해 logVisit이 두 번 호출되므로 이를 수정하고 싶을 수 있음. **이 코드를 그대로 유지하는 것이 좋음.** 이전 예제와 마찬가지로 **한 번 실행하는 것과 두 번 실행하는 것 사이에 사용자가 볼 수 있는 동작 차이는 없음.** **개발 머신의 로그가 상용 메트릭을 왜곡하는 것을 원하지는 않을 것이니, 실용적인 관점에서 개발 환경에서는 logVisit가 아무 것도 하지 않도록 하자.** 어차피 개발 환경에서는 파일을 저장할 때마다 컴포넌트가 다시 마운트될 것이고, 따라서 추가 방문이 기록될 테니..

상용 환경에서는 중복 방문 로그가 없음.

전송하는 분석 이벤트를 디버깅하려면 앱을 스테이징 환경(상용 모드에서 실행됨)에 배포하거나, Strict Mode 및 개발 전용의 중복 마운트 검사를 일시적으로 해제할 수 있음. Effect 대신 경로 변경 이벤트 핸들러에서 분석을 전송할 수도 있음. 보다 정확한 분석을 위해서는 intersection observers를 활용하면 뷰포트에 어떤 컴포넌트가 있고 얼마나 오래 표시되는지를 추적하는 데 도움이 될 수 있음.

## Effect가 아님: 애플리케이션 초기화하기

일부 로직은 애플리케이션이 시작될 때 한 번만 실행되어야. 이런 로직은 컴포넌트 외부에 넣을 수 있음.

```javascript
if (typeof window !== "undefined") {
  // Check if we're running in the browser.
  // 실행환경이 브라우저인지 여부 확인
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

## Effect가 아님: 제품 구매하기

클린업 함수를 작성하더라도 Effect를 두 번 실행함으로써 체감상 결과가 달라지는 것을 막을 방법이 없는 경우도 있음. 예를 들어, Effect가 제품 구매와 같은 POST 요청을 보낸다고 하자.

```javascript
useEffect(() => {
  // 🔴 Wrong: This Effect fires twice in development, exposing a problem in the code.
  // 🔴 틀렸습니다: 이 Effect는 개발모드에서 두 번 실행되며, 문제를 일으킵니다.
  fetch("/api/buy", { method: "POST" });
}, []);
```

제품을 두 번 사고 싶지는 않을 것. 이 로직을 Effect에 넣지 말아야 하는 이유이기도 함. 사용자가 다른 페이지로 이동한 다음 Back(뒤로가기)를 누르면, Effect가 다시 실행될 것. 사용자는 페이지를 방문할 때 제품을 구매하고 싶지는 않을 것. 구매 버튼을 클릭할 때 구매하길 원할 것

구매는 렌더링으로 인한 것이 아님. 특정 상호 작용으로 인해 발생함. 사용자가 버튼을 누를 때만 실행되어야 함. Effect를 삭제하고 /api/buy 요청을 구매 버튼 이벤트 핸들러로 이동.

사용자 관점에서 페이지를 방문하는 것은 페이지를 방문하여 링크를 클릭하고 뒤로가기 버튼을 누르는 것과 다르지 않아야 함. React는 개발 단계에서 컴포넌트를 다시 한 번 마운트하여 이 원칙을 준수하는지 확인함.

## 한데 모으기

이 플레이그라운드를 통해 실제로 Effect가 어떻게 작동하는지 “느껴볼” 수 있음.

이 예에서는 setTimeout을 사용하여 Effect가 실행되고 3초 뒤에 input 텍스트가 포함된 콘솔 로그를 표시하도록 예약함. 클린업 함수는 보류 중인 타임아웃을 취소. “Mount the component”를 눌러보라.

```javascript
import { useState, useEffect } from "react";

function Playground() {
  const [text, setText] = useState("a");

  useEffect(() => {
    function onTimeout() {
      console.log("⏰ " + text);
    }

    console.log('🔵 Schedule "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancel "' + text + '" log');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        What to log:{" "}
        <input value={text} onChange={(e) => setText(e.target.value)} />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? "Unmount" : "Mount"} the component
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

로그:
🔵 Schedule "a" log
🟡 Cancel "a" log
🔵 Schedule "a" log
⏰ a

왜 이런 로그가 나오는지 알지?

이제 input을 편집하여 abc라고 입력하라. 충분히 빨리 입력하면 Schedule "ab" log와 Cancel "ab" log, Schedule "abc" log가 바로 표시될 것.**React는 항상 다음 렌더링의 Effect 전에 이전 렌더링의 Effect를 정리함.(clean up)** 따라서 입력을 빠르게 입력하더라도 한 번에 최대 한 번만 타임아웃이 예약됨. 입력을 몇 번 편집하고 콘솔을 보면서 Effect가 어떻게 정리되는지 느껴보라.

input에 무언가를 입력한 다음 즉시 “Unmount the component”를 눌러보라. 마운트를 해제하면 마지막 렌더링의 Effect가 어떻게 정리되는지 확인하라. 여기서는 Effect가 실행 기회를 갖기 전에 마지막 타임아웃을 지움.

마지막으로 위의 컴포넌트를 편집하고 타임아웃이 취소되지 않도록 클린업 함수를 주석 처리 해보자. abcde를 빠르게 입력해 보자. 3초 후에 어떤 일이 일어날까? 타임아웃 내의 console.log(text)가 최신 text를 인쇄하고 5개의 abcde 로그를 생성할까?

3초 후, 5개의 abcde가 아닌 일련의 (a, ab, abc, abcd, abcde)가 표시되어야. **각 Effect는 해당 렌더링에서 text 값을 “캡처”함.** text state가 변경되더라도, text = 'ab'인 렌더링의 Effect는 항상 'ab'를 표시. **즉,각 렌더링의 Effect는 서로 분리되어 있음. 이것이 어떻게 작동하는지 궁금하다면 클로저를 읽어 보라.**

- 참고: 각 렌더링에는 고유한 Effect가 있습니다

  - useEffect는 렌더링 출력물에 동작의 일부를 “첨부”하는 것으로 생각할 수 있음. 다음 Effect를 보자.

    ```javascript
    export default function ChatRoom({ roomId }) {
      useEffect(() => {
        const connection = createConnection(roomId);
        connection.connect();
        return () => connection.disconnect();
      }, [roomId]);

      return <h1>Welcome to {roomId}!</h1>;
    }
    ```

  - 사용자가 앱을 탐색할 때 정확히 어떤 일이 발생하는지 살펴보자.
  - 초기렌더링
    - 사용자가 \<ChatRoom roomId="general" />을 방문
    - Effect는 렌더링 출력의 일부이기도 함. 첫 번째 렌더링의 Effect는 다음과 같음.
      ```javascript
      // Effect for the first render (roomId = "general")
      // 첫번째 렌더링시의 JSX (roomId = "general")
      () => {
        const connection = createConnection("general");
        connection.connect();
        return () => connection.disconnect();
      },
        // Dependencies for the first render (roomId = "general")
        // 첫번째 렌더링시의 의존성 (roomId = "general")
        ["general"];
      ```
    - React는 'general' 대화방으로 연결되는 이 Effect를 실행
  - 동일한 의존성을 사용하여 다시 렌더링
    - \<ChatRoom roomId="general" />이 다시 렌더링된다고 가정. JSX 출력은 동일. => 렌더링 출력이 변경되지 않았다고 판단하여 DOM을 업데이트하지 않음.
    - 두 번째 렌더링의 Effect는 다음과 같음.
      ```javascript
      // Effect for the second render (roomId = "general")
      // 두번째 렌더링시의 JSX (roomId = "general")
      () => {
        const connection = createConnection("general");
        connection.connect();
        return () => connection.disconnect();
      },
        // Dependencies for the second render (roomId = "general")
        // 두번째 렌더링시의 의존성 (roomId = "general")
        ["general"];
      ```
    - React는 두 번째 렌더링의 ['general']을 첫 번째 렌더링의 ['general']과 비교. 모든 의존성이 동일하기 때문에 React는 두 번째 렌더링의 Effect를 무시.
  - 다른 의존성으로 다시 렌더링
    - 그런 다음 사용자는 \<ChatRoom roomId="travel" />을 방문.
    - 세 번째 렌더링의 Effect는 다음과 같음.
      ```javascript
      // Effect for the third render (roomId = "travel")
      // 세번째 렌더링시의 JSX (roomId = "travel")
      () => {
        const connection = createConnection("travel");
        connection.connect();
        return () => connection.disconnect();
      },
        // Dependencies for the third render (roomId = "travel")
        // 세번째 렌더링시의 의존성 (roomId = "travel")
        ["travel"];
      ```
    - React는 세 번째 렌더링의 ['travel']을 두 번째 렌더링의 ['general']과 비교. 하나의 의존성이 다름. Object.is('travel', 'general')는 false입니다. Effect는 건너뛸 수 없음.
    - React가 세 번째 렌더링에서 Effect를 적용하려면 먼저 실행된 마지막 Effect를 정리해야(clean up). 두 번째 렌더링의 Effect를 건너뛰었으므로 React는 첫 번째 렌더링의 Effect를 정리. 첫 번째 렌더링까지 스크롤하면 createConnection('general')으로 생성된 연결에서 클린업이 disconnect()를 호출하는 것을 볼 수 있음. 이렇게 하면 'general' 대화방에서 앱의 연결이 끊어짐.
    - 그 후 React는 세 번째 렌더링의 Effect를 실행. 'travel' 대화방으로 연결됨.
  - 마운트 해제
    - 마지막으로 사용자가 다른 곳으로 이동하여 ChatRoom 컴포넌트가 마운트 해제되었다고 가정. React는 마지막 Effect의 클린업 함수를 실행함. 마지막 Effect는 세 번째 렌더링에서 가져온 것. 세 번째 렌더링의 클린업은 createConnection('travel') 연결을 해제. 따라서 'travel' 방에서 앱의 연결이 끊어짐.
  - 개발환경 전용 동작
    - Strict Mode가 켜져 있으면 React는 마운트 후 모든 컴포넌트를 한 번 다시 마운트(**state 및 DOM이 보존됨**). 이를 통해 정리가 필요한 Effect를 찾고 조건 경합과 같은 버그를 조기에 발견할 수 있음. 또한 React는 개발 중에 파일을 저장할 때마다 Effect를 다시 마운트. 이러한 동작은 모두 개발환경에서만 이뤄짐.
