---
badges:
  - breaking
---

# Глобальний АРІ "струшування дерева" <MigrationBadges :badges="$frontmatter.badges" />

## Синтаксис 2.x

Якщо вам коли-небудь доводилося вручну маніпулювати DOM у Vue, ви могли натрапити на цей шаблон:

```js
import Vue from 'vue'

Vue.nextTick(() => {
  // щось пов'язане з DOM
})
```

Або, якщо ви проводили модульне тестування програми з використанням асинхронних компонентів, швидше за все, ви написали щось на зразок цього:

```js
import { shallowMount } from '@vue/test-utils'
import { MyComponent } from './MyComponent.vue'

test('an async feature', async () => {
  const wrapper = shallowMount(MyComponent)

  // виконати деякі завдання, пов’язані з DOM

  await wrapper.vm.$nextTick()

  // виконати твердження
})
```

`Vue.nextTick()` – це глобальний API, відкритий безпосередньо для одного об'єкта Vue – насправді, метод екземпляра `nextTick()` — це просто зручна обгортка навколо `Vue.nextTick()` із контекстом `this` зворотного виклику, автоматично прив'язаним до поточного екземпляра для зручності.

Але що, якщо вам ніколи не доводилося мати справу з ручними маніпуляціями DOM, а також ви не використовуєте чи тестуєте асинхронні компоненти у своїй програмі? Або що, якщо з будь-якої причини ви надаєте перевагу старому доброму `window.setTimeout()` замість цього? У такому випадку код для `nextTick()` стане мертвим кодом, тобто кодом, який був написаний, але ніколи не використовувався. І мертвий код – це навряд чи добре, особливо в нашому контексті на стороні клієнта, де кожен кілобайт має значення.

Збірники модулів, такі як webpack і Rollup (на яких базується Vite), підтримують [струшування дерева](https://webpack.js.org/guides/tree-shaking/), який є модним терміном для «усунення мертвого коду». На жаль, через те, як написаний код у попередніх версіях Vue, щодо глобальних API, таких як `Vue.nextTick()`, не можна реалізувати tree-shaking і відповідний код буде включений в остаточний пакет незалежно від того, чи ці API фактично використовуються чи ні.

## Синтаксис 3.x

У Vue 3 глобальний і внутрішній API були реструктуровані з урахуванням струшування дерева. Як наслідок, доступ до глобальних API тепер доступний лише як іменований експорт для збірки модулів ES. Наприклад, наші попередні фрагменти тепер мають виглядати так:

```js
import { nextTick } from 'vue'

nextTick(() => {
  // щось пов'язане з DOM
})
```

and

```js
import { shallowMount } from '@vue/test-utils'
import { MyComponent } from './MyComponent.vue'
import { nextTick } from 'vue'

test('an async feature', async () => {
  const wrapper = shallowMount(MyComponent)

  // виконати деякі завдання, пов’язані з DOM

  await nextTick()

  // виконати твердження
})
```

Безпосередній виклик `Vue.nextTick()` тепер призведе до сумнозвісної помилки `undefined is not a function`.

З цією зміною, за умови, що збірник модулів підтримує струшування дерева, глобальні API, які не використовуються в програмі Vue, буде вилучено з остаточного пакета, що призведе до оптимального розміру файлу.

## API, які зазнали змін

Ця зміна стосується цих глобальних API у Vue 2.x:

- `Vue.nextTick`
- `Vue.observable` (замінено на `Vue.reactive`)
- `Vue.version`
- `Vue.compile` (тільки в повних збірках)
- `Vue.set` (тільки в сумісних збірках)
- `Vue.delete` (тільки в сумісних збірках)

## Внутрішні помічники

Окрім загальнодоступних API, багато внутрішніх помічників компонентів тепер також експортуються як іменовані експорти. Це дозволяє компілятору виводити код, який імпортує функції лише тоді, коли вони використовуються. Наприклад, такий шаблон:

```html
<transition>
  <div v-show="ok">hello</div>
</transition>
```

компілюється у щось подібне до наступного:

```js
import { h, Transition, withDirectives, vShow } from 'vue'

export function render() {
  return h(Transition, [withDirectives(h('div', 'hello'), [[vShow, this.ok]])])
}
```

По суті, це означає, що компонент `Transition` імпортується лише тоді, коли програма фактично використовує його. Іншими словами, якщо програма не має жодного компонента `<transition>`, код, що підтримує цю функцію, не буде присутній у фінальній збірці.

Завдяки глобальному струшуванню дерева користувачі «платять» лише за ті функції, якими вони фактично користуються. Навіть краще, знаючи, що додаткові функції не збільшать розмір комплекту для програм, які їх не використовують, розмір фреймворку стане набагато меншим занепокоєнням щодо додаткових основних функцій у майбутньому, якщо взагалі буде.

::: warning Важливо
Вищезазначене стосується лише [збірок модулів ES](https://github.com/vuejs/core/tree/master/packages/vue#which-dist-file-to-use) для використання з комплектувальниками з можливістю струшуванню дерева - збірка UMD все ще включає всі функції та розташовує все в глобальній змінній Vue (і компілятор створить відповідне виведення для використання API поза глобальними замість імпорту).
:::

## Використання в плагінах

Якщо ваш плагін покладається на глобальний API Vue 2.x, що зазнав змін, наприклад:

```js
const plugin = {
  install: Vue => {
    Vue.nextTick(() => {
      // ...
    })
  }
}
```

У Vue 3 вам доведеться імпортувати його явно:

```js
import { nextTick } from 'vue'

const plugin = {
  install: app => {
    nextTick(() => {
      // ...
    })
  }
}
```

Якщо ви використовуєте пакет модулів, як-от webpack, це може призвести до того, що вихідний код Vue буде об'єднано в плагін, і найчастіше це не те, чого ви очікуєте. Загальна практика, щоб цього не сталося, полягає в тому, щоб налаштувати комплектувальник модулів, щоб виключити Vue з остаточної збірки. У випадку webpack ви можете використовувати параметр конфігурації [`externals`](https://webpack.js.org/configuration/externals/):

```js
// webpack.config.js
module.exports = {
  /*...*/
  externals: {
    vue: 'Vue'
  }
}
```

Це вкаже webpack розглядати модуль Vue як зовнішню бібліотеку та не комплектувати його.

Якщо ви вибрали збірник модулів [Rollup](https://rollupjs.org/), ви фактично отримаєте той самий ефект безкоштовно, оскільки за замовчанням Rollup розглядатиме абсолютні ідентифікатори модулів (`'vue'` у нашому випадку) як зовнішні залежності і не включатиме їх у остаточний пакет. Однак під час пакування він може видати попередження [“Treating vue as external dependency” («Обробка vue як зовнішньої залежності»)](https://rollupjs.org/guide/en/#warning-treating-module-as-external-dependency), який можна придушити опцією `external`:

```js
// rollup.config.js
export default {
  /*...*/
  external: ['vue']
}
```