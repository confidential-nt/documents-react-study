# 웹 접근성

### 웹 접근성이란

접근성: 모든 사람이 쉽게 접근할 수 있도록 제품, 디바이스, 서비스, 환경을 설계함
'모든 사람'에는 남녀노소, 디지털 사용 능력치가 높고 낮은 사람들, 비장애인/장애인 등 모두 포함

즉, **모두를 위한 설계를 고려하는 기술**을 웹 접근성이라고 하며,
서비스를 제공할 때는 웹 접근성을 고려하고 모든 유저를 위한 개선된 경험을 제공해야 한다.

어떻게 웹 접근성을 고려할 수 있을까?

#### 1. 웹 표준

웹 표준을 잘 지킨다 === 웹 접근성이 완전히 보장된다는 아니지만, **올바른 목적으로 적절한 HTML 요소를 사용하는 것(=시멘틱 HTML 준수)**만으로도 사용성과 접근성 대부분이 보장된다.

홈 버튼을 만든다고 했을 때,

![image](https://techblog.woowahan.com/wp-content/uploads/2023/12/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-12-26-%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE-7.30.34-750x189.png)

스크린 리더기는 div 태그가 버튼임을 알 수 없어 클릭할 수 있는 요소라고 알려주지 않는다.
이로 인해 시각 장애가 있는 유저들은 홈 버튼을 인지할 수 없어 홈으로 가는 기능을 찾지 못하게 된다.

button 태그에는 PC, MO에서 탭이나 스와이프를 통해 이동하고 클릭할 수 있는 **키보드 접근성**이 내장되어 있지만 div 태그는 이러한 버튼의 속성 및 기능을 사용할 수도 없게 된다.

이렇듯 적절한 시멘틱 마크업을 사용하면 의미를 더 잘 파악할 수 있고, SEO에서도 유리하며 개발 시 가독성도 훨씬 좋아진다.

#### 2. 이미지 대체 텍스트

모든 이미지에 대체 텍스트를 제공해야 하는 것은 아니다.

![image](https://techblog.woowahan.com/wp-content/uploads/2023/12/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-12-26-%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE-7.58.57-750x376.png)

예제처럼 단순 꾸밈 요소의 경우 대체 텍스트를 제공하면 오히려 불필요한 정보 제공으로 정보 전달을 방해하게 된다.

![image](https://techblog.woowahan.com/wp-content/uploads/2023/12/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-12-26-%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE-8.10.56-750x398.png)

이 경우에도 '헐'이라는 글자가 보이지만, 실질적으로 제공해야 하는 정보가 아닌 단순 꾸밈 요소이기 때문에 텍스트를 제공하면 안된다.

그렇다면 단순 꾸밈 요소에는 alt 속성을 아예 생략해야 하는 걸까?
그렇지 않다. 이 경우에는 반드시 빈 alt=""로 제공되어야 한다.

속성을 아예 없애는 게 아니라 빈 값으로 넣는 이유는, 스크린 리더기가 이미지의 alt 속성이 없을 경우 이미지명을 읽어주기 때문이다. 즉 빈 값이면 꾸밈 요소로 판단해 넘어가기 때문에 유의미하게 콘텐츠의 의미를 전달하거나 묘사하는 시각적 콘텐츠가 아니라면 alt=""를 넣어줘야 한다.

#### 3. 접근성을 고려한 디자인

1. 접근하기 쉬운 색상과 대비를 사용하기
2. 배경과 텍스트의 대비를 고려하기
3. 적절한 글꼴 크기 사용하기 (권장 크기는 16px)
4. ARIA 속성 사용하기

#### 4. WAI-ARIA

Web Accessibility Initiative - Accessible Rich Internet Application
=> W3C에서 발표한 기술 명세로 접근성 및 상호 운용성을 향상시키기 위한 목적으로 탄생

역할(Role), 속성(Property), 상태(State) 정보를 제공해 접근성을 개선하도록 지원한다.
역할은 UI에 포함된 특정 컴포넌트의 '역할'을 정의한다.
속성은 컴포넌트의 특징이나 상황을 정의하며 'aria-'라는 접두사를 사용한다.
상태는 컴포넌트의 상태 정보를 정의한다.

1. 역할

```
<button role="button">버튼</button> -> X
```

HTML 요소가 가지고 있는 암묵적인 역할이 있기 때문에 중복해서 정의하지 않아야 한다.

```
<button role="presentation">버튼</button> -> X
```

button 태그처럼 요소의 역할을 알려줘야 하는 경우에는 presentation을 부여해선 안된다.
보조 기기는 **role="presentation"으로 지정된 요소 본연의 역할을 무시하고 단순 정보 전달을 위한 요소로 인식**하기 때문이다.

```
<h1 role=”button”>배달 홈</h1> -> X
```

role으로 본래 요소의 의미를 변경하는 것도 지양해야 한다. (키보드 접근성 제공 불가)

2. 속성 및 상태

```
<div>
  <button aria-label="Search">
    <i class="fa fa-search"></i>
  </button>
  <div role="tabpanel" aria-labelledby="tab1">
    <p>Tab panel content goes here.</p>
  </div>
  <button aria-expanded="false" aria-controls="collapse1">Show More</button>
  <div id="collapse1" aria-hidden="true">
    <p>Additional content goes here.</p>
  </div>
</div>
```

위의 예는 “Search”라는 aria 텍스트가 있는 검색 버튼의 역할에 대한 정보를 제공한다. aria-expanded 속성은 연결된 div 요소의 콘텐츠가 확장 또는 축소되는지 여부를 나타내고, aria-controls 속성은 연결된 div의 ID를 참조하는 데 사용되고,
aria-hidden 속성은 콘텐츠가 접혀 있을 때 숨겨져 있음을 나타내는 데 사용된다.

최신 브라우저는 aria-_의 속성값에 대소문자를 구분하지 않는 것으로 처리하지만
모든 브라우저의 버전 및 보조 기술들이 올바르게 구문 분석을 하지는 않는다.
따라서 role이나 aria-_ 속성값 지정 시 소문자를 사용하는 것이 좋다.

3. 랜드마크
   AARIA에서 스크린 리더 사용자가 웹사이트를 쉽게 탐색할 수 있도록 제공하는 표지판 같은 기능.
   화면에서 제공되는 콘텐츠 영역이 어떤 구조를 가지고 어떤 역할을 하는지 식별할 수 있게 한다.

랜드마크는 총 8가지가 있다.

Banner: header와 유사한 기능 (사이트 전체에 걸쳐 동일한 형태로 제공),
navigation: nav 요소와 동일한 기능 (중복 X, aria-label 속성과 어떤 영역인지 정보 제공),
search: 검색을 위한 입력 폼 영역에서 사용,
Main: main 요소와 동일한 기능 (중복 사용 X),
region: section 요소와 유사한 기능,
complementary: aside 요소와 동일한 기능,
form: form 요소와 동일한 기능,
contentinfo: footer와 유사한 기능

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ARIA Landmark Examples</title>
</head>
<body>

  <!-- ARIA 랜드마크 예제 -->

  <!-- 페이지 랜드마크 -->
  <header role="banner">
    <h1>My Website</h1>
    <nav role="navigation">
      <!-- 내비게이션 랜드마크 -->
      <ul>
        <li><a href="#home">Home</a></li>
        <li><a href="#about">About</a></li>
        <li><a href="#contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <!-- 주요 컨텐츠 랜드마크 -->
  <main role="main">
    <section id="home">
      <h2>Welcome to My Website</h2>
      <p>This is the home section of the website.</p>
    </section>
    <section id="about">
      <h2>About Us</h2>
      <p>Learn more about our company and mission.</p>
    </section>
  </main>

  <!-- 사이드바 랜드마크 -->
  <aside role="complementary">
    <h2>Related Links</h2>
    <ul>
      <li><a href="#link1">Link 1</a></li>
      <li><a href="#link2">Link 2</a></li>
    </ul>
  </aside>

  <!-- 푸터 랜드마크 -->
  <footer role="contentinfo">
    <p>&copy; 2024 My Website. All rights reserved.</p>
  </footer>

</body>
</html>
```

ARIA를 적용하면 브라우저의 접근성 API에 추가 정보만 노출되고 DOM에는 영향을 주지 않는다.
상태 및 속성은 widget, live region, d&d, relationship으로 분류할 수 있다.

아래 예제는 widget의 modal, live region의 live 속성에 대한 예시다.

![image](https://techblog.woowahan.com/wp-content/uploads/2023/12/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-12-29-%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB-9.30.07-750x363.png)

배경을 가려주는 backdrop을 제외한 영역이 얼럿 역할을 하기에
role을 alertdialog로 설정, aria-modal 속성 true
aria-labelledby 속성에서 라벨, aria-describedby로 설명 제공

![image](https://techblog.woowahan.com/wp-content/uploads/2024/01/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2024-01-02-%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB-8.27.58-750x368.png)

p 태그의 aria-live="assertive" 설정으로 현재 설정된 값이 업데이트 되면 유저에게 즉시 알려준다. 이 설정을 통해 유저에게 정보가 업데이트됨을 알려준다.

---

참고
https://techblog.woowahan.com/15541/
https://medium.com/@yujso66/%EB%B2%88%EC%97%AD-%EC%9B%B9-%EC%A0%91%EA%B7%BC%EC%84%B1-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0-%ED%94%84%EB%9F%B0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%A5%BC-%EC%9C%84%ED%95%9C-%EA%B0%80%EC%9D%B4%EB%93%9C-cdac7a1e2710
