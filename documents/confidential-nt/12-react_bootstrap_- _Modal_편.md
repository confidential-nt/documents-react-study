### 들어가면서

Modal은 compound pattern, ContextAPI 사용법, createPortal, 여러가지 꽤 복잡한 UI 관련 로직 등을 공부하기 좋은 컴포넌트다.

### 공부한 내용

[공식문서 주소](https://react-bootstrap.github.io/docs/components/modal)

#### 공식 문서로부터 알 수 있었던 것

1. 모달의 구성: 헤더, 바디, 액션의 집합이 있는 푸터(ex: 취소 버튼)
2. Modal 컴포넌트로 모든 걸 감쌈. 그리고 이걸로 모달 숨김/보여줌 조절.(show, onHide 프롭…) closeButton 누르면 아마 자동으로 onHide가 호출될듯.
3. The `<Modal/>` Component comes with a few convenient "sub components": `<Modal.Header/>`, `<Modal.Title/>`, `<Modal.Body/>`, and `<Modal.Footer/>`, which you can use to build the Modal content. → compound component
4. header에 closeButton이라는 프롭으로 헤더에 클로즈 버튼 자동 추가 가능 + 헤더에 모달 타이틀 넣는듯. 바디에는 그냥 본문 내용 따 때려박는거고..
5. 백드롭을 모달에 prop으로 넣어줄 수 있는데 얘가 static이면 modal will not close when clicking outside it → 즉, 백드롭을 클릭해도 안닫힌다. 또한, 이때만의 특유의 애니메이션이 호출됨. 그냥 backdrop으로 주면 닫힘. (얘가 디폴트인듯) (”static”, true, false 가질 수 있고 false면 백드롭 자체가 없음)
6. keyboard prop이 false면 esc로 닫을 수 없는듯.
7. animation 없앨려면 false로 설정. true가 디폴트인듯
8. You can vertically center a modal by passing the `centered` prop.
9. Modal에 주는 aria-labelledby prop과 Modal.title의 id와의 관계

   **`aria-labelledby="contained-modal-title-vcenter"`**와 **`id="contained-modal-title-vcenter"`**는 웹접근성을 향상시키기 위해 사용되며, 두 속성 간에는 직접적인 연결이 있다.

   - **`id="contained-modal-title-vcenter"`**: 이 속성은 모달의 타이틀에 고유한 식별자를 부여. **`id`** 속성 값은 페이지 내에서 고유해야 하며, 이 값을 통해 다른 HTML 요소나 자바스크립트에서 해당 요소를 구분하고 참조할 수 있음.
   - **`aria-labelledby="contained-modal-title-vcenter"`**: 이 속성은 모달 컴포넌트에 사용되어, 스크린 리더와 같은 보조 기술이 모달의 내용을 사용자에게 제시할 때 어떤 요소가 모달의 라벨(제목)로 사용되어야 하는지를 명시. 여기서 **`aria-labelledby`**가 가리키는 **`id`** 값을 가진 요소의 텍스트 콘텐츠가 모달의 라벨로 사용됨. 즉, 스크린 리더가 모달을 읽을 때, 이 **`id`**를 가진 요소의 텍스트를 모달의 이름으로 제공하여 사용자가 모달의 맥락을 이해할 수 있도록 도움.

10. aria-describedby

    접근성을 향상시키는 데 사용되며, 특정 HTML 요소가 설명하는 다른 요소들을 식별하는 데 사용됨. 즉, 요소에 대한 추가적인 설명이 필요할 때 사용됨. 예를 들어, 버튼이나 입력 필드 옆에 도움말 텍스트가 제공되는 경우, 이 도움말 텍스트를 해당 요소에 연결하여 스크린 리더 사용자가 필드를 탐색할 때 설명을 들을 수 있게함.

11. 모달의 포커싱
    따로 포커싱 속성 지정 안해줘도 탭키 누르면 모달 내부 버튼, 폼요소가 있다면 폼 요소들을 포커싱해줌.
    다만 모달을 켰을 때 맨 처음에 여기가 포커싱 되었으면 좋겠다 → 해당 요소에 autoFocus 지정.

12. You can specify a Bootstrap large or small modal by using the `size` prop. (sm, lg…)

13. fullscreen prop을 사용하여 모달을 전체화면으로 만들 수 있습니다. 중단점을 지정하면 중단점 크기 이하에서만 모달이 전체 화면으로 설정됩니다. →
    가능한 값:
    const values = [true, 'sm-down', 'md-down', 'lg-down', 'xl-down', 'xxl-down'];
    첫번째에서는 화면이 많이 작아야 모달이 풀스크린으로 꽉차고….갈수록 화면이 클때도 풀스크린인
14. Modal과 ModalDialog의 관계: Modal의 가장 큰 틀(형태)로서 ModalDialog를 쓰는 것인듯. 다른 거 쓸 수도 있음. Modal의 dialogAs 프롭으로..(A Component type that provides the modal content Markup. This is a useful prop when you want to use your own styles and markup to create a custom modal component.)

#### 코드로부터 알 수 있었던 것

1. restart/ui 및 restart/hooks 사용 중
   - 전자는 headless framework / 후자는 general하게 사용가능한 Hooks들의 집합.
   - headless 함이란? 기능(로직)은 있지만 마크업, 스타일은 없는거. 사용자가 알아서 정의해야함. 반대 개념이 우리가 흔히들 아는 것들. MUI나 이런거..
2. dom-helpers 사용 중 → 돔을 더 쉽게 조작할 수 있게 하는 함수의 집합인듯
3. restart/ui의 Modal을 기반으로 하고 있음.
4. ModalManager라는 것을 사용하여 복잡한 UI 관리 로직을 다루고 있으며, 다중 모달(모달 여러 개)에 관한 로직 또한 다루고 있음. 다만, 부트스트랩은 다중 모달은 지원 안하며, 사용하고 싶으면 restart/ui를 사용해야함. 싱글톤임.
5. ModalProps interface
   - 특정 props를 제거하고 동시에 확장하는 방식에 주목.
   ```javascript
   export interface ModalProps
     extends Omit<
       BaseModalProps,
       | "role"
       | "renderBackdrop"
       | "renderDialog"
       | "transition"
       | "backdropTransition"
       | "children"
     > {
     size?: "sm" | "lg" | "xl";
     fullscreen?:
       | true
       | string
       | "sm-down"
       | "md-down"
       | "lg-down"
       | "xl-down"
       | "xxl-down";
     bsPrefix?: string;
     centered?: boolean;
     backdropClassName?: string;
     animation?: boolean;
     dialogClassName?: string;
     contentClassName?: string;
     dialogAs?: React.ElementType;
     scrollable?: boolean;
     [other: string]: any;
   }
   ```
6. 역시 forwardRef 사용중
7. react-bootstrap의 Modal을 이루는 본격적인 코드들 -> 매우 복잡

   1. 관리하는 상태들

      ```jsx
      const [modalStyle, setStyle] = useState({});
      const [animateStaticModal, setAnimateStaticModal] = useState(false);
      const waitingForMouseUpRef = useRef(false);
      const ignoreBackdropClickRef = useRef(false);
      const removeStaticModalAnimationRef = useRef<(() => void) | null>(null);
      const [modal, setModalRef] = useCallbackRef<ModalInstance>();
      const mergedRef = useMergedRefs(ref, setModalRef);
      const handleHide = useEventCallback(onHide);
      const isRTL = useIsRTL();
      ```

   - modalStyle, setStyle → 현재 컨테이너, 모달의 flowing 상태에 따라 padding 조절해서 적절하게 보이도록 하는 UI 조절용 상태.
   - animateStaticModal, setAnimateStaticModal → 모달의 애니매이션 스타일 결정하는 애. backdrop이 static일때의 애니메이션.
   - waitingForMouseUpRef → handleDialogMouseDown 함수 안에서 waitingForMouseUpRef.current = true; 가 됨. 얘는 렌더링에 영향 줄 필요 없으니 ref로 만든듯
     - 얘의 역할:
     ```tsx
     // We prevent the modal from closing during a drag by detecting where the
     // click originates from. If it starts in the modal and then ends outside
     // don't close.
     ```
     즉, 모달 안에서 바깥(백드롭)으로 드래그(클릭) 하면 모달 안꺼지게 하겠다는 것.
   - ignoreBackdropClickRef →
     모달 안에서 클릭을 하게 되면 handleDialogMouseDown에서 일단 waitingForMouseUpRef.current가 true로 바뀜. 그 다음 handleMouseUp이 오출되는데
     ```jsx
     const handleMouseUp = (e) => {
       if (waitingForMouseUpRef.current && modal && e.target === modal.dialog) {
         ignoreBackdropClickRef.current = true;
       }
       waitingForMouseUpRef.current = false;
     };
     ```
     이런 조건이 걸려있음. 안에서 클릭했으면 waitingForMouseUpRef.current가 트루이므로 위의 조건 절이 실행되고 아래 절이 실행될 것.
     ignoreBackdropClickRef.current = true;가 됨.
     그 다음 handleClick 호출 되겠지.
     ```jsx
     const handleClick = (e) => {
       if (backdrop === "static") {
         handleStaticBackdropClick(e);
         return;
       }

       if (ignoreBackdropClickRef.current || e.target !== e.currentTarget) {
         ignoreBackdropClickRef.current = false;
         return;
       }

       onHide?.();
     };
     ```
     ignoreBackdropClickRef.current = true면 모달이 안꺼지게 되는 것. waitingForMouseUpRef로 모달 안에서 클릭했는지 안했는지 감지 하는 이유. 이에 비해 모달 바깥에서 백드롭을 클릭했으면 onHide가 호출되겠지.
   - removeStaticModalAnimationRef → 그냥 모달 꺼질 때? 언마운트될때? 어떤 동작을 할 지 지정하는 것 같음.
   - 이건 솔직히 뭔지 잘 모르겠는데...
     ```
      const [modal, setModalRef] = useCallbackRef<ModalInstance>();
      const mergedRef = useMergedRefs(ref, setModalRef);
     ```
     - modal은 말그대로 모달 인스턴스. 로직 작성할 때 필요함.
     - mergedRef를 아래 BaseModal에서 ref로 사용하고 있음. 아마, props로 들어온 ref + setModalRef를 통해 modal에 BaseModal 할당하고 있지 않을까? 싶음

   2. context

      ```jsx
      const modalContext = useMemo(
        () => ({
          onHide: handleHide,
        }),
        [handleHide]
      );
      ```

      맨 아래 BaseModal을 감싸고 있는 context provider에 value로 넣어주는 애.
      이걸 보고 의문이 들어 실험해보니까 Modal 컴포넌트는 제어로 써야하는 듯? show, onHide 안넣어주니까 안꺼짐.

   3. getModalManager
      - modal Manager의 역할 : 기본적으로는 body에 모달을 추가/삭제 하고 모달의 사이즈를 조절하는 애. 싱글톤으로 하나만 생성되는듯. 즉, 최초로 생성한 이후, 얘를 호출하면 계속 같은 객체가 반환될듯.
      - 로직에서도 얘를 다양하게 활용 중 + BaseModal의 manager 프롭에서도 얘가 호출하는 객체를 받고 있음.
   4. updateDialogStyle
      - 다이얼로그의 스타일(반응형 너비 조정) 조정 하는 역할.
      - 위에서 언급했지만, setStyle 호출 → style state는 다이얼로그의 style에 들어감
   5. handleWindowResize
      - updateDialogStyle 호출
   6. useWillUnmount
      - unmount 됐을 때에 해야하는 것들 지정
   7. handleStaticModalAnimation
      - modal static animation 켜지게 하는 거
      - 모달이 꺼질 때 호출됨.
   8. handleStaticBackdropClick
      - 백드롭이 static인데 이걸 클릭했을 때 해야하는 행동 정의
      - handleStaticModalAnimation 호출.
   9. handleEscapeKeyDown
      - esc 키 눌렀을 때의 행동 정의
      - keyboard prop이 true면 onEscapeKeyDown 호출
      - 아니라면 backdrop이 static일때 호출되는 애니메이션 같이 사용.
      ```
       // Call preventDefault to stop modal from closing in @restart/ui.
       e.preventDefault();

         if (backdrop === 'static') {
                  // Play static modal animation.
                  handleStaticModalAnimation();
      }
      ```
   10. 다음의 함수들

   ```jsx
    const handleEnter = (node, isAppearing) => {
           if (node) {
             updateDialogStyle(node);
           }

           onEnter?.(node, isAppearing);
         };

         const handleExit = (node) => {
           removeStaticModalAnimationRef.current?.();
           onExit?.(node);
         };

         const handleEntering = (node, isAppearing) => {
           onEntering?.(node, isAppearing);

           // FIXME: This should work even when animation is disabled.
           addEventListener(window as any, 'resize', handleWindowResize);
         };

         const handleExited = (node) => {
           if (node) node.style.display = ''; // RHL removes it sometimes
           onExited?.(node);

           // FIXME: This should work even when animation is disabled.
           removeEventListener(window as any, 'resize', handleWindowResize);
         };
   ```

   - 모달이 켜질때, 꺼질 때의 동작 정의하는 듯. (초기화, 혹은 뒷정리 작업)

     > 공식문서:
     >
     > | onEnter    | func |     | Callback fired before the Modal transitions in            |
     > | ---------- | ---- | --- | --------------------------------------------------------- |
     > | onEntering | func |     | Callback fired as the Modal begins to transition in       |
     > | onEntered  | func |     | Callback fired after the Modal finishes transitioning in  |
     > | onExit     | func |     | Callback fired right before the Modal transitions out     |
     > | onExiting  | func |     | Callback fired as the Modal begins to transition out      |
     > | onExited   | func |     | Callback fired after the Modal finishes transitioning out |

   11. 백드롭 렌더하는 함수

   ```jsx
   const renderBackdrop = useCallback(
     (backdropProps) => (
       <div
         {...backdropProps}
         className={classNames(
           `${bsPrefix}-backdrop`,
           backdropClassName,
           !animation && "show"
         )}
       />
     ),
     [animation, backdropClassName, bsPrefix]
   );
   ```

   - 얘는 밑에 BaseModal 컴포넌트에 prop으로 넣어주는 것.

   - 백드롭은 기본적으로 div고(따로 컴포넌트 존재 x), backdropClassName으로 커스텀 가능한듯?

   12. dialog 렌더하는 함수

   ```jsx
   const renderDialog = (dialogProps) => (
     <div
       role="dialog"
       {...dialogProps}
       style={baseModalStyle}
       className={classNames(
         className,
         bsPrefix,
         animateStaticModal && `${bsPrefix}-static`,
         !animation && "show"
       )}
       onClick={backdrop ? handleClick : undefined}
       onMouseUp={handleMouseUp}
       data-bs-theme={dataBsTheme}
       aria-label={ariaLabel}
       aria-labelledby={ariaLabelledby}
       aria-describedby={ariaDescribedby}
     >
       {/*
           // @ts-ignore */}
       <Dialog
         {...props}
         onMouseDown={handleDialogMouseDown}
         className={dialogClassName}
         contentClassName={contentClassName}
       >
         {children}
       </Dialog>
     </div>
   );
   ```

   - 얘가 사실상 모달 본체

   - dialogAs로 다른 다이얼로그 쓸 수 있는데 디폴트는 부트스트랩의 모달 다이얼로그.
   - 이 다이얼로그를 Dialog로 사용하고 있음. className을 이용해(dialogClassName,contentClassName…) 여러가지 스타일 커스텀 할 수 있게 해놨고 animate static이냐 아니냐에 따라 클래스 다르게 주고 있고...

   - onClick={backdrop ? handleClick : undefined} → backdrop이 static이나 true면 handleClick 호출.

   - 여기다가 children 박아두고 있음.

   - dialog 컴포넌트를 div태그로 또 감싸고 있음. dialog 컴포넌트도 사실상 div. 겉에 있는 div에 aria-… 박아둔거 주목.

   13. return 문

   ```jsx
   return (
     <ModalContext.Provider value={modalContext}>
       <BaseModal
         show={show}
         ref={mergedRef}
         backdrop={backdrop}
         container={container}
         keyboard // Always set true - see handleEscapeKeyDown
         autoFocus={autoFocus}
         enforceFocus={enforceFocus}
         restoreFocus={restoreFocus}
         restoreFocusOptions={restoreFocusOptions}
         onEscapeKeyDown={handleEscapeKeyDown}
         onShow={onShow}
         onHide={onHide}
         onEnter={handleEnter}
         onEntering={handleEntering}
         onEntered={onEntered}
         onExit={handleExit}
         onExiting={onExiting}
         onExited={handleExited}
         manager={getModalManager()}
         transition={animation ? DialogTransition : undefined}
         backdropTransition={animation ? BackdropTransition : undefined}
         renderBackdrop={renderBackdrop}
         renderDialog={renderDialog}
       />
     </ModalContext.Provider>
   );
   ```

   14. assign

   ```jsx
   export default Object.assign(Modal, {
     Body: ModalBody,
     Header: ModalHeader,
     Title: ModalTitle,
     Footer: ModalFooter,
     Dialog: ModalDialog,
     TRANSITION_DURATION: 300,
     BACKDROP_TRANSITION_DURATION: 150,
   });
   ```

   - TRANSITION_DURATION, BACKDROP_TRANSITION_DURATION도 export 저렇게 하고 있는 것에 주목.

8. @restart/ui의 BaseModal을 파보면서 알게된 것.
   1. useImperativeHandle 사용
      - 요약하자면, ref를 부모 컴포넌트에서 가져오고자 할 때, 자식 컴포넌트에서 돔 노드를 그대로 제공하는 것이 아니라 사용자 정의 객체를 리턴하는 것. 노출하고자 하는 메서드만 노출할 때 유용.
   2. import 순서 주목 (모든 파일에서 이런지는 모르겠음.) → 리액트 관련 - 기타 외부 라이브러리 - 직접 만든 것.
   3. 주석 경향: props에 대한 설명 + 중요 이슈 / 유의사항에 관한 설명 + 다음에 고쳐야할 것 / 해야할 것/ 기타 사용자가 이해해야할, 이해하기 힘들 수 있는 로직에 대한 설명 / 들에 대해 주로 이야기 하고 있음.
      함수 프롭에 대한 주석 주목 : 예시를 상세하게 들어주고 있음.
      ````
      /**
      * A function that returns the dialog component. Useful for custom
      * rendering. **Note:** the component should make sure to apply the provided ref.
      *
      * ```js static
      * renderDialog={props => <MyDialog {...props} />}
      * ```
      */
      renderDialog?: (props: RenderModalDialogProps) => React.ReactNode;
      ````
   4. cloneElement 사용 (레거시)
      - 얘가 뭐하는 거냐면, 말 그대로 엘리먼트를 복사해서 리턴하는 것. 원본 변경 x.
      - 근데 안티패턴. 데이터 흐름 추적 어렵대. 특히 props으로 넘겨준 값이 변경 되거나 그러면, 그 값의 출처를 명확하게 추적하기 어렵다나 뭐라나.. 대안
      - render prop, context, custom hooks (유용한 내용 많음)
   5. createPortal 사용
      - 어떤 엘리먼트를 기존 부모에서 탈출시켜 다른 부모의 자식처럼 렌더링되게 하는 방법. (단, dom 만 보면 이렇게 렌더링 되는데 리액트 돔 구조에서는 원래 구조가 그대로 유지됨 → 이벤트 이런거 그대로 유지됨.)
      - 모달 같은 거 만들 때 특히 유용.
      - 다른 사용 예제도 많음. (비 리액트 서버 마크업과 함께 렌더링하기, **비 React DOM 노드 렌더링하기…)** 매우 유용하니 참고.
      - 참고할만한 다른 문서 발견: https://www.w3.org/WAI/ARIA/apg/patterns/ → 어떻게 웹 접근성 지키면서 모달 같은 거 만드는지…(특히 이런 포탈 사용하면 키보드 포커스에 유의해야함.)
   6. export default로 컴포넌트 export해도, 타입 정도는 추가로 export 해도 되는 느낌..
   7. 끝에 ! 붙이는 문법
   ```javascript
   setDialogRef: useCallback((ref: HTMLElement | null) => {
    modal.current.dialog = ref!;
   }, []),
   ```
   ref!의 의미: TypeScript에서 "non-null assertion operator"라고 불리며, 개발자가 해당 변수가 null이나 undefined가 아니라는 것을 확신할 때 사용됨. 이 연산자는 TypeScript 컴파일러에게 특정 값이 절대 null이나 undefined가 아님을 알려주어, 그에 따른 타입 검사 오류를 방지. 주의해서 사용해야.

### 구현 연습

Modal.tsx

```javascript
import {
  ElementType,
  ReactElement,
  forwardRef,
  useCallback,
  useEffect,
} from "react";
import { createPortal } from "react-dom";
import ModalDialog from "../ModalDialog/ModalDialog";
import ModalContext from "../ModalContext/ModalContext";
import ModalBody from "../ModalBody/ModalBody";
import ModalFooter from "../ModalFooter/ModalFooter";
import ModalHeader from "../ModalHeader/ModalHeader";
import ModalTitle from "../ModalTitle/ModalTitle";
import "./Modal.css";
import classNames from "classnames";

export interface ModalProps {
  show?: boolean;
  backdrop?: boolean;
  onHide?: () => void;
  onShow?: () => void;
  keyboard?: boolean;
  container?: Element | DocumentFragment;
  dialogAs?: ElementType;
  backdropClassname?: string;
  dialogClassname?: string;
  centered?: boolean;
  animation?: boolean;
  fullscreen?: true | "md-down" | "lg-down" | "xl-down";
  size?: "sm" | "lg" | "xl";
  "aria-labelledby"?: string;
  "aria-describedby"?: string;
  "aria-label"?: string;
  children?: ReactElement;
  [other: string]: unknown;
}

const ModalImpl = forwardRef<HTMLDivElement, ModalProps>(
  (allProps: ModalProps, ref) => {
    const {
      container = document.body,
      show = false,
      onHide,
      keyboard = true,
      dialogAs: Dialog = ModalDialog,
      backdrop = true,
      backdropClassname = "",
      dialogClassname = "",
      centered = false,
      animation = true,
      fullscreen,
      size,
      "aria-labelledby": ariaLabelledby,
      "aria-describedby": ariaDescribedby,
      "aria-label": ariaLabel,
      children,
    } = allProps;

    const handleHide = useCallback(() => {
      onHide?.();
    }, [onHide]);

    const handleEscapeKeyDown = useCallback(
      (event: KeyboardEvent) => {
        if (keyboard && event.key === "Escape") {
          handleHide();
        }
      },
      [keyboard, handleHide]
    );

    const handleBackdropClick = () => {
      onHide?.();
    };

    useEffect(() => {
      if (show) {
        document.body.style.overflow = "hidden";
        window.addEventListener("keydown", handleEscapeKeyDown);
      }

      return () => {
        document.body.style.overflow = "";
        window.removeEventListener("keydown", handleEscapeKeyDown);
      };
    }, [show, handleEscapeKeyDown]);

    const dialog = (
      <Dialog
        centered={centered}
        animation={animation}
        fullscreen={fullscreen}
        size={size}
        ref={ref}
        className={classNames(dialogClassname)}
        aria-label={ariaLabel}
        aria-labelledby={ariaLabelledby}
        aria-describedby={ariaDescribedby}
      >
        {children}
      </Dialog>
    );

    const backdropImpl = backdrop ? (
      <div
        onClick={handleBackdropClick}
        className={classNames("backdrop", backdropClassname)}
      ></div>
    ) : null;

    return (
      <ModalContext.Provider
        value={{
          onHide: handleHide,
        }}
      >
        {show
          ? createPortal(
              <div className={"modalRoot"}>
                {dialog}
                {backdropImpl}
              </div>,
              container
            )
          : null}
      </ModalContext.Provider>
    );
  }
);

const Modal = Object.assign(ModalImpl, {
  Body: ModalBody,
  Dialog: ModalDialog,
  Footer: ModalFooter,
  Header: ModalHeader,
  Title: ModalTitle,
});

export default Modal;
```

ModalDialog.tsx

```javascript
import classNames from "classnames";
import { HTMLAttributes, forwardRef } from "react";
import "./ModalDialog.css";

export interface ModalDialogProps extends HTMLAttributes<HTMLDivElement> {
  centered?: boolean;
  animation?: boolean;
  fullscreen?: true | "md-down" | "lg-down" | "xl-down";
  size?: "sm" | "lg" | "xl";
}

const ModalDialog = forwardRef<HTMLDivElement, ModalDialogProps>(
  (allProps: ModalDialogProps, ref) => {
    const {
      centered = false,
      animation = true,
      fullscreen,
      size,
      className,
      children,
      ...props
    } = allProps;
    return (
      <div
        role="dialog"
        ref={ref}
        className={classNames("modalDialog", className, {
          centered,
          animation,
          [`fs-${fullscreen}`]: fullscreen,
          [`${size}`]: size,
        })}
        {...props}
      >
        {children}
      </div>
    );
  }
);

export default ModalDialog;
```

ModalHeader.tsx

```javascript
import classNames from "classnames";
import { HTMLAttributes, forwardRef, useContext } from "react";
import CloseButton from "../CloseButton/CloseButton";
import ModalContext from "../ModalContext/ModalContext";

export interface ModalHeaderProps extends HTMLAttributes<HTMLDivElement> {
  closeButton?: boolean;
}

const ModalHeader = forwardRef<HTMLDivElement, ModalHeaderProps>(
  (allProps: ModalHeaderProps, ref) => {
    const { className, closeButton = false, children, ...props } = allProps;
    const { onHide } = useContext(ModalContext);
    return (
      <header ref={ref} className={classNames(className)} {...props}>
        {closeButton ? <CloseButton onClick={onHide} /> : null}
        {children}
      </header>
    );
  }
);

export default ModalHeader;
```

ModalTitle.tsx

```javascript
import classNames from "classnames";
import { HTMLAttributes, forwardRef } from "react";

export interface ModalTitleProps extends HTMLAttributes<HTMLHeadingElement> {}

const ModalTitle = forwardRef<HTMLHeadingElement, ModalTitleProps>(
  (allProps: ModalTitleProps, ref) => {
    const { className, children, ...props } = allProps;
    return (
      <h4 ref={ref} className={classNames(className)} {...props}>
        {children}
      </h4>
    );
  }
);

export default ModalTitle;
```

ModalBody.tsx

```javascript
import classNames from "classnames";
import { HTMLAttributes, forwardRef } from "react";

export interface ModalBodyProps extends HTMLAttributes<HTMLDivElement> {}

const ModalBody = forwardRef<HTMLDivElement, ModalBodyProps>(
  (allProps: ModalBodyProps, ref) => {
    const { className, children, ...props } = allProps;
    return (
      <div ref={ref} className={classNames(className)} {...props}>
        {children}
      </div>
    );
  }
);

export default ModalBody;
```

ModalFooter.tsx

```javascript
import classNames from "classnames";
import { HTMLAttributes, forwardRef } from "react";

export interface ModalFooterProps extends HTMLAttributes<HTMLDivElement> {}

const ModalFooter = forwardRef<HTMLDivElement, ModalFooterProps>(
  (allProps: ModalFooterProps, ref) => {
    const { className, children, ...props } = allProps;
    return (
      <footer ref={ref} className={classNames(className)} {...props}>
        {children}
      </footer>
    );
  }
);

export default ModalFooter;
```

ModalContext.tsx

```javascript
import { createContext } from "react";

interface ModalContextType {
  onHide: () => void;
}

const ModalContext =
  createContext <
  ModalContextType >
  {
    onHide() {},
  };

export default ModalContext;
```

App.tsx

```javascript
import { useState } from "react";
import "./App.css";

import Modal from "./components/Modal/Modal";

function App() {
  const [show, setShow] = useState < boolean > false;

  return (
    <main>
      <button onClick={() => setShow(true)}>모달 보기</button>
      <p>
        Lorem ipsum dolor sit amet consectetur adipisicing elit. Cumque,
        cupiditate error esse excepturi totam labore expedita repellendus,
        dolorum quasi velit adipisci similique, quidem quis? Quidem assumenda
        odit quae pariatur sed.
      </p>
      <Modal
        show={show}
        onHide={() => setShow(false)}
        centered
        size={"xl"}
        animation={false}
        aria-labelledby="modal"
      >
        <>
          <Modal.Header closeButton>
            <Modal.Title id="modal">이것은 모달이다.</Modal.Title>
          </Modal.Header>
          <Modal.Body>
            <h1>모달 본문.</h1>
            <input type="text" placeholder="내용을 입력하세요." autoFocus />
          </Modal.Body>
          <Modal.Footer>
            <button onClick={() => setShow(false)}>확인</button>
            <button onClick={() => setShow(false)}>취소</button>
          </Modal.Footer>
        </>
      </Modal>
    </main>
  );
}

export default App;
```

화면
![](https://velog.velcdn.com/images/youyoy689/post/aade1064-d503-4965-a433-e8b09acee62a/image.png)
~~스타일링이 귀찮아서 스킵했습니다...~~

- BaseModalProps → html을 extends한게 아님. → 직접 구현 때 interface 어떻게 할 것이냐? 모달 근본은 div로 이루어져있으므로 div를 extends하는 건 어떨까? -> 그러나 사용자 입장에서 aria- 작성했을 때 너무 쓸데없는 것 까지 다 뜸. → ModalProps를 div를 extends하지 않는 걸로 결정. → 없어도 괜찮을듯. 어차피 Modal은 실질적으로 UI에 관여하는 게 없음. 그냥 내부에서 필요로 하는 것들만 받아오면 됨.
- **ReactElement vs ReactNode vs JSX.Element**
  - https://velog.io/@hanei100/ReactElement-vs-ReactNode-vs-JSX.Element#1-reactelement
- React.ReactElement vs React.ElementType

  - React.ReactElement는 React 요소 자체를, React.ElementType는 요소를 생성할 수 있는 컴포넌트의 타입을 참조.

- role=”heading” vs heading tags

  - https://www.w3.org/WAI/WCAG21/Techniques/aria/ARIA12
    결론부터 말하자면 특별한 이유가 없지 않은 이상, heading 태그 쓰는 게 나은 것 같음.
  - 특별한 이유1. 레거시 사이트 개선할때
  - 특별한 이유2. h6까지 있으므로 h7을 나타내고 싶을 떄 (aria-level)

- 위의 문서를 이유로 AlertHeading과는 달리 ModalTitle은 그냥 h4 태그로 구현. 왜 h4? 원본이 h4로 스타일링 되어있으니까..
- 그렇다면 ModalHeader도 div가 아니라 header 태그로 만드는 게 나아보임. 단 타입 중에는 HTMLHeaderElement가 없으므로 그대로 HTMLDivElement 쓰는 걸로..
- 모달의 바디는 무슨 태그로 해야할지 애매해서 일단 div
- ModalDialog에 ref 박아놓음 -> restart/ui도 그렇게 했더라..아마?
- createPortal 사용
- 사용성을 위한다면 웬만한 prop은 옵셔널로 설정해주고, 구현부에서 default를 설정해주는 게 나은 듯함.
- ModalContext → closeButton이 onHide를 필요로 함 → 내려주기 싫으니까 context 만들자.
- 모달 열릴 때 document.body에 overflow hidden 적용해주고 싶음 → useEffect 사용하면 됨.
- 모달이 닫혀야하는데 chatGPT가 준 방법으로 하면 모달에서 포커스가 사라졌을 때 모달이 안닫김 → window.addEventListener로 해결 → 아 근데 useCallback 쓰느라 코드가 너무 더러워짐. 사용자 입장에서 사용할때도 성능 안해치고 잘 사용할 수 있을까? 라는 걱정.
- 백드롭 눌렀을 때도 닫히는 기능
  ```javascript
  const backdropImpl = backdrop ? (
    <div onClick={handleBackdropClick}></div>
  ) : null;
  ```
  - 그냥 이렇게만 해줌. backdrop이 Parent가 아니라 sibling이라서 따로 뭐 해줄 필요없이, (뭐 안에서 드래그 했을 때 안닫히게 하기, 다이얼로그를 눌렀을 때 안닫히게 하기…) 그냥 저렇게만 해주니까 됨.
- 풀스크린, 모달 얼라인, 애니메이션, aria-..등 구현
- 스타일링에 관해서: 귀찮아서 스킵. 다만 헤더-바디-푸터 별로 각각 border-radius 준다는 사실, 그리고 각각 이러한 요소별로 패딩 주고 있다는 사실만 기억. (얘네를 감싸고 있는 컨테이너에서 한번에 안 줌)

### 커밋 주소

[커밋 주소](https://github.com/confidential-nt/components-study/commit/a64a46a04ecba452092b7d8d61fe1e3c38747e53)
