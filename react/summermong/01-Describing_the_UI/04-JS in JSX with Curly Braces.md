# ✏️ Describing the UI : JS in JSX with Curly Braces

### 따옴표로 문자열 전달하기

JSX에 문자열 속성을 전달하려면 작은따옴표 또는 큰따옴표로 묶는다.

```
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/7vQD0fPs.jpg"
      alt="Gregorio Y. Zara"
    />
  );
}
```

src나 alt를 동적으로 지정하려면 따옴표를 {}로 대체하면 된다.

```
export default function Avatar() {
  const avatar = 'https://i.imgur.com/7vQD0fPs.jpg';
  const description = 'Gregorio Y. Zara';
  return (
    <img
      className="avatar"
      src={avatar}
      alt={description}
    />
  );
}
```

JSX 내부에서는 중괄호로 두 가지 방법만 사용할 수 있다.

1. JSX 태그 안에 직접 텍스트로 사용하기
<h1>{name}'s To Do List</h1> (⭕️)
<{tag}>Gregorio Y. Zara's To Do List</{tag}> (❌)

2. = 기호 바로 뒤에 오는 속성
   src={avatar}는 변수 아바타를 읽지만, src="{avatar}"는 문자열 "{avatar}"를 전달한다.

---

### 이중 중괄호 사용 (JSX 내 CSS 및 다른 객체)

문자열, 숫자, JS 표현식 외에도 JSX로 객체를 전달할 수 있다.
객체는 중괄호로 표시하기 때문에 JSX에서 JS 객체를 전달하려면 다른 중괄호 쌍으로 객체를 감싸야 한다.

```
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Improve the videophone</li>
      <li>Prepare aeronautics lectures</li>
      <li>Work on the alcohol-fuelled engine</li>
    </ul>
  );
}
```

위와 같이 주로 인라인 CSS 스타일에서 볼 수 있다.
React에서는 유지보수 및 가독성을 위해 인라인 스타일을 지양하지만 필요한 경우 style 어트리뷰트에 객체를 전달한다.
이런 경우 인라인 style 프로퍼티는 카멜 케이스로 작성된다.

---

📌 따옴표 안의 JSX 속성은 문자열로 전달된다.
📌 중괄호를 사용하면 JS 로직과 변수를 마크업으로 가져올 수 있다.
📌 중괄호는 JSX 태그 콘텐츠 내부 또는 속성의 = 뒤에서 작동한다.
📌 {{}}는 JSX 중괄호 안에 들어 있는 JS 객체다.

---

출처: https://react-ko.dev/learn/javascript-in-jsx-with-curly-braces
