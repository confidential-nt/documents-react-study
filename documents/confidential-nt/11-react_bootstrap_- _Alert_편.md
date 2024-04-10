### 들어가면서

꼭 해보고 싶었던 것 중 하나가, MUI나 react bootstrap 같이 유명한 컴포넌트 라이브러리 하나를 택해서, 그 라이브러리의 컴포넌트들이 어떤 식으로 구현이 되어있는지 분석하고, 분석한 내용 및 기술 블로그에서 읽었던 내용을 바탕으로 실제로 그 컴포넌트를 직접 구현해보는 것이었다.

배울 것이 많다고 생각했기 때문이다. 특히 컴포넌트 설계와 관해서. 그리고 이와 같은 오픈 소스들은 좋은 코드들이기 때문에, 내 시야를 넓히는 데도 매우 도움이 될 것이다.

왜 react bootstrap을 택했는가?

MUI와 react bootstrap 사이에서 고민을 했더랬다. 사실 먼저 MUI의 코드를 까봤는데, 공부를 이제 막 시작하는 입장에서 MUI의 코드는 너무 복잡하게 느껴졌다. 이에 비해 react bootstrap은 폴더 구조도 잘 분석이 되고, 코드 구조도 낯설지 않았다. 따라서 react bootstrap을 선택했다.

공부 방식은 다음과 같다.

1. 공부하고픈 컴포넌트를 택한다.
2. 공식 문서를 살펴보면서 어떤 구조를 가지고 있는지 파악한다.
3. 실제 코드를 까보면서 어떤 라이브러리를 사용하고 있는지, 컴포넌트 구조가 어떤지, 주목해볼만한 좋은 코딩 습관은 무엇인지 등을 분석하고 공부한다
4. 공부한 내용을 바탕으로 공부한 컴포넌트를 직접 구현해본다.

첫 시작이므로, 어렵지 않은 컴포넌트부터 살펴보기로 했다. 따라서 가장 기본적인 컴포넌트 중 하나인 Alert를 시작점으로 삼았다.

### 공부한 내용

[공식문서 주소](https://react-bootstrap.github.io/docs/components/alerts)

#### 공식 문서로부터 알 수 있었던 것

1. Alert.Heading / Alert.Link등을 따로 제공. (compound pattern)
2. close 버튼을 dismissible이라는 props로 따로 제공하거나, button으로 직접 컨트롤 할 수 있게함.
3. 위에서 명시된 요소 빼고는 그냥 children으로 다 때려박을 수 있게 함.
4. 색상은 variant prop으로 조정. -> 적절한 범위의 확장성 제공

#### 코드로부터 알 수 있었던 것

1. prop-types 라이브러리를 사용: 얘가 무슨 라이브러냐? React 프로그래밍에서 사용되는 라이브러리로, 컴포넌트가 받아야 하는 props의 타입을 사전에 지정할 수 있게 해주는 것. 얘 대신 타입스크립트를 사용할수도 있음. 근데 이 프로젝트는 이미 타입스크립트 쓰고 있는데 왜 굳이 prop-types 쓰고 있을까? 정확한 이유는 알 수 없으나 호환성 문제나 런타임 체크 기능(외부라이브러리 사용할 때 특히 유용)을 사용하기 위해서이기 때문이 아닐까
2. uncontrollable 이라는 라이브러리 사용 중 : 비제어 컴포넌트를 쉽게 다룰 수 있도록 도와주는 것. 특히 제어 컴포넌트 / 비제어 컴포넌트 전환 쉽게 할 수 있게 해줌.
   [공식문서](https://www.npmjs.com/package/uncontrollable)
   [chatGPT와의 대화](https://chat.openai.com/share/764986a1-8c8c-49b5-a9cf-6b05c65c23b0)
3. 내부적으로 ThemeProvider라는 context 만들어서 사용. 여기서 주목해볼만한 점:

```javascript
export function useBootstrapBreakpoints() {
  const { breakpoints } = useContext(ThemeContext);
  return breakpoints;
}

export function useBootstrapMinBreakpoint() {
  const { minBreakpoint } = useContext(ThemeContext);
  return minBreakpoint;
}
```

위와 같이 자주 사용될 것들은 useContext와 함께 유틸로 만들어서 빼둠. 4. classnames 라이브러리 사용 : 클래스 이름을 조건부로 추가하거나 여러 클래스 이름을 쉽게 결합할 수 있게 해줌. 5. 컴포넌트 Props 타입 선언에서 주목해볼점:

```javascript
export interface AlertProps extends React.HTMLAttributes<HTMLDivElement> {
  bsPrefix?: string;
  variant?: Variant;
  dismissible?: boolean;
  show?: boolean;
  onClose?: (a: any, b: any) => void;
  closeLabel?: string;
  closeVariant?: CloseButtonVariant;
  transition?: TransitionType;
}
```

인터페이스를 사용. - interface vs type : 몇 가지 기능상의 차이점 빼고는 엄청난 성능상의 차이라거나 그런 건 없음. 다만, 전자는 extends의 개념이 확실하기(후자도 물론 &로 타입을 확장할수는 있으나 나는 extends가 더 가독성이 좋다고 느낌) 때문에 지금까지는 type을 써왔으나 앞으로는 interface를 사용하는 게 더 나을 것 같다고 생각함. - 참고한 문서들 - [Choosing Between “type” and “interface” in React](https://medium.com/nerd-for-tech/choosing-between-type-and-interface-in-react-da1deae677c9) - [type과 interface](https://hyung1.tistory.com/67) - [chatGPT와의 대화](https://chat.openai.com/share/cb00c10a-a8c6-479b-b55a-e5e093f8ef39) 6. AlertProps가 React.HTMLAttributes를 extends. - [가치있는 공통 컴포넌트 만들기 - 카카오 기술 블로그](https://fe-developers.kakaoent.com/2024/240116-common-component/) - 위 아티클에 의하면, 모든 상황을 variant로 대응하는 것에는 한계가 있기 때문에 기존 돔을 확장하는 경우도 있다고 함. 무조건 권장되는 것은 아님 - 내 추측에는, 이 라이브러리는 DOM Element처럼 사용될 것을 가정하고 있는 확장성이 엄청 높아야하는 UI 라이브러리이기 때문에 extends해도 되는 거라 생각. 7. forwardRef로 감싸져 있음 → ref로 사용될 가능성 생각한듯. 8. 본격 컴포넌트 코드 분석

```javascript
const Alert = React.forwardRef<HTMLDivElement, AlertProps>(
  (uncontrolledProps: AlertProps, ref) => {
    const {
      bsPrefix,
      show = true,
      closeLabel = 'Close alert',
      closeVariant,
      className,
      children,
      variant = 'primary',
      onClose,
      dismissible,
      transition = Fade,
      ...props
    } = useUncontrolled(uncontrolledProps, {
      show: 'onClose',
    });

    const prefix = useBootstrapPrefix(bsPrefix, 'alert');
    const handleClose = useEventCallback((e) => {
      if (onClose) {
        onClose(false, e);
      }
    });
    const Transition = transition === true ? Fade : transition;
    const alert = (
      <div
        role="alert"
        {...(!Transition ? props : undefined)}
        ref={ref}
        className={classNames(
          className,
          prefix,
          variant && `${prefix}-${variant}`,
          dismissible && `${prefix}-dismissible`,
        )}
      >
        {dismissible && (
          <CloseButton
            onClick={handleClose}
            aria-label={closeLabel}
            variant={closeVariant}
          />
        )}
        {children}
      </div>
    );

    if (!Transition) return show ? alert : null;

    return (
      <Transition unmountOnExit {...props} ref={undefined} in={show}>
        {alert}
      </Transition>
    );
  },
);
```

- div로 구현. 단, role=”alert”를 명시
- show, onClose는 일단 바깥에서 주입. 주입 안하면 내부에서 관리. (제어 - 비제어 둘다 가능)
- props를 일단 통째로 받아서 코드 작성하는 곳에서 해체
- 컴포넌트에 필요한 상태, 함수를 먼저 정의하고 alert라는 변수를 정의해서 일단 대표적인 alert의 형태를 여기 넣어줌
- 따로 더 필요한 변화(트랜지션)가 요구되면 여기 안에 끼워넣은 형태로 리턴하고 아니면 그냥 alert 변수를 리턴.
- AlertHeading의 성분 :
  1. divWithClassName이라는 컴포넌트를 호출 후 h4라는 것을 인자로 넣어주고 있었음
  2. 그래서 난 그냥 div로 구현 후 role에 heading 명시
  3. 나중에 스타일 바꿔서 heading처럼 만들어볼 것.
- AlertLink
  1.  구현 사항보니 얘는 a 태그로 만들어져 있음
  2.  AlertLink의 역할 : For links, use the `<Alert.Link>` component to provide matching colored links within any alert.

### 구현 연습

Alert.tsx

```javascript
import { HTMLAttributes, forwardRef, useState } from "react";
import CloseButton from "../CloseButton/CloseButton";
import AlertHeading from "../AlertHeading/AlertHeading";
import "./Alert.css";
import { Variant } from "../../type";
import classNames from "classnames";
import AlertLink from "../AlertLink/AlertLink";

export interface AlertProps extends HTMLAttributes<HTMLDivElement> {
  dismissible?: boolean;
  show?: boolean;
  onClose?: () => void;
  variant?: Variant;
}

const AlertImpl = forwardRef<HTMLDivElement, AlertProps>(
  (allProps: AlertProps, ref) => {
    const {
      dismissible,
      onClose,
      show: controlledShow,
      variant = "primary",
      children,
      className,
      ...props
    } = allProps;

    const isControlled = controlledShow !== undefined;
    const [show, setShow] = useState<boolean>(true);

    const handleClose = () => {
      if (!isControlled) {
        setShow(false);
        return;
      }
      onClose?.();
    };

    const alert = (
      <div
        role="alert"
        {...props}
        ref={ref}
        className={classNames(className, "alert", variant && variant)}
      >
        {dismissible ? <CloseButton onClick={handleClose} /> : null}
        {children}
      </div>
    );

    if (isControlled) {
      return controlledShow ? alert : null;
    }

    return show ? alert : null;
  }
);

const Alert = Object.assign(AlertImpl, {
  Heading: AlertHeading,
  Link: AlertLink,
});

export default Alert;
```

AlertHeading.tsx

```javascript
import { HTMLAttributes, forwardRef } from "react";
import "./AlertHeading.css";
import classNames from "classnames";

interface AlertHeadingProps extends HTMLAttributes<HTMLDivElement> {}

const AlertHeading = forwardRef<HTMLDivElement, AlertHeadingProps>(
  ({ children, className, ...props }: AlertHeadingProps, ref) => {
    return (
      <div
        ref={ref}
        role="heading"
        {...props}
        className={classNames(className, "alertHeading")}
      >
        {children}
      </div>
    );
  }
);

export default AlertHeading;
```

AlertLink.tsx

```javascript
import { AnchorHTMLAttributes, forwardRef } from "react";
import "./AlertLink.css";
import classNames from "classnames";

export interface AlertLinkProps
  extends AnchorHTMLAttributes<HTMLAnchorElement> {}

const AlertLink = forwardRef<HTMLAnchorElement, AlertLinkProps>(
  ({ href, children, ...props }: AlertLinkProps, ref) => {
    return (
      <a href={href} ref={ref} {...props} className={classNames("alertLink")}>
        {children}
      </a>
    );
  }
);

export default AlertLink;
```

App.tsx

```javascript
import { useRef, useState } from "react";
import "./App.css";
import Alert from "./components/Alert/Alert";

function App() {
  const [show, setShow] = useState < boolean > true;
  const alertRef = useRef < HTMLDivElement > null;

  return (
    <main>
      <Alert
        show={show}
        onClose={() => setShow(false)}
        variant="danger"
        dismissible
        ref={alertRef}
      >
        <Alert.Heading>경고문</Alert.Heading>
        <p>
          배가 고파요
          <Alert.Link href="#">요기요</Alert.Link>
        </p>
      </Alert>
    </main>
  );
}

export default App;
```

화면
![](https://velog.velcdn.com/images/youyoy689/post/131e5448-5ad7-43db-b389-90cd319df638/image.png)

- 라이브러리 쓰지 않고도 controlled + (like)uncontrolled 전환할 수 있게 만들기
  - [가치있는 공통 컴포넌트 만들기 - 카카오 기술 블로그](https://fe-developers.kakaoent.com/2024/240116-common-component/)
    - 위 내용 참고해서 만듦.
- 스타일에 관하여
  - 리액트 부트스트랩은 스타일을 어떻게 입히고 있나? 리액트 부트스트랩 쓸 때 외부 스타일관련 부트스트랩 파일도 같이 다운 받아야 함 -> 외부 파일 필요.
    - 처음에는 css module classes 사용하려다가 생각해보니까 관리가 복잡해짐. 랜덤 이름으로 클래스를 넣어주기 때문에 이런 경우, Alert 하위 컴포넌트를 스타일링 할 때, 상위 컴포넌트를 특정하기가 힘듦. (특히 AlertLink 같이, 상위에 정의된 스타일링에 따라 스타일링이 달라져야하는 경우..) 이런 경우 상위 컴포넌트에 스타일링을 넣어줘야하는데, 그러면 어떤 거는 하위 컴포넌트 css 파일에 스타일링이 들아가있고 어떤거는 상위에 들어가있는 혼돈이 발생 → 그냥 일반 css쓰자라는 결론.
    - tailwind 같은 거 사용하지 않은 이유: 고유의 스타일링을 입힌다는 목적이 있다는 가정 하에 진행해서

### 커밋 주소

[커밋 주소](https://github.com/confidential-nt/components-study/commit/fc3e06453cac122982243b05a116d0d901e40786)
