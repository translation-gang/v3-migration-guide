---
title: Удалён модификатор v-on.native
badges:
  - breaking
---

# Удалён модификатор `v-on.native` <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Модификатор `.native` для `v-on` был удалён.

## Синтаксис в 2.x

Обработчики событий, переданные в компонент через `v-on` по умолчанию вызываются по событию, которое генерируется с помощью `this.$emit`. Для добавления обработчика нативного события DOM к корневому элементу дочернего компонента требовалось использовать модификатор `.native`:

```html
<my-component
  v-on:close="handleComponentEvent"
  v-on:click.native="handleNativeClickEvent"
/>
```

## Что изменилось в 3.x

Модификатор `.native` для `v-on` был удалён. В то же время, [новая опция `emits`](./emits-option.md) позволяет дочерним компонентам определять, какие события он может генерировать.

Следовательно, Vue теперь добавит все слушатели событий, которые _не были объявлены_ как генерируемые дочерним компонентом, в качестве нативных слушателей событий (кроме случаев, когда `inheritAttrs: false` была задана в опциях дочернего компонента).

```html
<my-component
  v-on:close="handleComponentEvent"
  v-on:click="handleNativeClickEvent"
/>
```

Содержимое `MyComponent.vue`:

```html
<script>
  export default {
    emits: ['close']
  }
</script>
```

## Стратегия миграции

- удалить все экземпляры модификатора `.native`.
- убедитесь, что все компоненты документируют свои события с помощью опции `emits`.

[Флаг сборки для миграции: `COMPILER_V_ON_NATIVE`](../migration-build.html#compat-configuration)

## См. также

- [Соответствующий RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0031-attr-fallthrough.md#v-on-listener-fallthrough)
- [Руководство по миграции — Новая опция emits](./emits-option.html)
- [Руководство по миграции — Удалены `$listeners`](./listeners-removed.html)
- [Руководство по миграции — Изменения в API render-функций](./render-function-api.html)
