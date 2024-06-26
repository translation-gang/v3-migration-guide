---
badges:
  - breaking
---

# Изменения в глобальном API <MigrationBadges :badges="$frontmatter.badges" />

Vue 2.x имеет ряд глобальных API и конфигураций, которые глобально изменяют поведение Vue. Например, для регистрации глобального компонента можно воспользоваться методом API `Vue.component`:

```js
Vue.component('button-counter', {
  data: () => ({
    count: 0
  }),
  template: '<button @click="count++">Счётчик нажатий — {{ count }}.</button>'
})
```

Аналогичным образом объявляется глобальная директива:

```js
Vue.directive('focus', {
  inserted: (el) => el.focus()
})
```

Несмотря на удобство такого подхода, он приводит к нескольким проблемам. Технически, Vue 2 не имеет концепта «приложения». То что считаем приложением — просто корневой экземпляр Vue, созданный с помощью `new Vue()`. Каждый корневой экземпляр, созданный из одного и того же конструктора Vue будет **иметь одну и ту же глобальную конфигурацию**. В результате:

- Глобальная конфигурация позволяет легко случайно загрязнить другие тестовые случаи во время тестирования. Пользователям потребуется аккуратно хранить оригинальную глобальную конфигурацию и восстанавливать её после каждого теста (например, сбрасывать `Vue.config.errorHandler`). Некоторые API, такие как `Vue.use` и `Vue.mixin` даже не предоставляют возможности откатить свои изменения и эффекты. Это приводит к тому, что тесты с участием плагинов становятся очень хитрыми. Фактически, vue-test-utils должны реализовывать специальный API `createLocalVue` чтобы справиться с этим:

  ```js
  import { createLocalVue, mount } from '@vue/test-utils'

  // создание расширенного конструктора `Vue`
  const localVue = createLocalVue()

  // установка плагина «глобально» в «локальном» конструкторе Vue
  localVue.use(MyPlugin)

  // передача `localVue` в опции монтирования
  mount(Component, { localVue })
  ```

- Глобальная конфигурация затрудняет совместное использование одной и той же копии Vue между несколькими «приложениями» на одной странице с различными глобальными конфигурациями.

  ```js
  // это повлияет на оба корневых экземпляра
  Vue.mixin({
    /* ... */
  })

  const app1 = new Vue({ el: '#app-1' })
  const app2 = new Vue({ el: '#app-2' })
  ```

Чтобы избежать этих проблем, во Vue 3 представляем…

## Новый глобальный API: `createApp`

Вызов `createApp` возвращает _экземпляр приложения_, новый концепт во Vue 3.

```js
import { createApp } from 'vue'

const app = createApp({})
```

При использовании сборки с CDN `createApp` доступен через глобальный объект `Vue`:

```js
const { createApp } = Vue

const app = createApp({})
```

Экземпляр приложения представляет собой подмножество от глобального API Vue 2. Главное правило заключается в том, что _любое API, которое глобально изменяет поведение Vue теперь переносится в экземпляр приложения_. Ниже таблица соответствий глобального API Vue 2 и соответствующих API экземпляра:

| Глобальное API в 2.x       | API экземпляра (`app`) в 3.x                                                                                                            |
|----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| Vue.config                 | app.config                                                                                                                              |
| Vue.config.productionTip   | _удалено_ ([см. ниже](#config-productiontip-removed))                                                                          |
| Vue.config.ignoredElements | app.config.compilerOptions.isCustomElement ([см. ниже](#config-ignoredelements-is-now-config-compileroptions-iscustomelement)) |
| Vue.component              | app.component                                                                                                                           |
| Vue.directive              | app.directive                                                                                                                           |
| Vue.mixin                  | app.mixin                                                                                                                               |
| Vue.use                    | app.use ([см. ниже](#a-note-for-plugin-authors))                                                                            |
| Vue.prototype              | app.config.globalProperties ([см. ниже](#vue-prototype-replaced-by-config-globalproperties))                                   |
| Vue.extend                 | _удалён_ ([см. ниже](#vue-extend-removed))                                                                                         |

Все другие глобальные API, которые глобально не изменяют поведение, теперь доступны именованными экспортами, как описывается в разделе [Treeshaking глобального API](./global-api-treeshaking.html).

### Удалено свойство `config.productionTip`

Во Vue 3.x, подсказка «use production build» отображается только при использовании «dev + full build» (сборка, с компилятором шаблонов и отображением предупреждений).

Так как сборки в виде ES-модулей используются с системами сборки, и в большинстве случаев CLI или шаблон конфигурируют переменную окружения сборки, то эти предупреждения теперь больше не будут показываться.

[Флаг сборки для миграции: `CONFIG_PRODUCTION_TIP`](../migration-build.html#compat-configuration)

### Свойство `config.ignoredElements` теперь `config.compilerOptions.isCustomElement`

Эта опция конфигурации добавлена для поддержки нативных пользовательских элементов, поэтому переименование лучше отражает что она делает. Новая опция ожидает функцию, что обеспечивает большую гибкость, нежели старый подход со строками и RegExp:

```js
// раньше
Vue.config.ignoredElements = ['my-el', /^ion-/]

// теперь
const app = createApp({})

app.config.compilerOptions.isCustomElement = tag => tag.startsWith('ion-')
```

:::tip Внимание
В 3.х, проверка, является ли элемент компонентом или нет, была перенесена на этап компиляции шаблона, поэтому данная опция конфигурации используется только при компиляции шаблонов на лету. При использовании только-runtime сборки, опция `isCustomElement` должна передаваться в `@vue/compiler-dom` в настройках сборки — например, через [опцию `compilerOptions` в vue-loader](https://vue-loader.vuejs.org/ru/options.html#compileroptions).

- Если `config.compilerOptions.isCustomElement` указан при использовании только-runtime сборки, то будет выведено предупреждение о необходимости передавать опцию в настройках сборки;
- Это будет новая опция верхнего уровня в конфигурации Vue CLI.
:::

[Флаг сборки для миграции: `CONFIG_IGNORED_ELEMENTS`](../migration-build.html#compat-configuration)

### Свойство `Vue.prototype` заменено на `config.globalProperties`

Во Vue 2, расширение `Vue.prototype` обычно использовалось для добавления свойств, которые были бы доступны во всех компонентах.

Эквивалентом во Vue 3 теперь будет [`config.globalProperties`](https://ru.vuejs.org/api/application.html#app-config-globalproperties). Эти свойства будут скопированы как часть процесса инициализации компонента внутри приложения:

```js
// раньше - Vue 2
Vue.prototype.$http = () => {}
```

```js
// теперь - Vue 3
const app = createApp({})

app.config.globalProperties.$http = () => {}
```

Использование `provide` (обсуждается [ниже](#provide-inject)) также следует рассматривать как альтернативу `globalProperties`.

[Флаги сборки для миграции: `GLOBAL_PROTOTYPE`](../migration-build.html#compat-configuration)

### Удалён метод `Vue.extend`

Во Vue 2.x метод `Vue.extend` использовался для создания «подкласса» базового конструктора Vue с аргументом, который должен быть объектом, содержащим опции компонента. Во Vue 3.x больше нет концепции конструкторов компонентов. Монтируемый компонент всегда должен использовать глобальный API `createApp`:

```js
// раньше - Vue 2
// создание конструктора
const Profile = Vue.extend({
  template: '<p>{{ firstName }} {{ lastName }} — {{ alias }}</p>',
  data() {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg'
    }
  }
})

// создание экземпляра Profile и его монтирование на элементе
new Profile().$mount('#mount-point')
```

```js
// сейчас - Vue 3
const Profile = {
  template: '<p>{{ firstName }} {{ lastName }} — {{ alias }}</p>',
  data() {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg'
    }
  }
}

Vue.createApp(Profile).mount('#mount-point')
```

#### Выведение типов

Во Vue 2 метод `Vue.extend` также использовался с TypeScript для выведения типов опций компонента. Во Vue 3 можно использовать глобальный API `defineComponent` вместо `Vue.extend` для этих же целей.

Обратите внимание, что хотя возвращаемый тип `defineComponent` является типом, подобным конструктору, он используется только для вывода TSX. Во время выполнения `defineComponent` по большей части ничего не делает и возвращает объект опций как есть.

#### Наследование компонентов

Во Vue 3 рекомендуется отдавать предпочтение композиции через [Composition API](https://ru.vuejs.org/guide/reusability/composables.html) вместо наследования или примесей. Если по какой-то причине всё ещё требуется наследование компонентов, то можно воспользоваться [опцией `extends`](https://ru.vuejs.org/api/options-composition.html#extends) вместо `Vue.extend`.

[Флаги сборки для миграции: `GLOBAL_EXTEND`](../migration-build.html#compat-configuration)

### Примечание для разработчиков плагинов

Частая практика разработчиков плагинов — устанавливать плагины автоматически в UMD-сборках с помощью `Vue.use`. Например, официальный плагин `vue-router` устанавливается в окружении браузера таким образом:

```js
var inBrowser = typeof window !== 'undefined'
/* … */
if (inBrowser && window.Vue) {
  window.Vue.use(VueRouter)
}
```

Так как глобальный API `use` больше недоступен во Vue 3, этот метод перестанет работать, а вызов `Vue.use()` теперь выведет предупреждение. Вместо этого, пользователь теперь должен явно указать использования плагина в экземпляре приложения:

```js
const app = createApp(MyApp)
app.use(VueRouter)
```

## Монтирование экземпляра приложения

После инициализации через `createApp(/* опции */)`, экземпляр приложения `app` можно использовать для монтирования корневого экземпляра компонента через `app.mount(domTarget)`:

```js
import { createApp } from 'vue'
import MyApp from './MyApp.vue'

const app = createApp(MyApp)
app.mount('#app')
```

Со всеми этими изменениями, компонент и директива, о которых обсуждали в начале этого руководства, будут переписаны примерно таким образом:

```js
const app = createApp(MyApp)

app.component('button-counter', {
  data: () => ({
    count: 0
  }),
  template: '<button @click="count++">Счётчик нажатий — {{ count }}.</button>'
})

app.directive('focus', {
  mounted: (el) => el.focus()
})

// теперь каждый экземпляр приложения, смонтированный с помощью app.mount(),
// вместе со своим деревом компонентов, будет иметь тот же самый компонент
//  “button-counter” и директиву “focus”, не загрязняя глобальное окружение
app.mount('#app')
```

[Флаги сборки для миграции: `GLOBAL_MOUNT`](../migration-build.html#compat-configuration)

## Provide / Inject

Аналогично опции `provide` в корневом экземпляре в 2.x, экземпляр приложения во Vue 3 также может предоставлять зависимости, которые могут внедряться любым компонентом внутри приложения:

```js
// в точке создания экземпляра приложения
app.provide('guide', 'Руководство по Vue 3')

// в дочернем компоненте
export default {
  inject: {
    book: {
      from: 'guide'
    }
  },
  template: `<div>{{ book }}</div>`
}
```

Использование `provide` особенно полезно при создании плагина, в качестве альтернативы свойству `globalProperties`.

## Общая конфигурация между приложениями

Одним из способов создания общей конфигурации, например из компонентов или директив, для использования между приложениями — создать функцию фабрику, подобную такой:

```js
import { createApp } from 'vue'
import Foo from './Foo.vue'
import Bar from './Bar.vue'

const createMyApp = (options) => {
  const app = createApp(options)
  app.directive('focus' /* ... */)

  return app
}

createMyApp(Foo).mount('#foo')
createMyApp(Bar).mount('#bar')
```

Теперь директива `focus` будет доступна как в экземплярах `Foo` и `Bar`, так и в их потомках.
