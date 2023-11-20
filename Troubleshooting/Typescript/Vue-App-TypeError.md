# Typescript + Vue 에서 App.vue를 인식하지 못했을 때

타입스크립트에서 App.vue 의 타입을 추론할 수 없어 에러가 뜬다.

```ts
import { createApp } from 'vue';

import './style.css';

import App from './App.vue'; // Error: Cannot find module './App.vue' or its corresponding type declarations

createApp(App).mount('#app');
```

TypeScript Vue Plugin (Volar) 해당 확장 프로그램을 vscode에 추가 설치 해줬다.

물론 타입 정의 파일을 생성해서 타입 에러를 제거할 수도 있다고 한다:

```ts
// 파일명: shims-vue.d.ts
declare module '*.vue' {
  import { ComponentOptions } from 'vue';
  const componentOptions: ComponentOptions;
  export default componentOptions;
}
```
