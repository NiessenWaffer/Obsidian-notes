# Reactivity — The One Idea Everything Else Builds On

**Date:** 2026-07-10
**Prerequisite:** [[01 - Foundation of Vue]] and [[02 - Vue Foundation Roadmap]]
**Next:** [[04 - Options API Shape]]

---

## The Problem Vue Solves

In plain JavaScript, if you update a variable, the DOM does **not** update automatically:

```js
let count = 5;
document.querySelector('#num').innerText = count;

count = 10;
// The DOM still shows 5. You'd have to manually re-set innerText.
```

You have to **manually push changes to the screen.** Vue's whole reason for existing is to remove that manual step.

---

## How Vue Fixes It

```js
export default {
  data() {
    return {
      count: 5
    }
  }
}
```

```html
<p>{{ count }}</p>
<button @click="count++">Add</button>
```

Click the button → `count` changes → the `<p>` updates itself. **No manual DOM code.**

This is reactivity. When your data changes, Vue updates the DOM automatically.

---

## Why `data()` Is a Function (Not an Object)

If `data` were a plain object, every instance of that component would **share the same object in memory** — change it in one instance, it changes in all of them.

```js
// ❌ WRONG — all instances share one object
data: {
  count: 0
}

// ✅ CORRECT — each instance gets its own fresh object
data() {
  return {
    count: 0
  }
}
```

Returning a fresh object from a function means each component instance gets its own **independent copy** of the data.

---

## How Vue Knows to Update the DOM

Under the hood, Vue 2 walks through everything in `data()` and converts each property into a **getter/setter pair**:

```js
// What you write:
data() {
  return { count: 5 }
}

// What Vue does internally (simplified):
// It wraps 'count' with Object.defineProperty()
// - getter: "who's reading count? I'll note them as a dependency."
// - setter: "count changed! I'll notify everyone who depends on it."
```

When your template reads `count`, Vue notes: *"this piece of the page depends on count."*

When `count++` runs, the setter fires, Vue checks who depends on it, and **re-renders just that part.**

That's the entire trick — nothing magic, just tracked getters/setters.

---

## Vue 2 Reactivity Gotchas

This will save you a debugging headache later.

### Gotcha 1: Adding New Properties to Objects

If you add a new property to an object **after it's already reactive**, Vue 2 doesn't see it — because it never wrapped that property in a getter/setter.

```js
data() {
  return {
    user: { name: 'Niesen' }
  }
},
methods: {
  addAge() {
    this.user.age = 25;
    // ❌ Vue 2 won't detect this — no re-render
    // The DOM won't update to show the age

    this.$set(this.user, 'age', 25);
    // ✅ This works — Vue wraps the new property in getter/setter
  }
}
```

**Rule of thumb:** If you're adding a brand new property to an object that was already in `data()`, use `Vue.set()` or `this.$set()`.

### Gotcha 2: Array Index Updates

Same issue with arrays — Vue 2 cannot detect when you directly set an array item by index:

```js
data() {
  return {
    items: ['apple', 'banana', 'cherry']
  }
},
methods: {
  changeFirst() {
    this.items[0] = 'avocado';
    // ❌ Vue 2 won't detect this — no re-render

    this.items.splice(0, 1, 'avocado');
    // ✅ This works — splice triggers reactivity

    // OR:
    this.$set(this.items, 0, 'avocado');
    // ✅ Also works
  }
}
```

**Why?** Vue 2 wraps array mutation methods (`push`, `pop`, `splice`, `shift`, `unshift`, `reverse`, `sort`) to trigger reactivity, but it **cannot** intercept index-based assignment.

---

## Summary: Vue 2 Reactivity Rules

| What you do | Vue's reaction |
|-------------|----------------|
| Change a property in `data()` | DOM updates automatically ✓ |
| Add a new property to existing object | NOT reactive ✗ (use `Vue.set`) |
| Modify array by index (`arr[0] = x`) | NOT reactive ✗ (use `splice` or `Vue.set`) |
| Use `push`, `pop`, `splice` on array | Reactive ✓ |
| Reassign entire array/object | Reactive ✓ |

---

## Research Task (Before Lesson 2)

**Look up:** "Vue 2 reactivity caveats" or "Vue 2 Vue.set array/object limitations"

Don't just read it — try to find **one real example** in your Item Importation module where you might have hit this without realizing it, or could imagine hitting it.

Examples to think about:
- Do you dynamically add properties to objects from API responses?
- Do you replace array items by index anywhere?
- Do you have objects that start with some fields and gain more later?

**Come back and tell me what you found**, or tell me it's confusing and we'll dig in more before moving to [[04 - Options API Shape]].
