---
badges:
  - breaking
---

# Изменения использования атрибута `key` <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

- **НОВОЕ:** больше не требуется указывать `key` на ветках `v-if`/`v-else`/`v-else-if`, поскольку Vue теперь автоматически генерирует уникальные `key`.
  - **КАРДИНАЛЬНОЕ ИЗМЕНЕНИЕ:** Если вручную указывали `key`, то каждая ветка должна использовать свой уникальный `key`.
- **КАРДИНАЛЬНОЕ ИЗМЕНЕНИЕ:** для `<template v-for>` атрибут `key` теперь должен указываться на теге `<template>` (а не на его дочерних элементах).

## Предыстория

Специальный атрибут `key` используется в качестве подсказки для Vue и его алгоритма виртуального дерева для отслеживания идентичности узла. Благодаря ему Vue понимает, когда можно переиспользовать и обновить существующие узлы, а когда требуется переупорядочить или пересоздать их. Подробнее можно узнать в следующих разделах:

- [Отрисовка списков: сохранение состояния](https://ru.vuejs.org/guide/essentials/list.html#maintaining-state-with-key)
- [Справочник API: специальный атрибут `key`](https://ru.vuejs.org/api/built-in-special-attributes.html#key)

## Использование на ветках с условием

Во Vue 2.x рекомендовалось использовать `key` на ветках `v-if`/`v-else`/`v-else-if`.

```html
<!-- Vue 2.x -->
<div v-if="condition" key="yes">Да</div>
<div v-else key="no">Нет</div>
```

Пример выше будет работать и во Vue 3.x. Но теперь перестаём рекомендовать указывать атрибут `key` на ветках `v-if`/`v-else`/`v-else-if`, поскольку уникальные `key` теперь будут генерироваться автоматически на ветках с условиями, если не указаны вручную.

```html
<!-- Vue 3.x -->
<div v-if="condition">Да</div>
<div v-else>Нет</div>
```

Кардинальное изменение заключается в том, что если указываете `key` вручную, то каждая ветка должна использовать свой уникальный `key`. В большинстве случаев можно просто удалить атрибуты `key`.

```html
<!-- Vue 2.x -->
<div v-if="condition" key="a">Да</div>
<div v-else key="a">Нет</div>

<!-- Vue 3.x (рекомендуется: удалить атрибуты key) -->
<div v-if="condition">Да</div>
<div v-else>Нет</div>

<!-- Vue 3.x (альтернатива: убедиться в уникальности атрибутов key) -->
<div v-if="condition" key="yes">Да</div>
<div v-else key="no">Нет</div>
```

## Использование с `<template v-for>`

Во Vue 2.x, не было возможности `<template>` указывать атрибут `key`. Вместо этого надо было указывать уникальный `key` на каждом из его дочерних элементов.

```html
<!-- Vue 2.x -->
<template v-for="item in list">
  <div :key="'heading-' + item.id">...</div>
  <span :key="'content-' + item.id">...</span>
</template>
```

Во Vue 3.x, атрибут `key` теперь можно указывать на теге `<template>`.

```html
<!-- Vue 3.x -->
<template v-for="item in list" :key="item.id">
  <div>...</div>
  <span>...</span>
</template>
```

Аналогично, при использовании `<template v-for>` с дочерним элементом, использующим `v-if`, атрибут `key` нужно переместить на тег `<template>`.

```html
<!-- Vue 2.x -->
<template v-for="item in list">
  <div v-if="item.isVisible" :key="item.id">...</div>
  <span v-else :key="item.id">...</span>
</template>

<!-- Vue 3.x -->
<template v-for="item in list" :key="item.id">
  <div v-if="item.isVisible">...</div>
  <span v-else>...</span>
</template>
```
