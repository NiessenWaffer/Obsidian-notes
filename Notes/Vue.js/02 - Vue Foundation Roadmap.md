# Vue Foundation Roadmap

**Date:** 2026-07-10
**Prerequisite:** [[01 - Foundation of Vue]] (Step 1: Why reactivity-first)

---

## The Order

Each topic only makes sense after the one before it. Skipping ahead creates gaps that compound later.

| # | Topic | Why it's here |
|---|-------|---------------|
| 1 | [[03 - Reactivity]] | The "why" behind everything Vue does |
| 2 | [[04 - Options API Shape]] | How `data`, `methods`, `computed`, `watch` connect as one object |
| 3 | [[05 - Template Bindings]] | Directives are just "reactivity, but in HTML" |
| 4 | [[06 - Computed vs Methods]] | The #1 source of beginner confusion — easy once reactivity clicks |
| 5 | [[07 - Components]] | How pieces talk without global state mess |
| 6 | [[08 - Lifecycle Hooks]] | When your code actually runs (API calls, DOM access) |
| 7 | [[09 - Forms and v-model]] | Directly relevant to Buefy inputs/selects you already use |

---

## Why This Order

### 1. [[03 - Reactivity]]

Everything in Vue flows from here. If you don't understand *how* Vue tracks changes and updates the DOM, every other concept is just syntax to memorize.

**After this, you'll understand:**
- Why `data()` is a function (not an object)
- Why `Vue.set()` exists in Vue 2
- Why changing an array index doesn't trigger re-render

---

### 2. [[04 - Options API Shape]]

Once you understand reactivity, the Options API structure makes sense — it's not random config, it's a organized way to tell Vue "here's my state, here's how to react to it."

**After this, you'll understand:**
- Why `data`, `methods`, `computed`, `watch` are separate (not all in one object)
- How `this` ties everything together inside a component
- Where to put logic when you're staring at a blank `.vue` file

---

### 3. [[05 - Template Bindings]]

Directives (`v-if`, `v-for`, `v-bind`, `v-on`) look like new syntax, but they're just "reactivity expressed in HTML." `v-if` is a reactive conditional. `v-for` is a reactive loop. `v-bind` is a reactive attribute.

**After this, you'll understand:**
- Why templates feel like HTML with superpowers (because they are)
- The difference between `v-if` and `v-show` from a reactivity perspective
- Why `:key` matters for `v-for`

---

### 4. [[06 - Computed vs Methods]]

This is where most beginners get confused. "When do I use computed? When do I use a method? What's a watcher?" Once reactivity clicks, the answer is obvious:

- **Computed:** Derived value that caches based on reactive dependencies
- **Method:** Function that runs on demand (no caching)
- **Watch:** Side effect when a reactive property changes

**After this, you'll understand:**
- Why computed is better than methods for template values
- When to use watch (hint: usually you don't need it)
- How this applies to your Buefy table filtering/sorting logic

---

### 5. [[07 - Components]]

Components are isolated reactive units. Props pass data down (parent → child). Events pass data up (child → parent). This keeps data flow predictable without global state.

**After this, you'll understand:**
- Why components are reusable (isolated reactive instances)
- Why you can't mutate props directly (one-way data flow)
- How to structure your Laravel/Vue pages as component trees

---

### 6. [[08 - Lifecycle Hooks]]

Your code doesn't run whenever you want — it runs at specific moments. `created` runs after the instance is set up. `mounted` runs after the DOM is ready. `beforeDestroy` runs before cleanup.

**After this, you'll understand:**
- Where to make API calls (created vs mounted)
- Where to access DOM elements (mounted)
- Where to clean up timers/listeners (beforeDestroy)

---

### 7. [[09 - Forms and v-model]]

`v-model` is syntactic sugar for `v-bind` + `v-on`. It's the bridge between reactivity and user input. Directly relevant to every Buefy form you build.

**After this, you'll understand:**
- How `v-model` works under the hood (it's not magic)
- Why `v-model` on components needs `value` + `input` events
- How to handle Buefy `<b-input>`, `<b-select>`, `<b-checkbox>` properly

---

## What Comes After (Not Foundation)

These topics build on the foundation but aren't required to understand Vue's core:

| Topic | When to learn |
|-------|---------------|
| Component communication edge cases | When props/events feel limiting |
| Vuex / centralized state | When prop drilling gets painful |
| Mixins | When you have repeated logic across components |
| Custom events / event bus | When you need cross-component communication without Vuex |

---

## How to Use This Roadmap

1. Read each topic in order
2. After each, write a small example using your real work context (Laravel approval workflows, Buefy tables)
3. Don't skip ahead — if something doesn't click, reread the previous topic
4. Check off each topic as you complete it

**Progress:** ☐ 1 → ☐ 2 → ☐ 3 → ☐ 4 → ☐ 5 → ☐ 6 → ☐ 7
