# Foundation of Vue: Reactivity-First Learning Path

**Date:** 2026-07-10
**Approach:** Reactivity-first (Option B)
**Context:** Laravel 7 + Vue 2 Options API + Buefy

---

## Why This Approach?

There are three common ways people teach Vue:

### A) Official-docs order
Instance → directives → components → props/events → lifecycle → advanced (Vuex, router)

**Problem:** Teaches syntax before *why* Vue behaves the way it does. People memorize `v-if` vs `v-show` without understanding reactivity, then get confused later when things don't update.

### B) Reactivity-first (This path)
Start with why Vue re-renders things, then layer syntax on top.

**Benefit:** Slower start, but every later topic (computed, watch, props) becomes "oh, this is just reactivity doing its thing" instead of a new rule to memorize.

### C) Project-based
Jump straight into building a small app, learn pieces as needed.

**Problem:** Leaves gaps — usually right in the fundamentals people skip past because "it worked." Since you already have basic programming knowledge and you're already touching Vue 2 Options API + Buefy in real work (approval workflows, tables, forms), option C alone is risky — you'd learn "how to make this modal work" without understanding *why* it works, which bites you later when something breaks in a non-obvious way.

---

## The Core Principle: Reactivity

Vue's superpower is **reactivity** — when your data changes, the DOM updates automatically. Everything else in Vue is built on top of this.

### What Reactivity Actually Means

```js
const app = new Vue({
  el: '#app',
  data() {
    return {
      message: "Hello Vue!"
    }
  }
})
```

When you write `data()` as a function returning an object, Vue wraps every property in reactive getters/setters. This means:

1. Vue **tracks** when you read a property (getter)
2. Vue **tracks** when you change a property (setter)
3. When a tracked property changes, Vue **re-renders** only the parts of the DOM that use that property

**This is the foundation.** Every other Vue feature exists to work with this system.

---

## Reactivity in Action: Small Concrete Examples

### Example 1: Basic Reactivity

```html
<div id="app">
  <p>{{ message }}</p>
  <button @click="message = 'Changed!'">Change</button>
</div>

<script>
const app = new Vue({
  el: '#app',
  data() {
    return {
      message: "Hello Vue!"
    }
  }
})
</script>
```

**What happens:** Click the button → `message` changes → Vue detects the change → Vue updates only the `<p>` tag. The button stays untouched.

### Example 2: Reactivity with Objects

```js
data() {
  return {
    user: {
      name: "Wilcon",
      role: "admin"
    }
  }
}
```

Vue tracks nested properties too. If you change `this.user.name = "New Name"`, Vue knows to re-render anything that uses `user.name`.

**Gotcha:** Vue 2 cannot detect property addition. This won't be reactive:
```js
this.user.age = 25  // NOT reactive in Vue 2
```

You need `Vue.set(this.user, 'age', 25)` or `this.$set(this.user, 'age', 25)`.

### Example 3: Reactivity with Arrays

```js
data() {
  return {
    items: ["apple", "banana"]
  }
}
```

Vue wraps array mutation methods (`push`, `pop`, `splice`, etc.) to trigger reactivity. But reassigning the array won't trigger updates:
```js
this.items = ["new", "array"]  // This works
this.items[0] = "changed"      // This does NOT trigger update in Vue 2
```

---

## How This Connects to Your Real Work

In your Laravel/Vue/Buefy projects, you're already using reactivity without thinking about it:

- **Tables with data:** When `data()` returns an array of records, and you use `v-for` to render rows, any change to that array automatically updates the table
- **Forms with `v-model`:** Two-way binding is reactivity — input changes update data, data changes update input
- **Conditional modals:** `v-show="showModal"` works because Vue watches `showModal` and toggles CSS when it changes

Understanding reactivity-first means you'll understand *why* these things work, not just *how* to make them work.

---

## Learning Path Outline

This foundation document leads into the [[02 - Vue Foundation Roadmap|full roadmap]]. Each lesson builds on the previous:

1. [[03 - Reactivity]] — How Vue tracks changes and updates the DOM
2. [[04 - Options API Shape]] — Where to put your logic
3. [[05 - Template Bindings]] — Directives that leverage reactivity
4. [[06 - Computed vs Methods]] — The #1 beginner confusion
5. [[07 - Components]] — How pieces talk
6. [[08 - Lifecycle Hooks]] — When your code actually runs
7. [[09 - Forms and v-model]] — Directly relevant to Buefy

---

## Quick Reference: Reactivity Rules

| What you do | Vue's reaction |
|-------------|----------------|
| Change a property in `data()` | DOM updates automatically |
| Add a new property to existing object | NOT reactive (use `Vue.set`) |
| Modify array by index | NOT reactive in Vue 2 (use `Vue.set` or splice) |
| Use `push`, `pop`, `splice` on array | Reactive ✓ |
| Reassign entire array/object | Reactive ✓ |

---

## Next Step

→ [[03 - Reactivity]] — Deep dive into how Vue's reactivity actually works under the hood
