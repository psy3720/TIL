해당 내용은 `Vue3 완벽 마스터: 기초부터 실전까지`를 바탕으로 작성했습니다.

---
#### script setup
`&lt;script setup&gt;`은 Single-File Component 내에서 Composition API를 사용하기 위한 syntactic sugar(문법적 설탕)이다.

SFC와 Composition API를 사용하는 경우 `&lt;script setup&gt;`을 사용하는 것을 권장한다.
 - 간결한 문법으로 상용구를 줄일 수 있다.
 - 타입스크립트를 사용한 `props`와 `emits` 선언 가능
 - 런타임 성능의 향상(템플릿이 `setup` 스크립트와 같은 스코프에 있는 render 함수로 컴파일 되므로 프록시가 필요 없음)
 - 더 뛰어난 IDE 타입추론 성능(language 서버가 코드로부터 타입을 추론해내는데 비용이 덜 든다)
 
##### 기본 문법
`&lt;script&gt;` 블록에 setup 속성을 추가해서 시작할 수 있다.
```
<script setup></script> 
```
내부 코드는 컴포넌트의 `setup()` 함수 안의 코드로 컴파일 된다.
컴포넌트를 처음 가져올 때 한번만 실행되는 일반 `&lt;script&gt;`와 달리, `<script setup>`는 컴포넌트의 인스턴스가 생성될 때마다 
`<script setup>` 내부 코드가 실행된다.

**Top-level에 선언**
- `<script setup>` 내부 최상위에 선언된 변수, 함수, import는 `<template>`에서 직접 사용할 수 있다.
- import된 자원(Component, Utils 등)도 동일한 방식으로 `<template>`에서 직접 사용할 수 있다.

```html
<script setup>
const msg = 'Hello!'

function log() {
  console.log(msg)
}
</script>

<template>
  <div @click="log">{{ msg }}</div>
	<HelloComponent></HelloComponent>
</template>
```

```html
<script setup>
import HelloComponent from './components/HelloComponent.vue'

</script>

<template>
	<HelloComponent></HelloComponent>
</template>
```

#### Reactivity
Reactivity APIs(ref, reactivity, computed, watch 등)를 `<script setup>` 안에서 생성하면 `<template>`에서 직접적으로 사용가능하다.

```html
<template>
	<p>{{ message }}</p>
</template>
<script setup>
import { ref } from 'vue';

const message = ref('Hello World!');
</script>
```

#### defineProps() & defineEmits()
defineProps()와 defineEmits() APIs를 `<script setup>` 내에 선언하여 `props`와 `emits`을 사용할 수 있다.

```html
<script setup>
const props = defineProps({
  foo: String
})

const emit = defineEmits(['change', 'delete'])
</script>
```
- defineProps와 defineEmits는 `<script setup>` 내부에서만 사용할 수 있는 컴파일러 매크로이다. 그렇기 때문에 import 할 필요가 없으면 `<script setup>`이 처리될 때 컴파일 된다.
- defineProps는 props 옵션과 동일한 값을 허용한다. 그리고 defineEmits는 emits 옵션과 동일한 값을 허용한다.
- defineProps와 defineEmits는 전달된 옵션을 기반으로 타입 추론을 제공한다.
- defineProps와 defineEmits에 전달된 옵션은 `setup()`에서 모듈 영역으로 호이스트된다.
  따라서 옵션은 `setup()` 영역에 선언된 지역 변수를 참조할 수 없다. 만약 그렇게 하면 컴파일 오류가 발생한다.
  하지만 import된 옵션은 사용할 수 있다. 왜냐하면 import도 모듈 영역으로 호이스트 되기 때문이다.

#### defineExpose()
`<script setup>`을 사용하는 컴포넌트는 기본적으로 Template Refs나 $parent와 같이 컴포넌트간 통신이 닫혀있다.
`<script setup>`을 사용하는 컴포넌트의 내부 데이터나 메서드르 명시적으로 노출하려면 `defineExpose()` 컴파일러 매크로를 사용할 수 있다.

```html
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
``` 
expose는 일반 `<script>`에서도 사용할 수 있다.
```
export default {
setup(props, context) {
// Expose public properties (Function)
console.log(context.expose)
}
}
```

#### useSlots() & useAttrs()
slots과 attrs는 `<template>` 내부에서 $slots와 $attrs로 직접 접근해서 사용할 수 있다. 
만약 `<script setup>` 내부에서 slots과 attrs를 사용하고 싶다면 각각 useSlots(), useAttrs() `helper` 메서드를 사용할 수 있다.

```html
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs() // fallthrough 속성 접근하기
</script>
```

```
export default {
    setup(props, context) {
    // Attributes (Non-reactive object, equivalent to $attrs)
    console.log(context.attrs)
    
        // Slots (Non-reactive object, equivalent to $slots)
        console.log(context.slots)
    }
}
```

#### `<script>`와 `<script setup>` 함께 사용
`<script setup>`은 normal `<script>`와 함께 사용할 수 있다. 예를 들면 다음과 같은 경우에 normal `<script>`가 필요할 수 있다.
 - 예를 들어 `<script setup>`에서 표현할 수 없는 `inheritAttrs` 옵션이나 Plugin을 통해 활성화된 Custom 옵션을 사용하고자 할 때 normal `<script>`를 함께 선언한다.
 - named export를 선언 했을 때(예: export const data)
 - 한 번만 실행되어야 하는 로직이 있을 때

```
<script>
// 일반 스크립트, 모듈 범위에서 한 번만 실행
runSideEffectOnce()

// 옵션 선언
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// 각 인스턴스 생성시 setup() 범위에서 실행
</script>
```

#### Top-level await
`<script setup>` 내의 Top-level에서 `await`을 사용할 수 있다. 그리고 코드는 `async setup()` 이렇게 컴파일 된다.
```
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```

---
#### VueRouter란?
> 뷰 라우터는 Vue.js를 이용하여 싱글 페이지 애플리케이션(SPA)을 구현할 때 사용하는 Vue.js의 공식 라우터이다.

**라우터란?(Router)**  
> `라우터`라고 하면 일반적으로 네트워크간에 데이터를 전송하는 장치를 말한다. 뷰에서 말하는 라우터는 쉽게 말해서 URL에 따라 어떤 페이지를 보여줄지 매핑해주는 라이브러리라고 보면 된다.
예를들어 `/home` 경로로 요청이 들어왓을 때 `Home.vue` 컴포넌트를 화면에 렌더링해라라는 역할을 수행하는 라이브러리이다.
그리고 이때 `/home` -> `Home.vue` 이러한 매핑 정보를 `라우트`라고도 한다.

**라우트란?(Route)**  
어떤 URL에 대해 어떤 페이지를 표시해야 하는지에 대한 정보

**설치**
```
> npm install vue-router
```

**시작하기**  
Home.vue와 AboutView.vue라는 페이지용 컴포넌트를 만든 후 `/` 경로로 들어왔을 경우 `HomeView.vue` 페이지를 렌더링하고, `/about` 경로로 들어왓을 경우 `AboutView.vue` 페이지를 렌더링 하는 예시

**컴포넌트 생성**
 (생략)

**라우트정의**  
 URL 요청에 대해 어떤 페이지를 보여줄지에 대한 매핑정보를 정의
```
// src/router/index.js
import HomeView from '@/views/HomeView.vue';
import AboutView from '@/views/AboutView.vue';

const routes = [
    {
        path: '/',
        name: 'home',
        component: HomeView,
    },
    {
        path: '/about',
        name: 'about',
        component: AboutView,
    },
];

```
**라우터(router) 설정**
```
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router';
import HomeView from '@/views/HomeView.vue';
import AboutView from '@/views/AboutView.vue';

const routes = [
    {
        path: '/',
        name: 'home',
        component: HomeView,
    },
    {
        path: '/about',
        name: 'about',
        component: AboutView,
    },
];

const router = createRouter({
    history: createWebHistory('/'),
    routes,
});

export default router;
```

**설정한 라우터 객체를 Vue 인스턴스에 추가**
```
import { createApp } from 'vue';

import App from './App.vue';
import router from './router';

createApp(App).use(router).mount('#app');
```
`app.use(router)`를 호출함으로써 컴포넌트 내부에서 `$router`, `$route` 객체에 접근할 수 있다.

#### 네비게이션
뷰 라우터를 HTML과 JavaScript로 사용하는 방법  
**HTML**
```html
// src/App.vue
<script setup></script>

<template>
  <nav>
    <Routerlink to="/">Home</Routerlink>
    <span> | </span>
    <RouterLink to="/about">About</RouterLink>
  </nav>
  <main>
    <RouterView></RouterView>
  </main>
</template>
```
- `<RouterLink>` Vue Router에서는 페이지를 이동할 때는 일반 a 태그를 사용하는 대신 커스텀 컴포넌트인 `<RouterLink>`를 사용하여 다른 페이지 링크를 만들어야 한다.
이를 통해 Vue Router는 페이지를 리로딩하지 않고, URL에 매핑된 페이지를 렌더링할 수 있다.
- `<RouterView>`는 URL에 매핑된 컴포넌트를 화면에 표시한다.

**JavaScript**
위에서 router를 설정할때 `app.use(router)`를 호출했다. 이렇게 호출함으로써 모든 자식 컴포넌트에 `router`, `route` 같은 객체를 사용할 수 있다. 
그리고 이러한 객체를 페이지 이동 또는 현재 활성 라우트(경로 매핑)정보에 접근하는데 사용할 수 있다.

**router**  
  라우터 인스턴스로 JavaScript에서 다른 페이지로 이동할 수 잇다.
- Options API : `this.$router`
- Composition API : `useRouter()`
- template: `$router`

**route**  
  현재 활성 라우트 정보에 접근할 수 있다.(readonly)
- Options API: `this.$route`
- Composition API: `useRoute()`
- template: `$route`

```html
<!-- HomeView.vue -->
<script setup>
import { useRoute, useRouter } from 'vue-router';

const router = useRouter();
const route = useRoute();
console.log('route.name: ', route.name);
console.log('route.path: ', route.path);
const goAboutPage = () => router.push('/about');
</script>
<template>
  <h1>Home Page</h1>
  <button @click="goAboutPage">About 페이지로 이동</button>
</template>
```

```html
<!-- AboutView.vue -->
<script setup></script>
<template>
  <h1>About Page</h1>
  <ul>
    <li>$route.name: {{ $route.name }}</li>
    <li>$route.path: {{ $route.path }}</li>
  </ul>
  <button @click="$router.push('/')">Home 페이지로 이동</button>
</template>
```
---
#### 동적 라우트 매칭
주어진 패턴을 가진 라우트를 동일한 컴포넌트에 매핑해야 하는 경우가 자주 있다.  
예를 들어 사용자 목록(User List)은 `/users`와 같은 경로에 매핑되면 되지만 사용자 상세는 사용자 식별자 별로 같은 컴포넌트에 매핑되어야 한다.
이럴때 Vue Router에서는 경로에서 `동적 세그먼트`를 사용하여 해결할 수 있다. 이를 `param`이라고 한다.

```
const User = {
  template: '<div>User</div>',
}

const routes = [
  { path: '/users/:id', component: User },
]
```
- 동적 세그먼트는 `콜론(:)`으로 표시한다.
- 그리고 컴포넌트에서 동적 세그먼트의 값은 `$route.params` 필드로 접근할 수 있다.

```
const User = {
  template: '<div>User {{ $route.params.id }}</div>',
}
```
동일한 라우트에 여러 동적 세그먼트를 가질 수 있으며, `$route.params` 필드에 매핑된다.

**query, hash**  
`$route.params` 외에도 `$route` 객체는 `$route.query(쿼리 스트링)`, `$route.hash(해시 태그)` 등과 같은 다른 유용한 정보도 노출한다.

**404 Not Found Route**  
일반 파라미터(`:id`)는 슬래쉬(`/`)로 구분된 URL 사이의 문자만 일치시킨다. 무엇이든 일치시키려면 `param` 바로 뒤에 괄호 안에 정규식(regexp)을 사용할 수 있다.

```
const routes = [
  // will match everything and put it under `$route.params.pathMatch`
  { path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound },
  // will match anything starting with `/user-` and put it under `$route.params.afterUser`
  { path: '/user-:afterUser(.*)', component: UserGeneric },
]
```

**프로그래밍 방식 네비게이션**  
`<RouterLink>`를 사용하여 선언적 네비게이션용 anchor 태그를 사용하는 것 외에도 라우터 인스턴스 메소드를 사용하여 프로그래밍 방식으로 이를 수행할 수 있다.

**router.push**  
다른 URL로 이동하려면 `router.push`를 사용할 수 있다.   
이 메소드는 새로운 항목을 히스토리 스택에 넣기 때문에 사용자가 브라우저의 뒤로 가기 버튼을 클릭하면 이전 URL로 이동하게 된다.

```
// 리터럴 문자열 경로
router.push('/users/eduardo')

// 경로가 있는 개체
router.push({ path: '/users/eduardo' })

// 이름을 가지는 라우트
router.push({ name: 'user', params: { username: 'eduardo' } })

// 쿼리와 함께 사용, 결과적으로 /register?plan=private가 됩니다.
router.push({ path: '/register', query: { plan: 'private' } })

// 해시와 함께 사용, 결과적으로 /about#team가 됩니다.
router.push({ path: '/about', hash: '#team' })
```

```
const username = 'eduardo'
// URL을 수동으로 작성할 수 있지만 인코딩을 직접 처리해야 합니다.
router.push(`/user/${username}`) // -> /user/eduardo
// 위와 동일
router.push({ path: `/user/${username}` }) // -> /user/eduardo
// 가능하면 `name`과 `params`를 사용하여 자동 URL 인코딩의 이점을 얻습니다.
router.push({ name: 'user', params: { username } }) // -> /user/eduardo
// `params`는 `path`와 함께 사용할 수 없습니다.
router.push({ path: '/user', params: { username } }) // -> /user
```

**router.replace**  
`router.push`와 같은 역할을 하지만 유일한 차이는 새로운 히스토리 항목에 추가하지 않고 탐색한다는 것이다. 
이름에서 알 수 있듯이 현재 항목을 대체한다.

router.push 메소드에 `replace: true` 속성을 추가하여 동일하게 동작시킬 수 있다.
```
router.push({ path: '/home', replace: true })
// equivalent to
router.replace({ path: '/home' })
```

**router.go(n)**  
이 메소드는 `window.history.go(n)`와 비슷하게 히스토리 스택에서 앞으로 또는 뒤로 이동하는 단계를 나타내는 하나의 정수를 매개 변수로 사용한다.

```
// 한 단계 앞으로 갑니다. history.forward()와 같습니다. history.forward()와 같습니다.
router.go(1)

// 한 단계 뒤로 갑니다. history.back()와 같습니다.
router.go(-1)

// 3 단계 앞으로 갑니다.
router.go(3)

// 지정한 만큼의 기록이 없으면 자동으로 실패 합니다.
router.go(-100)
router.go(100)
```

**Params 변경 사항에 반응하기**  
매개 변수와 함께 라우트를 사용할 때 주의 해야할점은 사용자가 `/users/alice`에서 `/users/emma`로 이동할때 동일한 컴포넌트 인스턴스가 재사용된다는 것이다.
왜냐하면 두 라우트 모두 동일한 컴포넌트를 렌더링 하므로, 이전 인스턴스를 삭제한 다음 새 인스턴스를 만드는 것보다 효율적이다. 
그러나 이는 또한 컴포넌트의 라이프 사이클 훅이 호출되지 않음을 의미한다.

이렇게 동일한 컴포넌트를 재사용할 때 URL이 변경되게 되면 라이프사이클 훅이 호출되지 않기 때문에 훅에서 하던 일을 할 수 없다.

이럴때는 `Watcher(watch, watchEffect)` 또는 `beforeRouteUpdate navigation guard`를 사용하여 `params`와 같은 URL 변경사항에 반응할 수 있다.

```
// <script setup>
import { useRoute, watch } from 'vue-router';

const route = useRoute();

watch(
  () => route.params,
  (toParams, previousParams) => {
		// working
  }
);

export default {
	beforeRouteUpdate(to, from) {
		// working
		this.userData = await fetchUser(to.params.id)
	}
}

// <script setup>
import { onBeforeRouteUpdate } from 'vue-router';
onBeforeRouteUpdate((to, from) => {
  console.log('onBeforeRouteUpdate');
});
```
`beforeRouteUpdate`는 동일한 컴포넌트를 재사용할 때 URL이 변경되는 경우 호출된다.

---
#### 이름을 가지는 라우트(Named Routes)
Router 인스턴스를 생성할 때 `push`와 함께 `name`을 지정할 수 있다.
```
const routes = [
  {
    path: '/user/:username',
    name: 'user',
    component: User
  }
]

router.push({ name: 'user', params: { username: 'erina' } })
```

#### 이름을 가지는 뷰(Named Views)
때로는 여러 개의 뷰(router-view)를 중첩하지 않고 동시에 표시해야 하는 경우가 있다. 
이때 `router-view`에 이름을 지정하여 여러개의 `router-view`를 사용할 수 있다. 그리고 이름이 없는 router-view는 default가 이름으로 주어진다.

```
<router-view class="view left-sidebar" name="LeftSidebar"></router-view>
<router-view class="view main-content"></router-view>
<router-view class="view right-sidebar" name="RightSidebar"></router-view>
```
뷰 컴포넌트를 사용하여 렌더링 되므로 여러 뷰에는 동일한 라우트에 대해 여러 컴포넌트가 필요하다. 
`components` 옵션을 사용해야 한다.

```
const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: '/',
      components: {
        default: Home,
        // short for LeftSidebar: LeftSidebar
        LeftSidebar,
        // they match the `name` attribute on `<router-view>`
        RightSidebar,
      },
    },
  ],
})
```

#### 라우트 컴포넌트에 속성 전달
컴포넌트에서 `$route` 객체를 사용하면 특정 URL에서만 사용할 수 있게 되어 라우트와 강한 결합을 만든다. 
즉 컴포넌트의 유연성이 제한된다. 이러한 결합이 꼭 나쁜 것은 아니지만 `props` 옵션으로 이 동작을 분리할 수 있다.
컴포넌트와 라우터 속성을 분리하려면 다음과 같이 작성한다.  

**라우트에 의존된 컴포넌트**  
```
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const routes = [{ path: '/user/:id', component: User }]
```

**라우트 의존도 해제**  
```
const User = {
  // make sure to add a prop named exactly like the route param
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const routes = [{ path: '/user/:id', component: User, props: true }]
```

#### Boolean 모드
`props`를 true로 설정하면 `route.params`가 컴포넌트 props로 설정된다.

#### Named views
이름을 가지는 뷰가 있는 경우 각 Named Views에 대한 props 옵션을 정의해야 한다.

```
const routes = [
  {
    path: '/user/:id',
    components: { default: User, sidebar: Sidebar },
    props: { default: true, sidebar: false }
  }
]
```

**객체 모드**  
props가 객체일때 컴포넌트 props가 있는 그대로 설정된다. props가 정적일 때 유용하다.
```
const routes = [
  {
    path: '/promotion/from-newsletter',
    component: Promotion,
    props: { newsletterPopup: false }
  }
]
```

**함수 모드**  
props를 반환하는 함수를 만들 수 있다. 이를 통해 전달인자를 다른 타입으로 캐스팅하고 적정인 값을 라우트 기반 값과 결합된다.

```
const routes = [
  {
    path: '/search',
    component: SearchUser,
    props: route => ({ query: route.query.q })
  }
]
```
---
#### 네비게이션 가드(navigation guard)
이름에서 알 수 있듯이 Vue Router에서 제공하는 네비게이션 가드는 주로 페이지 이동을 리다이렉션 하거나 취소하여 특정 페이지 진입을 보호하는 데 사용된다.
라우트 탐색 프로세스에 연결하는 방법에는 전역, 라우트별 또는 컴포넌트가 있다.

#### 전역가드
Global Before Guards  
`router.beforeEach`를 사용하여 전역 가드를 등록할 수 있다.

```
const router = createRouter({ ... })

router.beforeEach((to, from) => {
  // ...
  // 네비게이션을 취소하려면 명시적으로 false를 반환합니다.
  return false
})
```
네비게이션이 트리거될 때마다 가드가 작성 순서에 따라 호출되기 전의 모든 경우에 발생한다.   
가드는 비동기식으로 실행될 수 있으며, 네비게이션은 모든 훅이 해결되기 전까지 보류 중으로 간주된다.

모든 가드 함수는 두 개의 인수를 받는다.  
- to: 라우팅 되는 `RouteLocationNormalizaed` 객체(라우트 위치 정보를 담고 있는 객체)  
- from: 라우팅 되기 전의 `RouteLocationNormalized` 객체(라우트 위치 정보를 담고 있는 객체)

그리고 선택적으로 다음 값중 하나를 반환할 수 있다.  
false: 현재 라우팅을 취소한다.  
A Route Location: 경로 위치를 반환하여 다른 위치로 리다이렉션할 수 있다. 
이때 전달될 값은 `router.push()`를 호출할 때와 같은 값을 내보내면 된다.

만약 `undefined` 또는 `true`가 반환되면 해당 네비게이션 가드가 검증이 된것으로 판단되어 다음 네비게이션 가드를 수행한다.


#### Global Resolve Guards
`router.beforeResolve`로 글로벌 가드를 등록할 수 있다. 이는 `router.beforeEach`와 유사하다. 
모든 컴포넌트 가드와 비동기 라우트 컴포넌트를 불러온 후 네비게이션 가드를 확인하기 전에 호출된다는 차이가 있다.

---
#### Plugins
플러그인은 일반적으로 Vue에 전역 수준의 기능을 추가할 때 사용하는 기능을 말한다.  
플러그인에 대해 엄격하게 정의된 범위는 없다. 일반적으로 플러그인이 유용한 시나리오는 다음과 같다.  
- app.component() // 메서드를 사용하여 전역 컴포넌트를 등록 하고자 할 때
- app.directive() // 메서드를 사용하여 커스텀 디렉티브를 등록 하고자 할때
- app.provide() // 사용하여 앱 전체에 리소스를 주입할때
- app.config.globalProperties // 전역 애플리케이션 인스턴스에 속성 또는 메서드를 추가하고자 할 때
- 위의 몇 가지 조합을 수행하는 라이브러리를 설치하고자 할 때 (예: vue-router)

**플러그인 작성하기**  
플러그인은 `install()` 메서드를 갖고 있는 객체나 단순히 설치 함수로 만들 수 있다.
```
// install() 메서드를 갖고 있는 객체
const objPlugin = {
	install(app, options) {
		
	}
}

// 단순히 설치 함수
function funcPlugin(app, options) {

}
```
그리고 작성한 플러그인을 전역 수준의 기능으로 추가할 때는 `app.use()` 메서드를 사용할 수 있다.

```
import { createApp } from 'vue';
import router from '@/router';
import { funcPlugin } from './plugins/func';
import { objPlugin } from './plugins/obj';

const app = createApp(App);
app.use(router);
app.use(funcPlugin, { // options });
app.use(objPlugin, { // options });
app.mount('#app');
```
`app.use()` 메소드로 플러그인을 설치하면 플러그인의 매개변수로 `app instance`와 `options`이 전달된다.
```
install: (app, options) => {
	// app.provide, app.component 등 사용할 수 있는 전역 인스턴스
	// app.use(plugin, { options }) 호출 시 전달한 두 번째 파라미터
}
```