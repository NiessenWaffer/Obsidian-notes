# Vue Components

Components are **reusable, self-contained pieces** of the UI. Every Vue app is a tree of components.

## Why Components?
- Reusability — write once, use everywhere
- Separation of concerns
- Easier testing & maintenance

## Basic Component (Vue 3 Options API)

```js
// Defining a component
app.component('my-button', {
  template: `
    <button @click="count++">
      Clicked {{ count }} times
    </button>
  `,
  data() {
    return { count: 0 }
  }
})
```

## Single-File Components (`.vue`)

The modern way — all in one file:

```vue
<template>
  <button class="btn" @click="handleClick">
    <slot />
  </button>
</template>

<script>
export default {
  methods: {
    handleClick() {
      this.$emit('click')
    }
  }
}
</script>

<style scoped>
.btn { padding: 8px 16px; }
</style>
```

- `<template>` — HTML
- `<script>` — JS logic
- `<style scoped>` — CSS (scoped = only this component)

## Parent-Child Relationship

Components form a **tree**:
```
App
├── Header
├── Sidebar
│   └── NavItem
└── MainContent
    └── ArticleCard
```

## `$emit` — Child to Parent

Child sends events up:

```vue
<!-- Child -->
<button @click="$emit('delete', id)">Delete</button>

<!-- Parent -->
<ArticleCard @delete="handleDelete" />
```

---

**Next:** [[Props]] — passing data from parent to child
