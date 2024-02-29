## State: A Component's Memory

### 들어가면서

컴포넌트는 상호작용의 결과로 화면의 내용을 변경해야 하는 경우가 있음. 그리고 변경을 위해 '현재의 상태'를 기억하고 있어야.

React에서는 이런 종류의 컴포넌트별 메모리를 state라고 한다.

### 일반 변수로 충분하지 않은 경우

어떤 컴포넌트가 있고 그 컴포넌트 안에 지역변수인 let index를 선언하고 이것을 0으로 초기화 했다고 해보자.

버튼을 클릭했을 때 발생하는 이벤트를 처리하기 위해 handleClick이라는 핸들러를 만들고 이곳에서 index의 값을 변경한다고 해보자.

그리고 index가 변경하는 것을 이용해, 화면에 다른 사진을 표시하려 한다고 해보자.

과연 잘 동작할까? 안된다. 그 이유는 다음 두 가지.

1. 지역변수의 값은 일단 기본적으로 렌더링 간에 유지가 되지 않는다.
2. 이보다 더 중요한 것은, 일반 변수의 변경은 리렌더링을 촉발하지 않는다.

그래서 우리가 필요한 것은 렌더링 간의 데이터 유지, 리렌더링 촉발인데 useState가 이것들을 제공한다.

1. 렌더링 사이에 데이터를 유지하기 위한 state 변수.
2. 변수를 업데이트하고 React가 컴포넌트를 다시 렌더링하도록 촉발하는 state 설정자 함수.

### state 변수 추가하기

- 참고: useState가 반환하는 배열에는 항상 정확히 두 개의 항목이 있음.

(나머지 내용 생략. 그냥 useState 사용법에 관한 내용.)

### 첫 번째 훅 만나기

리액트에서는 "use"로 시작하는 함수를 훅이라고 한다.

문서상 useState는 당신이 첫번째로 만난 훅이다.

**훅은 React가 렌더링 중일때만 사용할 수 있는 특별한 함수.**(이 부분과 관련해선 뒤에 더 자세한 설명이 나온다고 함.) 이를 통해 다양한 React 기능을 연결할 수 있다.

- 참고:
  - **훅(use로 시작하는 함수)은 “컴포넌트의 최상위 레벨” (최상위 컴포넌트 아님) 또는 커스텀 훅에서만 호출 가능.** 조건문, 반복문 또는 기타 중첩된 함수 내부에서는 훅을 호출할 수 없음. **훅은 함수이지만 컴포넌트의 필요에 대한 무조건적인 선언** 파일 상단에서 모듈을 “import”하는 것과 유사하게 컴포넌트 상단에서 React 기능을 “사용”.
  - 단순한 함수보다는 **컴포넌트가 필요로 하는 것들에 대한 '무조건적 선언'** 이라는 의미.
  - 여기서 말하는 무조건적 선언이란? **훅을 사용할 때, 그것들이 컴포넌트의 렌더링 과정에서 항상 일관된 순서로 호출되어야 한다는 리액트의 규칙을 지칭.** 즉, 조건부 논리 안에서 훅을 호출하거나 반복문, 중첩된 함수 내에서 훅을 사용해서는 안됨. 여기 안에서 호출될 경우 일관된 순서로 호출되지 못하니까.
  - 이러한 규칙은 리액트가 내부적으로 훅의 상태를 관리하는 방식과 관련.
  - 훅을 컴포넌트의 필요에 대한 선언으로 생각하는 것은, 개발자가 컴포넌트의 상태 관리, 사이드 이펙트 처리, 컨텍스트 사용 등의 기능을 구현할 때, **이러한 기능들이 컴포넌트의 구조와 라이프 사이클에 밀접하게 통합되어 있음을 인식하게함.**
  - 예를 들어, useState 훅은 컴포넌트의 상태 관리를, useEffect 훅은 컴포넌트가 렌더링된 후에 실행해야 하는 사이드 이펙트를 처리하는 방법을 제공.
  - 결론적으로, **리액트 훅을 사용하는 것이 단순한 함수 호출 이상의 의미를 가지며, 컴포넌트의 필요와 기능을 명시적으로 선언하는 방법론이라는 점을 강조**하는 것.

### useState 해부하기

useState의 유일한 인수는 state 변수의 초기값. 이 예제에서는 useState(0)에 의해 index의 초기값이 0으로 설정되어 있음.

**컴포넌트가 렌더링될 때마다** useState는 두 개의 값을 포함하는 배열을 제공

1. 저장한 값을 가진 state 변수(index).
2. state 변수를 업데이트하고 React가 컴포넌트를 다시 렌더링하도록 촉발할 수 있는 state 설정자 함수

실제 작동 방식은 다음과 같음

```javascript
const [index, setIndex] = useState(0);
```

1. 컴포넌트가 처음 렌더링됨. index의 초기값으로 0을 useState에 전달했으므로 [0, setIndex]가 반환. React는 0을 최신 state 값으로 기억.
2. state를 업데이트. 사용자가 버튼을 클릭하면 setIndex(index + 1)를 호출. index는 0이므로 setIndex(1)입니다. 이렇게 하면 React는 이제 index가 1임을 기억하고 다음 렌더링을 촉발.
3. 컴포넌트가 두 번째로 렌더링됨. React는 여전히 useState(0)을 보지만, index를 1로 설정한 것을 기억하고 있기 때문에, 이번에는 [1, setIndex]를 반환.
4. 이런 식으로 계속 진행

### 컴포넌트에 여러 state 변수 지정하기

서로 연관이 없는 경우 여러 개의 state 변수를 갖는 것이 좋음. 그러나 두 개의 state 변수를 자주 함께 변경하는 경우에는 두 변수를 하나로 합치는 것이 더 좋을 수 있음. 예를 들어, 필드가 많은 폼의 경우 필드별로 state 변수를 사용하는 것보다 객체를 값으로 하는 하나의 state 변수를 사용하는 것이 더 편리할 것.

- 참고: React는 어떤 state를 반환할지 어떻게 알 수 있을까요?

  - useState 호출이 어떤 state 변수를 참조하는지에 대한 정보를 받지 못한다는 것을 눈치챘을 것.
  - useState에 전달되는 “식별자”가 없는데 어떤 state 변수를 반환할지 어떻게 알 수 있을까?
  - 함수를 파싱하는 것과 같은 마법에 의존할까? 대답은 ‘no’
  - **대신 간결한 구문을 구현하기 위해 훅은 동일한 컴포넌트의 모든 렌더링에서 안정적인 호출 순서에 의존.**
  - 위의 규칙(“최상위 수준에서만 훅 호출”)을 따르면, 훅은 항상 같은 순서로 호출되기 때문에 실제로 잘 작동.
  - 내부적으로 React는 모든 컴포넌트에 대해 한 쌍의 state 배열을 가짐.
  - 또한 렌더링 전에 0으로 설정된 현재 쌍 인덱스를 유지.
  - useState를 호출할 때마다 React는 다음 state 쌍을 제공하고 인덱스를 증가시킴.
  - 자세한 내용은 [React Hook: 마법이 아니라 배열일 뿐입니다](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)에서 확인 가능.
  - React를 사용하기 위해 이 작동 방식을 이해해야할 필요는 없지만, 유용한 멘탈 모델이 될 수 있을 것.
  - useState의 내부동작을 간단히 작성한 코드 예제(대충 각 useState 호출 시, 알맞는 pair(state, setter)를 만들어 놓고, index(순서)에 의존하여 state를 식별한다는 것 같음. 그래서 최상위 수준에서만 훅 호출 규칙이 중요한거고...)
  - (내부적으로 각 컴포넌트에 대해 상태 배열을 관리하고, useState 호출 시 현재 인덱스를 기반으로 해당 상태 쌍을 제공하며 인덱스를 증가시키는 방식으로, 호출 순서에 따라 상태를 식별하고 관리)

    ```javascript
        let componentHooks = [];
        let currentHookIndex = 0;

        // How useState works inside React (simplified).
        function useState(initialState) {
        let pair = componentHooks[currentHookIndex];
        if (pair) {
            // This is not the first render,
            // so the state pair already exists.
            // Return it and prepare for next Hook call.
            currentHookIndex++;
            return pair;
        }

        // This is the first time we're rendering,
        // so create a state pair and store it.
        pair = [initialState, setState];

        function setState(nextState) {
            // When the user requests a state change,
            // put the new value into the pair.
            pair[0] = nextState;
            updateDOM();
        }

        // Store the pair for future renders
        // and prepare for the next Hook call.
        componentHooks[currentHookIndex] = pair;
        currentHookIndex++;
        return pair;
        }

        function Gallery() {
        // Each useState() call will get the next pair.
        const [index, setIndex] = useState(0);
        const [showMore, setShowMore] = useState(false);

        function handleNextClick() {
            setIndex(index + 1);
        }

        function handleMoreClick() {
            setShowMore(!showMore);
        }

        let sculpture = sculptureList[index];
        // This example doesn't use React, so
        // return an output object instead of JSX.
        return {
            onNextClick: handleNextClick,
            onMoreClick: handleMoreClick,
            header: `${sculpture.name} by ${sculpture.artist}`,
            counter: `${index + 1} of ${sculptureList.length}`,
            more: `${showMore ? 'Hide' : 'Show'} details`,
            description: showMore ? sculpture.description : null,
            imageSrc: sculpture.url,
            imageAlt: sculpture.alt
        };
        }

        function updateDOM() {
        // Reset the current Hook index
        // before rendering the component.
        currentHookIndex = 0;
        let output = Gallery();

        // Update the DOM to match the output.
        // This is the part React does for you.
        nextButton.onclick = output.onNextClick;
        header.textContent = output.header;
        moreButton.onclick = output.onMoreClick;
        moreButton.textContent = output.more;
        image.src = output.imageSrc;
        image.alt = output.imageAlt;
        if (output.description !== null) {
            description.textContent = output.description;
            description.style.display = '';
        } else {
            description.style.display = 'none';
        }
        }

        let nextButton = document.getElementById('nextButton');
        let header = document.getElementById('header');
        let moreButton = document.getElementById('moreButton');
        let description = document.getElementById('description');
        let image = document.getElementById('image');
        let sculptureList = [{
        name: 'Homenaje a la Neurocirugía',
        artist: 'Marta Colvin Andrade',
        description: 'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
        url: 'https://i.imgur.com/Mx7dA2Y.jpg',
        alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.'
        }, {
        name: 'Floralis Genérica',
        artist: 'Eduardo Catalano',
        description: 'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
        url: 'https://i.imgur.com/ZF6s192m.jpg',
        alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.'
        }, {
        name: 'Eternal Presence',
        artist: 'John Woodrow Wilson',
        description: 'Wilson was known for his preoccupation with equality, social justice, as well as the essential and spiritual qualities of humankind. This massive (7ft. or 2,13m) bronze represents what he described as "a symbolic Black presence infused with a sense of universal humanity."',
        url: 'https://i.imgur.com/aTtVpES.jpg',
        alt: 'The sculpture depicting a human head seems ever-present and solemn. It radiates calm and serenity.'
        }, {
        name: 'Moai',
        artist: 'Unknown Artist',
        description: 'Located on the Easter Island, there are 1,000 moai, or extant monumental statues, created by the early Rapa Nui people, which some believe represented deified ancestors.',
        url: 'https://i.imgur.com/RCwLEoQm.jpg',
        alt: 'Three monumental stone busts with the heads that are disproportionately large with somber faces.'
        }, {
        name: 'Blue Nana',
        artist: 'Niki de Saint Phalle',
        description: 'The Nanas are triumphant creatures, symbols of femininity and maternity. Initially, Saint Phalle used fabric and found objects for the Nanas, and later on introduced polyester to achieve a more vibrant effect.',
        url: 'https://i.imgur.com/Sd1AgUOm.jpg',
        alt: 'A large mosaic sculpture of a whimsical dancing female figure in a colorful costume emanating joy.'
        }, {
        name: 'Ultimate Form',
        artist: 'Barbara Hepworth',
        description: 'This abstract bronze sculpture is a part of The Family of Man series located at Yorkshire Sculpture Park. Hepworth chose not to create literal representations of the world but developed abstract forms inspired by people and landscapes.',
        url: 'https://i.imgur.com/2heNQDcm.jpg',
        alt: 'A tall sculpture made of three elements stacked on each other reminding of a human figure.'
        }, {
        name: 'Cavaliere',
        artist: 'Lamidi Olonade Fakeye',
        description: "Descended from four generations of woodcarvers, Fakeye's work blended traditional and contemporary Yoruba themes.",
        url: 'https://i.imgur.com/wIdGuZwm.png',
        alt: 'An intricate wood sculpture of a warrior with a focused face on a horse adorned with patterns.'
        }, {
        name: 'Big Bellies',
        artist: 'Alina Szapocznikow',
        description: "Szapocznikow is known for her sculptures of the fragmented body as a metaphor for the fragility and impermanence of youth and beauty. This sculpture depicts two very realistic large bellies stacked on top of each other, each around five feet (1,5m) tall.",
        url: 'https://i.imgur.com/AlHTAdDm.jpg',
        alt: 'The sculpture reminds a cascade of folds, quite different from bellies in classical sculptures.'
        }, {
        name: 'Terracotta Army',
        artist: 'Unknown Artist',
        description: 'The Terracotta Army is a collection of terracotta sculptures depicting the armies of Qin Shi Huang, the first Emperor of China. The army consisted of more than 8,000 soldiers, 130 chariots with 520 horses, and 150 cavalry horses.',
        url: 'https://i.imgur.com/HMFmH6m.jpg',
        alt: '12 terracotta sculptures of solemn warriors, each with a unique facial expression and armor.'
        }, {
        name: 'Lunar Landscape',
        artist: 'Louise Nevelson',
        description: 'Nevelson was known for scavenging objects from New York City debris, which she would later assemble into monumental constructions. In this one, she used disparate parts like a bedpost, juggling pin, and seat fragment, nailing and gluing them into boxes that reflect the influence of Cubism’s geometric abstraction of space and form.',
        url: 'https://i.imgur.com/rN7hY6om.jpg',
        alt: 'A black matte sculpture where the individual elements are initially indistinguishable.'
        }, {
        name: 'Aureole',
        artist: 'Ranjani Shettar',
        description: 'Shettar merges the traditional and the modern, the natural and the industrial. Her art focuses on the relationship between man and nature. Her work was described as compelling both abstractly and figuratively, gravity defying, and a "fine synthesis of unlikely materials."',
        url: 'https://i.imgur.com/okTpbHhm.jpg',
        alt: 'A pale wire-like sculpture mounted on concrete wall and descending on the floor. It appears light.'
        }, {
        name: 'Hippos',
        artist: 'Taipei Zoo',
        description: 'The Taipei Zoo commissioned a Hippo Square featuring submerged hippos at play.',
        url: 'https://i.imgur.com/6o5Vuyu.jpg',
        alt: 'A group of bronze hippo sculptures emerging from the sett sidewalk as if they were swimming.'
        }];

        // Make UI match the initial state.
        updateDOM();`

    ```

### state는 격리되고 프라이빗합니다

동일한 컴포넌트를 두 군데에서 렌더링하면 각 사본은 완전히 격리된 state를 갖게된다. 이 중 하나를 변경해도 다른 컴포넌트에는 영향을 미치지 않는다. 이것이 바로 모듈 상단에 선언하는 일반 변수와 state의 차이점.

또한, props와 달리 state는 이를 선언하는 컴포넌트 외에는 완전히 비공개이며, 부모 컴포넌트는 이를 변경할 수 없음. 따라서 다른 컴포넌트에 영향을 주지 않고 state를 추가하거나 제거할 수 있음.

동일한 컴포넌트의 동일한 각각의 state를 동기화 시키려면? React에서 이를 수행하는 올바른 방법은 자식 컴포넌트에서 state를 제거하고 가장 가까운 공유 부모 컴포넌트에 추가하는 것.
