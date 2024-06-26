---
badges:
  - removed
---

# $children <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Свойство экземпляра `$children` было удалено из Vue 3.0 и больше не поддерживается.

## Синтаксис в 2.x

В 2.x разработчики могли получить доступ к прямым дочерним компонентам текущего экземпляра с помощью `this.$children`:

```vue
<template>
  <div>
    <img alt="Vue logo" src="./assets/logo.png">
    <my-button>Измени лого</my-button>
  </div>
</template>

<script>
import MyButton from './MyButton'

export default {
  components: {
    MyButton
  },
  mounted() {
    console.log(this.$children) // [VueComponent]
  }
}
</script>
```

## Что изменилось в 3.x

В версии 3.x свойство `$children` удалено и больше не поддерживается. Вместо этого, если вам нужно получить доступ к экземпляру дочернего компонента, мы рекомендуем использовать [template refs](https://ru.vuejs.org/guide/essentials/template-refs.html#template-refs).

## Стратегия миграции

[Флаг сборки для миграции: `INSTANCE_CHILDREN`](../migration-build.html#compat-configuration)
