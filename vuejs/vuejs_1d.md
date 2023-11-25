해당 내용은 `Vue3 완벽 마스터: 기초부터 실전까지`를 바탕으로 작성했습니다.

---
#### 개발환경 구성
- chrome
- visual studio code
- vscode plugins
  - Indent-Rainbow: `Tab` 영역을 컬러별로 다르게 표시
  - Auto Rename Tag: `HTML Tag`에서 태그 이름을 바꾸면 쌍을 이루는 태그명도 같이 바꿔준다.
  - CSS Peek: `HTML` 문서에서 정의된 `CSS`를 찾을 수 있도록 도와준다(`ctrl + 클릭`)
  - HTML to CSS autocompletion: `HTML` 문서에서 선언된 class명을 `.css`파일에서 입력할 때 자동완성 기능을 제공
  - HTML CSS Support: `HTML` 문서에서 `CSS` 자동완성을 이용할 수 있다.
  - Live Server: `HTML` 파일 수정 시 새로고침 없이 바로바로 즉각 적용되도록 도와준다.
  - ESLint: 코드 검사기로써 에러가 있는지 검사해주는 도구
  - Vue.js devtools
  
- Vue.js를 위한 VSCode Plugins
  - Volar
  - Vue VSCode Snippets

---
#### Vue란?
 > `Vue`는 UI 개발을 위한 자바스크립트 프레임워크이다. `HTML`, `CSS`, `JavaScript` 기반으로 단순 하거나 복잡한 사용자 인터페이스를 효율적으로 개발하는데 도움을 준다.

*Vue의 두 가지 핵심 기능*
- 선언적 렌더링: `Vue`는 템플릿 구문(`{{ 데이터 }}`)를 활용하여 데이터를 선언적으로 출력할 수 있다.
- 반응성: `Vue`는 JavaScript 상태 변경을 자동으로 추적하고 변경이 발생하면 DOM을 효율적으로 업데이트한다.

**바인딩(v-bind)**  
:템플릿 구문 이외에도 다음과 같은 방법으로 엘리먼트 속성에 데이터를 바인딩(연결)할 수 있다.
```html
 <input type="text" v-bind:placeholder="message">
``` 
위의 예제에서 `v-bind` 속성은 데이터 속성에 바인딩할 때 사용하는 특수 속성이다.
바인딩된 `DOM`은 `placeholder` 속성을 Vue 인스턴스의 `message` 속성으로 최신상태를 유지한다.
이렇게 v- 접두어가 붙은 특수 속성을 **디렉티브**라고 한다.

**이벤트 핸들링(v-on)**
사용자가 앱과 상호작용할 수 있게 하기 위해 `v-on` 디렉티브를 사용하여 Vue 인스턴스의 메소드를 호출하는 이벤트 리스너를 추가할 수 있다.

```html
<button v-on:click="reverseMessage">reverseMessage</button>

<script>
 const reverseMessage = () => {
	this.message = this.message
	.split('')
	.reverse()
	.join('')
 }
</script>
```
이 방법은 직접적으로 `DOM`을 건드리지 않고 앱의 상태만 업데이트한다.

**양방향 바인딩(v-model)**  
`Vue`는 양식의 입력과 앱의 상태를 양방향으로 바인딩하는 `v-model` 디렉티브를 제공한다.

```html
<p>{{ bindingMessage }}</p>
<input type="text" v-model="bindingMessage" />
```

**조건문**  
엘리먼트 표시여부는 `v-if` 디렉티브로 제어할 수 있다.

```html
<p v-if="visible">보이나요?</p>
<button type="button" v-on:click="visible= true">visible</button>
```

**반복문**
`v-for`는 배열에서 데이터를 가져와 아이템 목록을 표시하는데 사용할 수 있다.
```html
<ul>
    <li v-for="todo in todos">{{ todo }}</li>
</ul>
```
---
**컴포넌트란?**
> 자바스크립트에서 재사용할 수 있도록 코드를 분리한 파일을 `모듈`이라고 하는데 `Vue`에서도 마찬가지로 UI를 재사용할 수 있도록 정의한 것을 **컴포넌트**라고 한다.

**컴포넌트를 사용하는 이유**
- 컴포넌트를 사용하면 `UI`를 `재사용`할 수 있다.
- 컴포넌트를 사용하여 `UI`를 독립적으로 나눔으로써 코드를 클린하게 할 수 있다.

컴포넌트를 사용하려면 `정의` -> `등록` -> `사용` 순서로 진행하면 된다.

**컴포넌트 정의 방법**  
컴포넌트를 어떤 방법으로 정의하느냐에 따라 `문자열 템플릿`과 `Single File Component` 두 가지 방법이 있다.

**Single File Component(SFC)**  
`Vue.js`는 `Webpack`, `Browserify`, `Vite`와 같은 빌드 도구를 활용하여 `.vue` 확장자를 가진 `SFC`를 사용한다.  
`SFC`는 `template`, `script`, `style` 크게 세 가지로 구성되어 있다.

**컴포넌트 등록**
어디에서 사용하냐에 따라 `전역 등록`과 `지역 등록`으로 나누어진다.

- 전역등록(Global)  
  `app.component`를 이용해서 컴포넌트를 등록하면, 컴포넌트는 애플리케이션 전역등록되어 모든 컴포넌트 인스턴스의 템플릿 내부에서 사용할 수 있다.
 ```
 const app = createApp({...});
 app.component('BookComponent', BookComponent);
 ```

- 지역등록(Local)
  `Webpack`과 같은 빌드 시스템을 사용하는 경우, 컴포넌트를 전역 등록하게 되면 컴포넌트를 사용하지 않더라도 계속해서 최종 빌드에 해당 컴포넌트가 포함된다.   
  이는 사용자가 다운로드하는 자바스크립트 파일의 크기를 불필요하게 증가시킨다.

**컴포넌트 사용**
컴포넌트는 `template`에서 사용할 수 있다.  
```html
<BookComponent></BookComponent> // PascalCase
<book-component></book-component> // kebab-cased
```
- `PascalCase`로 등록된 컴포넌트는 `PascalCase, kebab-cased` 둘다 사용 가능하다.
- `kebab-cased`로 등록된 컴포넌트는 `kebab-cased`로만 사용 가능하다.
---
**컴포넌트 네이밍룰**  

- 컴포넌트를 사용할때 `PascalCased`를 권장.  
- `PascalCased` 이름은 유효한 JavaScript 식별자이다. 이렇게 하면 `JavaScript`에서 구성요소를 더 쉽게 가져오고 등록할 수 있다.
- `<PascalCase />`는 템플릿의 기본 HTML 요소가 아닌 Vue 구성요소라는 것을 더 분명하게 만든다.

**컴포넌트 시스템**  

컴포넌트 시스템은 `Vue`의 또 다른 중요한 개념이다.    
작고 독립적이며 `재사용`할 수 있는 컴포넌트로 구성된 대규모 애플리케이션을 구축할 수 있게 해주는 추상적인 개념이다.  
생각해보면 거의 모든 유형의 애플리케이션 인터페이스를 컴포넌트 트리로 추상화 할 수 있다.

---
#### 프로젝트 구성
Vue 설치
- Vite(비트): `Vite`는 `SFC(Single File Component)`를 지원하고 매우 가볍고 빠른 빌드 도구이다.   
개발 서버를 구동할 때 매우 빠르다. 소스코드의 변경이 일어났을 때 전체 모듈을 번들링하는게 아니라 변경된 모듈만 교체하기 때문에 개발을 더욱 더 빠르게 진행할 수 있다.

```
> npm init vue@latest
> cd {project name}
> npm install
> npm run dev
```

vite.config.js
: `Vite` 명령어로 dev 서버를 실행할 때 프로젝트 루트의 `vite.config.js` 파일 확인을 시도한다.  
그리고 내부에 설정된 값을 참고한다.  

* **alias**: 파일 시스템의 경로에 `별칭`을 만들때 사용한다. 미리 설정된 '`@`' 기호를 통해 './src' 디렉토리에 절대경로로 쉽게 접근할 수 있다.

**package.json**  
`npm`으로 관리하기 위한 프로젝트 정보를 갖고 있는 파일

**ESLint, Prettier**
- ESLint: `ESLint`는 코드 검사기로 코드에 `에러`가 있는지 검사해주는 도구이다.
- Prettier: `Prettier`는 코드 포매터로 코드를 일관성있고 예쁘게 `정렬`해주는 도구이다.

.eslintrc.cjs
```
/* eslint-env node */
require("@rushstack/eslint-patch/modern-module-resolution");

module.exports = {
  "root": true,
  "extends": [
    "plugin:vue/vue3-essential",
    "eslint:recommended",
    "@vue/eslint-config-prettier"
  ],
  "env": {
    "vue/setup-compiler-macros": true
  }
}
```
---
#### Vue2 -> Vue3 차이점

**Options API vs Composition API**

**Composition API란?**  
`Composition API`는 옵션(data, methods, ..)을 선언하는 대신 가져온 함수(ref, onMounted, ...)를 사용하여 Vue 컴포넌트를 작성할 수 있는 `API 세트`를 말한다.

**등장배경**  
`Vue2`에서 `Options API`로 작성된 코드를 보면 동일한 논리적 관심사를 처리하는 코드가 파일의 다른 부분에 분산되어 있어 코드를 분석하기가 매우 힘들었다.

만약 코드가 더 복잡하고 길어질 경우 파일을 위아래로 스크롤해야 하기 때문에 더 이해하기 힘든 상황이 온다.

`Composition API`에서는 동일한 논리적 관심사 코드가 그룹화되어 코드를 분석하기도 쉽고 `유지보수`가 `용이`해진다. 또한 논리적 관심사 코드를 외부 유틸 파일로 추출하기가 쉽다.

**코드 재사용성**  
`Composition API`의 가장 큰 장점은 `Composable` 함수의 형태로 로직의 재사용이 가능하다는 것이다.

**Composition API로 기존 모든 사용 사례를 커버?**  
`Options API` 구성요소의 setup() 옵션을 통해 Composition API를 사용할 수 있다.

---
#### Composition API
- `반응형 API(Reactivity API)`
  예를들어 `ref(), reactive()`와 같은 API를 사용하여 reactive state, computed, state, watchers와 같은 것들을 만들수 있다.
- 라이프 사이클 훅  
  예를들어 `onMounted(), onUnmounted()`와 같은 API를 사용하여 프로그래밍 방식으로 컴포넌트 라이프사이클에 접근할 수 있다.
  크게 `create -> mount -> update -> destroy`순으로 진행된다.
- 종속성 주입(DI)  
  `provide()와 inject()`는 Reactivity API를 사용하는 동안 Vue의 의존성 주입 시스템을 활용할 수 있게 해준다.

---
#### Setup hook
**Setup**  
`setup()` 함수는 Composition API 사용을 위한 진입점 역할은 한다.
setup 함수가 실행되는 시점은 컴포넌트 인스턴스가 생성되기 전에 실행된다.

**기본사용**  
반응형 API를 사용하여 반응형 상태를 선언하고 setup()에서 객체를 반환하여 `<template>`에 노출할 수 있다. 반환된 객체의 속성은 구성 요소 인스턴스에서도 사용할 수 있다.

**Props 접근**  
`setup`함수의 첫번째 매개변수는 `props`이다. `props`는 반응형 객체이다.
`props` 객체를 구조 분해할당을 하면 반응성을 잃게 된다.   
따라서 항상 props.xxx 형식으로 `props`에 접근하는 것이 좋다.

만약 `props`의 반응성을 유지하면서 구조분해할당을 해야한다면 `toRefs() 및 toRef()` 유틸리티 API를 사용하여 이를 수행할 수 있다.

 ```
 setup(props) {
	const {title} = toRefs(props);
	const title = toRef(props, 'title');
 }
 ```

**Setup Context**  
`setup` 함수에 전달된 두 번째 매개변수는 `Setup Context` 객체이다. 컨텍스트 객체는 setup 함수내에서 유용하게 사용할 수 있는 속성을 갖고 있다.

 ```
 setup(props, context) {
	context.attrs // 속성
	context.slots // 슬롯
	context.emit // 이벤트 발생
	context.expose // Public한 속성
 }
 ```
컨텍스트 객체는 반응형이 아니기 때문에 안전하게 구조 분해 할당을 할 수 있다.

**attrs, slots**  
- attrs와 slots는 컴포넌트 자체가 업데이트될때 항상 업데이트되는 상태 저장 객체이다.
- 이러한 것들은 구조 분해할당을 피해야하며 항상 속성을 attrs.x 또는 slot.x로 접근해야 한다.
또한 props와 달리 attrs, slots의 속성은 반응형이지 않다.
- attrs 또는 slots 변경에 따라 다른 작업을 하려고 하는 경우 `onBeforeUpdate` 라이프사이클 훅 내에서 수행할 수 있다.
 
---
#### 템플릿 문법
Vue는 템플릿 문법을 사용하여 렌더링된 DOM을 컴포넌트의 인스턴스 데이터에 선언적으로 바인딩할 수 있다.

**텍스트 보간법**  
데이터 바인딩의 가장 기본형태는 `{{ data }}`(이중 중괄호, 콧수염 괄호)를 사용하는 것이다.  
이중 중괄호를 사용하면 해당 문법은 컴포넌트 인스턴스의 `message`값으로 대체된다.

`v-once` 디렉티브를 사용하여 데이터가 변경되어도 갱신되지 않는 일회성 보간을 수행할 수 있다.
```
<p v-once>문자열: {{ message }}</p>
```

**HTML(v-html)**  
이중 중괄호는 데이터를 `HTML`이 아닌 일반 텍스트로 해석한다. 실제 `HTML`을 출력하려면 `v-html` 디렉티브를 사용해야 한다.

**속성 바인딩(v-bind)**  
이중 중괄호는 `HTML` 속성에 사용할 수 없다. 대신 `v-bind` 디렉티브를 사용하면 된다.

```
<div v-bind:title="dynamicTitle">마우스를 올려보세요.</div>

<div :title="dynamicTitle">마우스를 올려주세요.</div>
```
다음과 같이 단축문법을 사용해서 표현할 수 있다.

---
#### 반응형 기초
**반응형 상태 선언하기**  
JavaScript 객체에서 반응형 상태를 생성하기 위해서는 `reactive()` 함수를 사용할 수 있다.

```
import { reactive } from 'vue';

const state = reactive({ count : 0})
```

`reactive()` 함수는 객체타입에만 동작한다. 그래서 `기본타입(primitive type)`을 반응형으로 만들고자할때에는 `ref` 메소드를 사용할 수 있다.

```
import { ref } from 'vue'
const count = ref(0);
```
`ref` 메서드는 변이가능한 객체를 반환한다. 이 객체안에는 `value`라는 하나의 속성만 포함한다. value값은 ref() 메서드에 매개변수로 받은 값을 가지고 있다. 이 객체는 내부의 value 값에 대한 반응형 참조역할을 한다.

**템플릿에서 사용**  
템플릿에서 사용할 때는 자동으로 내부값을(`value`) 풀어내기 때문에 `.value`를 추가할 필요없이 사용할 수 있다.

반응형 상태 구조 분해하기
큰 반응형 객체의 몇몇 속성을 사용하기를 원할 때, 원하는 속성을 얻기 위해 ES6 구조 분해 할당을 사용하는 것은 매우 일반적이다.

구조분해로 두 속성은 반응형을 잃게 된다. 이런 경우 반응형 객체를 일련의 `ref`들로 변환해야 한다.
`toRefs, toRef`를 사용하면 반응형 객체의 속성과 동기화 된다. 그래서 원본 속성을 변경하면 `ref` 객체가 업데이트 되고 그 반대의 경우도 마찬가지이다.

**readonly를 이용하여 반응형 객체의 변경 방지**  
때때로 반응형 객체의 변화를 추적하기 원하지만, 특정 부분에서는 변화를 막기를 원하기도 한다. 예를들어 `Provide/Inject`로 주입된 반응형 객체를 갖고 있을 때, 우리는 그것이 주입된 곳에서는 해당 객체가 변이되는걸 막고자할것이다. 이렇게 하려면 원래 객체에 대한 읽기전용 프록시를 생성해야한다.

---
#### Computed
템플릿 문법은 간단히 사용하면 매우 편리하지만 템플릿 표현식 내 코드가 길어지는 경우 가독성이 떨어지고, 유지보수가 어려워질수 있다.

```html
<span>{{ teacher.lectures.length > 0 ? 'Yes' : 'No' }} </span>
```
이러한 코드를 여러곳에서 반복적으로 사용해야하는 경우 비효율적이다.
이때 사용하면 되는 것이 `계산된 속성(computed property)`이다.

```html
const hasLecture = computed(() => {
	return teacher.lectures.length > 0 ? 'Yes' : 'No'
})

<span>{{ hasLecture }}</span>
```

**Computed vs Method**  
메서드를 활용해도 `computed`와 동일한 효과를 얻을 수 있다. 하지만 차이점은 결과가 캐시된다는 것이다.
`computed` 내 반응형 데이터가 변경된 경우에만 다시 계산된다.
- computed는 캐시된다.
- method는 파라미터가 올수 있다.
- 컴포넌트 렌더링시 computed는 비용이 적게든다.

**Writable Computed**  
`computed`는 기본적으로 geter 전용이다. 계산된 속성에 새 값을 할당하려고하면 런타임 경고가 표시된다.
새로운 계산된 속성이 필요한 경우에는 `getter`와 `setter`를 모두 제공하여 속성을 만들 수 있다.

```
const firstName = ref('홍');
const lastName = ref('길동');

const fullName = computed({
    get() {
        return firstName.value + ' ' + lastName.value;
    },
    set(newValue) {
        [firstName.value, lastName.value] = newValue.split(' ');
    },
}

fullName.value = '안녕 하세요';
```
