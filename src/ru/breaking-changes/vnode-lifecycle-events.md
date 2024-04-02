---
badges:
  - breaking
---

# События жизненного цикла VNode <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

В Vue 2 можно было использовать события для отслеживания ключевых этапов жизненного цикла компонента. Имена этих событий начинались с префикса `hook:`, за которым следовало имя соответствующего хука жизненного цикла.

В Vue 3 этот префикс был заменен на `vue:`. Кроме того, эти события теперь доступны как для HTML-элементов, так и для компонентов.

## 2.x Синтаксис

В Vue 2 имя события совпадает с именем эквивалентного хука жизненного цикла, с префиксом `hook:`:

```html
<template>
  <child-component @hook:updated="onUpdated">
</template>
```

## 3.x Синтаксис

В Vue 3 имя события имеет префикс `vue:`:

```html
<template>
  <child-component @vue:updated="onUpdated">
</template>
```

## Стратегия миграции

В большинстве случаев для этого достаточно изменить префикс. Хуки жизненного цикла `beforeDestroy` и `destroyed` были переименованы в `beforeUnmount` и `unmounted` соответственно, поэтому соответствующие имена событий также должны быть обновлены.

[Флаг миграционной сборки: `INSTANCE_EVENT_HOOKS`](../migration-build.html#compat-configuration)

## См. также

- [Руководство по миграции - API событий](./events-api.html)