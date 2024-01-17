## Reacting to Input with State

### 들어가면서

리액트는 UI를 조작하는 **선언적인 방법** 을 제공한다. UI가 어떻게 생겨먹었고 어떤 상태가 있고 이런 상태일때는 어떻게 나타내는지등을 선언적으로 나타낸다. 그리고 그러한 상태는 인풋에 따라 바뀔 수 있다. 이것은 디자이너가 UI에 관해 생각하는 방식과 유사하다.

### 선언적 UI와 명령형 UI의 차이

명령형 UI: 어떤 UI를 구현하려면 하나부터 열까지, 모든 조작을 해야함. 즉, 컴퓨터에게 UI가 언제 어떻게 변경되고, 어떨땐 어떤 UI가 보여야하는지를 일일히 지시를 내려야함. -> 명령형 프로그래밍이라고 불리는 이유.

이러한 명령형 프로그래밍은, 간단한 것을 만들때는 괜찮지만 복잡한 어플리케이션을 만들때는 예상치 못한 버그를 일으킬 수 있고 관리도 힘듦. 그리고 변경에 취약함. 리액트의 선언적 프로그래밍 방식은 이러한 문제를 해결하기 위해 등장한 것.

선언적 UI: 일일히 명령 내리지 않아도 됨. 그저, **뭐가 보이기를 원하는 지를 선언**하면 끝임. 나머지는 다 리액트가 알아서 해줌.

### 선언적으로 UI에 대해 생각하는 법

1. 컴포넌트의 다양한 시각적 상태 식별

   - 사용자 입장에서, UI가 가질 수 있는 다양한 상태를 시각화 하라.

     예시)

     > Empty: Form has a disabled “Submit” button.

     > Typing: Form has an enabled “Submit” button.

     > Submitting: Form is completely disabled. Spinner is shown.

     > Success: “Thank you” message is shown instead of a form.

     > Error: Same as Typing state, but with an extra error message.

   - 그리고 이러한 상태를 가지고 원한다면 디자이너처럼 상태에 따라 UI가 어떻게 보일지 모킹할 수 있다. 본격적으로 로직을 작성하기 전에
     ```javascript
     export default function Form({
       // Try 'submitting', 'error', 'success':
       status = "submitting",
     }) {
       if (status === "success") {
         return <h1>That's right!</h1>;
       }
       return (
         <>
           <h2>City quiz</h2>
           <p>
             In which city is there a billboard that turns air into drinkable
             water?
           </p>
           <form>
             <textarea disabled={status === "submitting"} />
             <br />
             <button disabled={status === "empty" || status === "submitting"}>
               Submit
             </button>
             {status === "error" && (
               <p className="Error">
                 Good guess but a wrong answer. Try again!
               </p>
             )}
           </form>
         </>
       );
     }
     ```
   - 팁
     이러한 시각적 상태를 많이 가지고 있다면, 가질 수 있는 상태들을 map 함수로 돌면서 한 페이지에 한번에 나타내어 시각적으로 볼 수 있도록 하기도 한다. 이런 페이지를 “living styleguides” 나 “storybooks”. 라고 부른다.

2. 이러한 상태 변화를 유발하는 요소 파악

   - 상태 변화를 유발하는 input은 두 가지 종류. 휴먼 인풋 / 컴퓨터 인풋
   - 휴먼 인풋은 사람에 의해 발생하는 것.
   - 컴퓨터 인풋은 이미지 로딩, 타임아웃, 서버로부터 도착한 응답 등
   - 상태가 어떤 인풋으로 인해 다른 상태로 변하는지 flow를 그리면, 버그를 줄일 수 있다.

3. useState를 사용하여 메모리에서 상태를 표현

   - 상태를 useState를 통해 표현하면 됨.
   - 정말 필수적이라고 생각되는 상태부터 고려.
   - 상태가 너무 많아도 복잡해지고 버그 발생 확률이 높아짐.
   - 그런데 뭘 넣고 안 넣고를 잘 모르겠다면, 일단 다 때려박아라. 그리고 리팩토링 하면 된다. 다음의 과정을 거쳐서.

4. 필수적이지 않은 상태 변수를 제거

   - 이러한 리팩토링은 컴포넌트를 이해하기 쉽게 만들고, 중복을 줄이며, 의도치 않은 결과를 방지함.
   - 특히나, 상태의 구조를 잘못 정해서 그 어떤 상태도 사용자가 보기를 바랐던 유효한 UI를 나타내지 않는다면 안되겠지..
   - 이러한 제거 작업을 하기 위해서는 세 가지 질문을 해야함.

     1. 이 상태가 역설을 야기하는가?
        - 동시에 발생할 수 없는 상태들이 있다면 이들을 'status'라는 하나의 상태로 묶어라
     2. 다른 상태를 통해 동일한 정보를 가져올 수 있는가?
     3. 다른 상태의 역을 통해 동일한 정보를 가져올 수 있는가?

   - 이렇게 리팩토링을 해도 모순이 발생할 수 있음. 이럴 땐 리듀서를 사용하는 것을 고려해봐라. Reducers let you unify multiple state variables into a single object and consolidate all the related logic!

5. 이벤트 핸들러를 연결해 상태 설정
   - 상태를 업데이트하는 이벤트 핸들러를 만들어라.
