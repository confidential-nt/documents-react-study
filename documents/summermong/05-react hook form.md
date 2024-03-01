## **react-hook-form**

아래 글에서 Web IDE의 로그인/회원가입을 구현할 때 **react-hook-form**을 사용했다고 적었다.

수료 후 우리 팀은 꾸준히 Web IDE 리팩토링을 하고 있는데, 이제 슬슬 나도 코드를 뜯어보기 위해 에디터를 들여다 보다가 react-hook-form을 좀 더 잘 이해해 봐야겠다는 생각이 들었다. (왜냐하면 저 때는 데드라인 맞추는 게 중요해서 공식 문서를 제대로 안 봤기 때문)
(https://summermong.tistory.com/423)

**react-hook-form**은 react 기반의 폼 관리 라이브러리로, 폼 상태와 유효성 검사를 간단하게 관리해준다.
(https://react-hook-form.com/get-started)

react-hook-form을 안 쓰고 폼을 관리하면 대부분 useState, onSubmit, onChange 등을 써서 구현할 것이다.

그렇게 되면 폼 내 필드의 입력, 제출, 유효성 검사를 비롯한 오류 등 관리해야 하는 값이 많아져 자연스럽게 코드 양도 늘어나게 된다.

특히 **유효성 검사 같은 경우 하나만 해도 코드가 복잡해지기 때문에 유지보수 하기도 어려워진다.**

또, 입력 필드의 값이 하나만 바뀌어도 그 변화를 추적하기 때문에 불필요한 리렌더링이 발생하기도 한다.

이 코드는 기존 코드를 react-hook-form 없이 바꿔본 버전이다.

처음부터 react-hook-form으로 짜서 다시 생짜로 짜긴 시간이 좀 걸릴 것 같아 챗지피티에게 시켰다.

내가 생각한 것보단 깔끔하긴 한데 객체가 너무 많아서 뭔가 복잡해보인다;

```
import React, { useState, useRef } from 'react';
import { useNavigate } from 'react-router-dom';
import styles from './Signup.module.css';
import instance from '../api/CustomAxios';

const Signup = () => {
  const axiosInstance = instance();

  const [formData, setFormData] = useState({
    email: '',
    password: '',
    confirm: '',
    username: '',
  });

  const [errors, setErrors] = useState({
    email: '',
    password: '',
    confirm: '',
    username: '',
  });

  const [isEmailUnique, setIsEmailUnique] = useState(false);

  const navigate = useNavigate();

  const passwordInputRef = useRef(null);
  passwordInputRef.current = formData.password;

  const confirmID = async () => {
    const email = formData.email;

    if (email) {
      try {
        const response = await axiosInstance.post(
          `members/signup/email-check`,
          { email }
        );

        if (response.data.status === 200) {
          alert('사용 가능한 이메일입니다.');
          setIsEmailUnique(true);
        } else if (response.data.status === 400) {
          alert('사용할 수 없는 이메일입니다.');
          setErrors((prevErrors) => ({
            ...prevErrors,
            email: '사용할 수 없는 이메일입니다.',
          }));
          setIsEmailUnique(false);
        }
      } catch (error) {
        console.error(error);
      }
    }
  };

  const handleChange = (e) => {
    const { name, value } = e.target;

    let error = '';
    switch (name) {
      case 'email':
        error = !value.match(/^\S+@\S+$/i)
          ? '아이디 형식이 올바르지 않습니다.'
          : '';
        break;
      case 'password':
        error = !value.match(
          /^(?=.*[a-zA-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{7,15}$/i
        )
          ? '비밀번호 형식이 올바르지 않습니다.'
          : '';
        break;
      case 'confirm':
        error =
          value !== passwordInputRef.current
            ? '비밀번호가 일치하지 않습니다.'
            : '';
        break;
      case 'username':
        error = value.length > 10 ? '10자 이내로 입력해주세요.' : '';
        break;
      default:
        break;
    }

    setErrors((prevErrors) => ({
      ...prevErrors,
      [name]: error,
    }));

    setFormData((prevFormData) => ({
      ...prevFormData,
      [name]: value,
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();

    if (isEmailUnique) {
      const { email, password, username } = formData;

      axiosInstance
        .post(`members/signup`, { email, password, username })
        .then((response) => {
          if (response.data.status === 200) {
            alert('회원가입이 완료되었습니다.');
            navigate('/login');
          }
        })
        .catch((error) => {
          console.error(error);
        });
    } else {
      alert('이메일 중복을 확인하세요.');
    }
  };

  return (
    <div className={styles.Signup}>
      <div className={styles.subtitle}>Step for Developer</div>
      <div className={styles.title}>Hongsam IDE</div>
      <form className={styles.form} onSubmit={handleSubmit}>
        <label>ID</label>
        <div className={styles.id}>
          <input
            name="email"
            type="text"
            autoComplete="off"
            placeholder="이메일 형식으로 입력해주세요."
            value={formData.email}
            onChange={handleChange}
          />
          <button
            className={styles.confirmIdBtn}
            onClick={confirmID}
            type="button"
          >
            중복 확인
          </button>
        </div>
        {errors.email && <p>{errors.email}</p>}

        <label>Password</label>
        <input
          name="password"
          type="password"
          autoComplete="off"
          placeholder="영문+숫자+특수문자 조합의 7~15자로 입력해주세요."
          value={formData.password}
          onChange={handleChange}
        />
        {errors.password && <p>{errors.password}</p>}

        <label>Confirm Password</label>
        <input
          name="confirm"
          type="password"
          autoComplete="off"
          placeholder="비밀번호를 다시 입력해주세요."
          value={formData.confirm}
          onChange={handleChange}
        />
        {errors.confirm && <p>{errors.confirm}</p>}

        <label>Name</label>
        <input
          name="username"
          autoComplete="off"
          placeholder="이름을 10자 이내로 입력해주세요."
          value={formData.username}
          onChange={handleChange}
        />
        {errors.username && <p>{errors.username}</p>}

        <button className={styles.submitBtn} type="submit">
          Sign up
        </button>
      </form>
    </div>
  );
};

export default Signup;
```

react-hook-form을 사용하면 위 코드보다 라이브러리의 API로 폼을 더 쉽게 관리할 수 있다.

그래서 유효성 검사도 훨씬 간단하게 구현할 수 있고, 에러 처리 같은 부분도 더 간략하게 재사용할 수 있다.

또 위에서 말한 상태 대신 각 입력 필드의 참조를 사용해 불필요한 리렌더링을 방지해 성능을 개선할 수도 있다.

위 부분을 더 자세히 알아봤는데, 컴포넌트는 제어 컴포넌트/비제어 컴포넌트 2가지로 나눌 수 있다.

라이브러리 없이 개발하는 경우, useState 훅으로 데이터를 관리하는 경우를 제어 컴포넌트라고 한다.

React에 의해 값이 제어되고 따라서 값이 바뀔 때마다 리렌더링이 이뤄지는 것이다.

반면 **react-hook-form은 비제어 방식**으로 동작한다.

이는 React에 의해 값이 제어되지 않고 useRef로 데이터를 관리함을 의미한다.

**최근 공식 문서에서 useRef를 봤는데, useRef는 값이 변경되어도 리렌더링이 이루어지지 않는다.**

useRef로 해당 DOM 요소에 대한 참조를 얻어 직접 상태를 관리하기 때문에, 사용자의 입력이나 업데이트를 수동으로 처리할 수 있어 불필요한 리렌더링을 방지할 수 있는 것이다!

즉 **코드 개선을 비롯해 DX가 개선될 여지가 충분하다는 것!**

(역시 우선 라이브러리 없이 생짜로 개발을 해봐야 라이브러리의 소중함을 알게 된다... 🥲)

설치하는 방법은 다음과 같다.

```
npm install react-hook-form
yarn add react-hook-form
```

---

## **예시 및 사용 방법**

공식 문서에 나와 있는 간단한 예시다.

```
import { useForm } from "react-hook-form"


export default function App() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors },
  } = useForm()

  const onSubmit = (data) => console.log(data)
  console.log(watch("example")) // watch input value by passing the name of it

  return (
    /* "handleSubmit" will validate your inputs before invoking "onSubmit" */
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* register your input into the hook by invoking the "register" function */}
      <input defaultValue="test" {...register("example")} />
      {/* include validation with required or other standard HTML validation rules */}
      <input {...register("exampleRequired", { required: true })} />
      {/* errors will return when field validation fails  */}
      {errors.exampleRequired && <span>This field is required</span>}
      <input type="submit" />
    </form>
  )
}
```

먼저 useForm을 import한다.

**useForm은 register, handleSubmit, watch, formState 등 폼을 쉽게 관리할 수 있는 커스텀 훅이다.**

**register는 input 필드를 라이브러리에 등록 시켜서, 해당 필드에 대한 유효성 검사 규칙이나 기본값 등을 설정할 수 있게 한다.**

1번째 인자에는 관리하려는 Input의 이름, 2번째 인자에는 옵션을 적는다. (required, maxLength, 유효성 검사 등)

**handleSubmit은 폼이 제출 될 때 실행하는 함수다. 이 함수로 유효성 검사 수행 및 제출된 데이터를 처리하는 로직을 구현한다.**

input 필드의 데이터를 다루는 콜백 함수로, 콘솔에 찍어보면 폼의 데이터들이 나온다.

**watch는 input 필드의 데이터들을 submit 하기 전에 다룰 수 있는 기능이다.**

간단하게 계속 추적한다고 보면 된다. onChange 이벤트가 나타나는 것처럼 input 필드에 들어온 값을 감시한다.

예컨대 비밀번호를 입력할 때 대소문자 12자리라고 하는데, 11자리까지는 계속 에러를 표시하는 그런 기능을 구현할 때 유용하다.

**errors는 말 그대로 에러를 다루는 객체다.**

register에 등록한 input 필드에서 에러가 발생할 때 보여줄 메세지를 여기서 관리할 수 있다.

---

## **실제 사용하기**

이메일(아이디), 비밀번호, 비밀번호 확인, 이름 총 4가지의 input 필드를 가진 폼을 만들었다.

이메일 검사를 하지 않으면 회원가입을 할 수 없고, 간단하게 이메일은 @의 앞뒤로 공백이 아닌 문자여야 한다.

비밀번호는 영문+숫자+특수문자 조합의 7~15자여야 한다.

비밀번호 확인은 위 비밀번호와 같은지 확인해야 한다.

이름은 10자 내외로 입력해야 한다.

모든 input 필드는 반드시 입력해야 한다.

먼저 form 코드에는 onSubmit 시 onSignup이라는 함수를 매개로 handleSubmit을 호출해야 한다.

onSignup에는 input 필드 값인 email, password, username이 들어 있다.

여기서 이메일 검사를 했는지 검사 유무를 확인하는 isEmailUnique의 값을 불리언으로 바꿔준다.

사용할 수 없는 이메일의 경우 email에 포커스를 주고, 가능하면 불리언 값을 true로 바꿔준다.

```
  /* 이메일 중복 검사 확인 */
  const [isEmailUnique, setIsEmailUnique] = useState(false);

  const confirmID = async () => {
    const email = watch('email'); // 이메일 값을 가져옴

    if (email) {
      try {
        const response = await axiosInstance.post(
          `members/signup/email-check`,
          { email }
        );

        if (response.data.status === 200) {
          alert('사용 가능한 이메일입니다.');
          setIsEmailUnique(true);
        } else if (response.data.status === 400) {
          alert('사용할 수 없는 이메일입니다.');
          setFocus('email');
          setIsEmailUnique(false);
        }
      } catch (error) {
        console.error(error);
      }
    }
  };
```

input 필드는 이메일부터 보면 register에 email을 넣고 2번째 인수로 옵션에 필수 & 유효성 검사를 넣어줬다.

만약 email에서 에러가 발생하고, 그 에러의 타입에 따라 에러 메세지를 보여줄 수 있다.

이메일에서는 옵션이 2개이므로 에러 메세지도 2개다.

```
<form className={styles.form} onSubmit={handleSubmit(onSignup)}>
        <label>ID</label>
        <div className={styles.id}>
          <input
            name="email"
            type="text"
            autoComplete="off"
            placeholder="이메일 형식으로 입력해주세요."
            {...register('email', {
              required: true,
              pattern: /^\S+@\S+$/i,
            })}
          />
          <button
            className={styles.confirmIdBtn}
            onClick={confirmID}
            type="button"
          >
            중복 확인
          </button>
        </div>
        {errors.email && errors.email.type === 'required' && (
          <p>이 칸을 입력해주세요.</p>
        )}
        {errors.email && errors.email.type === 'pattern' && (
          <p>아이디 형식이 올바르지 않습니다.</p>
        )}
```

패스워드도 비슷하게 진행하면 된다.

패스워드는 대신 영문+숫자+특수문자 조합의 7~15자라 조금 더 외계어같이 생겼다.

minLength, maxLength는 유효성 검사에서 확인했기 때문에 굳이 설정하지 않았다.

```
<label>Password</label>
        <input
          name="password"
          type="password"
          autoComplete="off"
          placeholder="영문+숫자+특수문자 조합의 7~15자로 입력해주세요."
          {...register('password', {
            required: true,
            pattern:
              /^(?=.*[a-zA-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{7,15}$/i,
          })}
        />
        {errors.password && errors.password.type === 'required' && (
          <p>이 칸을 입력해주세요.</p>
        )}
        {errors.password && errors.password.type === 'pattern' && (
          <p>비밀번호 형식이 올바르지 않습니다.</p>
        )}
```

다음은 비밀번호 확인이다.

비밀번호 확인은 현재 input에 들어온 값(value)이 이전 input, 즉 name="password"에 들어온 값과 동일한지 확인하면 된다.

이 때 validate 옵션을 추가했는데, value가 passwordInputRef의 current 값과 같은지 확인한다.

순서가 바뀌었는데 password input을 watch 한 걸 passwordInputRef.current라고 했다.

name="password"의 값을 실시간으로 감시하고 있는 passwordInputRef와 현재 value가 같은지 보면 되는 것이다.

```
        <label>Confirm Password</label>
        <input
          name="confirm"
          type="password"
          autoComplete="off"
          placeholder="비밀번호를 다시 입력해주세요."
          {...register('confirm', {
            required: true,
            validate: (value) => value === passwordInputRef.current,
          })}
        />
        {errors.confirm && errors.confirm.type === 'required' && (
          <p>이 칸을 입력해주세요.</p>
        )}
        {errors.confirm && errors.confirm.type === 'validate' && (
          <p>비밀번호가 일치하지 않습니다.</p>
        )}

...

  /* 비밀번호 일치를 위한 useRef */
  const passwordInputRef = useRef(null);
  passwordInputRef.current = watch('password');
```

마지막으로 이름에서 maxLength를 10으로 제한할 수 있다.

```
        <label>Name</label>
        <input
          name="username"
          autoComplete="off"
          placeholder="이름을 10자 이내로 입력해주세요."
          {...register('username', {
            required: true,
            maxLength: 10,
          })}
        />
        {errors.username && errors.username.type === 'required' && (
          <p>이 칸을 입력해주세요.</p>
        )}
        {errors.username && errors.username.type === 'maxLength' && (
          <p>10자 이내로 입력해주세요.</p>
        )}
```

그리고 Sign up 버튼을 누르면 폼이 제출된다.

폼이 제출되면 onSignup 함수가 실행되고 여기에 지금까지 폼에 있던 값들이 전달된다.

이메일 검사를 정상적으로 통과했다면 true가 되었을 테니 Data를 전달하고 회원가입이 잘 진행된다.

만약 이메일 검사를 통과하지 않았다면 제출해도 넘어가지 않게 조건을 걸어두었다.

```
        <button className={styles.submitBtn} type="submit">
          Sign up
        </button>

...

/* 회원가입 폼 제출 함수 */
  const onSignup = ({ email, password, username }) => {
    /* 이메일 중복 검사 완료 */
    if (isEmailUnique) {
      const Data = {
        email,
        password,
        username,
      };

      axiosInstance
        .post(`members/signup`, Data)
        .then((response) => {
          /* 회원가입 완료 */
          if (response.data.status === 200) {
            alert('회원가입이 완료되었습니다.');
            navigate('/login');
          }
        })
        .catch((error) => {
          console.error(error);
        });
      /* 이메일 중복 검사 미완료 */
    } else {
      alert('이메일 중복을 확인하세요.');
    }
  };
```

---

## **끝으로**

현재 이 코드는 10월, 한참 개발에 재미를 붙인 3-4개월 차에 작성되었다.

지금 보니 너무 미흡한 부분이 많고 좀 더러운데 여기저기 얽힌 게 많아 서둘러 조금씩 고쳐봐야겠다.

**더 나은 코드 및 피드백은 언제나 환영입니다...**

참고

[https://velog.io/@boyeon_jeong/React-Hook-Form](https://velog.io/@boyeon_jeong/React-Hook-Form)

[https://choi-records.tistory.com/entry/React-react-hook-form-%EC%82%AC%EC%9A%A9%EB%B2%95useForm-watch](https://choi-records.tistory.com/entry/React-react-hook-form-%EC%82%AC%EC%9A%A9%EB%B2%95useForm-watch)
