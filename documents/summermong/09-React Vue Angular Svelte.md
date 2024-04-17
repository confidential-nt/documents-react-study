# React와 Vue, Angular, Svelte

프론트엔드 개발자가 사용하는 JS 라이브러리 또는 프레임워크들이 있다.
React와 Vue, Angular와 Svelte에 대해 간단하게 알아보자!

## Vue

- 웹 애플리케이션의 UI를 위한 프로그레시브 JS 프레임워크
- 표준 HTML, CSS, JS를 기반으로 구축되는 컴포넌트 기반 프로그래밍 모델

```
import { createApp, ref } from 'vue'

createApp({
  setup() {
    return {
      count: ref(0)
    }
  }
}).mount('#app')
```

```
<div id="app">
  <button @click="count++">
    숫자 세기: {{ count }}
  </button>
</div>
```

- 표준 HTML을 템플릿 문법으로 확장해 JS 상태를 기반으로 화면에 출력될 HTML을 선언적으로 작성함
- JS 상태 변경을 추적하고 변경이 발생하면 DOM을 자동으로 업데이트함
- HTML과 유사한 싱글 파일 컴포넌트 형식으로 Vue 컴포넌트를 작성함

```
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

- Vue 컴포넌트는 옵션 API, 컴포지션 API 2가지 스타일로 작성할 수 있음
- 옵션 API: 객체를 사용해 컴포넌트 로직 정의 (클래스형 같은 느낌)
- 컴포지션 API: import한 API 함수들로 컴포넌트 로직 정의 (함수형 같은 느낌)
- Vue.js 2.0부터 가상 DOM을 사용함

- 양방향, 단방향 데이터 바인딩을 지원함
- 양방향: v-model 디렉티브를 사용해 입력 요소와 데이터를 양방향으로 바인딩
- 단방향: v-bind 디렉티브를 사용해 데이터의 변경이 DOM에 반영되지만 DOM의 변경은 데이터에 영향을 주지 않음 (JS -> HTML)

- 경량 프레임워크라 가볍고 유연함
- 다른 라이브러리/프레임워크와 쉽게 호환 및 통합됨
- Evan You 개인이 개발한 프레임워크이고, 비교적 커뮤니티가 작음

## Angular

- 구글에서 개발한 TypeScript 기반 웹 프레임워크
- 프로젝트의 생성, 테스팅, 빌드, 배포까지 모든 기능을 제공함 (무거움)
- 러닝 커브가 비교적 가파른 편
- 업데이트가 잦음

- MVVM 패턴 기반
- 데이터(모델)와 UI(뷰)를 분리하고, 그 사이에 뷰 모델을 둬 양방향 데이터 바인딩을 지원함
- 뷰 모델은 뷰에서 필요한 데이터와 UI 로직을 갖고 뷰에 대한 상태와 행동을 관리함
- 모델에서 가져온 데이터를 뷰에 바인딩하고, 유저의 입력을 처리해 모델에 전달함

- React, Vue와 달리 가상 DOM을 사용하지 않음
- 변경 감지(Change Detection) 메커니즘으로 실제 DOM을 조작/업데이트함
- 비동기 작업을 추적하고 모니터링하는 라이브러리 Zone.js를 사용함
- Observable 객체는 RxJS 라이브러리에서 사용하는 개념으로 비동기적인 데이터 스트림을 다루는데 사용됨
- Zone.js는 Observable이 발생 시키는 비동기 작업을 추적, 변경을 감지해 업데이트를 함
- 둘은 직접적으로 관련이 있지는 않지만 종종 상호작용함

```
// greeting.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class GreetingService {
  constructor() { }

  getGreeting(): string {
    return 'Hello, Angular!';
  }
}
```

```
// app.component.ts
import { Component } from '@angular/core';
import { GreetingService } from './greeting.service';

@Component({
  selector: 'app-root',
  template: `
    <div>
      <h1>{{ greeting }}</h1>
    </div>
  `,
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  greeting: string;

  constructor(private greetingService: GreetingService) {
    this.greeting = this.greetingService.getGreeting();
  }
}
```

- 종속성 주입을 사용해 모듈성이 향상되고 유지보수가 용이함
- 아래 코드에서 GrettingService는 종속성 중 하나인데, getGreeting 메서드를 제공해 인사말을 반환함.
- AppComponent는 GreetingService를 주입해 해당 서비스의 인스턴스를 사용, 화면에 표시할 인사말을 가져옴

## Svelte

- 오픈소스 웹 프레임 워크
- 가상 DOM을 사용하지 않고 빌드 단계에서 구성 요소를 컴파일함
- 런타임에서 해석하는 게 아니라 빌드 타임에서 컴포넌트를 바닐라 JS 번들로 컴파일함
- 속도가 빠르고 따로 라이브러리를 배포할 필요가 없어 간단함
- 가상 DOM을 사용하는 React, Vue에서는 실제 DOM을 제어하지 않는데, Svelte에서는 document.querySelector 같은 DOM API로 바로 접근함

```
<script>
  import { onMount } from 'svelte';

  let inputValue = '';
  let outputValue = '';

  function handleChange(event) {
    inputValue = event.target.value;
  }

  onMount(() => {
    const inputElement = document.getElementById('input');
    const outputElement = document.getElementById('output');

    function updateOutput() {
      outputValue = inputElement.value.toUpperCase();
    }

    inputElement.addEventListener('input', updateOutput);

    return () => {
      inputElement.removeEventListener('input', updateOutput);
    };
  });
</script>

<input id="input" type="text" bind:value={inputValue} on:input={handleChange}>
<p id="output">Output: {outputValue}</p>
```

## 그래서 비교하면

| React                            | Vue                             | Angular                              | Svelte                                |
| -------------------------------- | ------------------------------- | ------------------------------------ | ------------------------------------- |
| 페이스북산 라이브러리            | Evan you산 프레임워크           | 구글산 TS기반 프레임워크             | Rich Harris산 컴파일러기반 프레임워크 |
| JSX 문법                         | 템플릿 문법과 옵션/컴포지션 API | TypeScript                           | 웹 표준에 가까운 HCJ                  |
| 가상 DOM                         | 가상 DOM                        | 변경 감지 매커니즘으로 실제 DOM 조작 | 실제 DOM 조작                         |
| 컴포넌트 기반 단방향 데이터 흐름 | 양방향 데이터 바인딩            | MVVM패턴의 양방향 데이터 바인딩      | 단방향 데이터 흐름                    |

---
