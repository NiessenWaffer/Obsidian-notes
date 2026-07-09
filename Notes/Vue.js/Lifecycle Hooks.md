# Lifecycle Hooks

Every Vue component goes through a series of steps from creation to destruction. Lifecycle hooks let you run code at specific stages.

## Diagram

```
beforeCreate
    │
  created    ← data is reactive, component not mounted yet
    │
beforeMount
    │
  mounted    ← DOM is ready (best time to fetch API data)
    │
   ... (component is alive)
    │
beforeUpdate (when reactive data changes)
    │
  updated
    │
   ... (component is about to be removed)
    │
beforeUnmount
    │
  unmounted  ← cleanup timers, remove listeners
```

## Most Common Hooks

### `created()`
Data is reactive, template not rendered yet. Good for initial data setup.

```js
created() {
  console.log('Component created')
}
```

### `mounted()`
DOM is ready. **Best place to fetch data from an API.**

```js
mounted() {
  fetch('/api/users')
    .then(res => res.json())
    .then(data => this.users = data)
}
```

### `beforeUnmount()`
Cleanup before component is destroyed.

```js
beforeUnmount() {
  clearInterval(this.timer)
  window.removeEventListener('scroll', this.handler)
}
```

## Vue 3 Composition API (for reference)

```js
import { onMounted, onUnmounted } from 'vue'

setup() {
  onMounted(() => {
    // fetch data
  })

  onUnmounted(() => {
    // cleanup
  })
}
```

---

**Previous:** [[Props]]
**Next:** [[Vue Router]]
