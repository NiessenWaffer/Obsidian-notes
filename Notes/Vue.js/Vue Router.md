# Vue Router

Vue Router is the official routing library for Vue. It maps URL paths to components — enabling multi-page behavior in a Single Page Application (SPA).

## Installation

```bash
npm install vue-router@4
```

## Basic Setup

```js
import { createRouter, createWebHistory } from 'vue-router'
import Home from './views/Home.vue'
import About from './views/About.vue'

const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// In main.js
app.use(router)
```

## `<router-link>` instead of `<a>`

```html
<router-link to="/">Home</router-link>
<router-link to="/about">About</router-link>
```
Use `router-link` — it prevents full page reloads and works with Vue's reactivity.

## `<router-view>` — Where Components Render

```html
<!-- App.vue -->
<header>
  <nav>
    <router-link to="/">Home</router-link>
    <router-link to="/about">About</router-link>
  </nav>
</header>

<main>
  <router-view />
</main>
```

The matched component for the current route renders inside `<router-view>`.

## Dynamic Routes

```js
{ path: '/users/:id', component: User }
```

Access the param inside the component:

```js
this.$route.params.id  // e.g., "42"
```

## Navigation

```js
// In a method
this.$router.push('/about')
this.$router.push({ name: 'user', params: { id: 3 } })
this.$router.go(-1)  // back
```

## Nested Routes

```js
const routes = [
  {
    path: '/dashboard',
    component: Dashboard,
    children: [
      { path: 'stats', component: Stats },
      { path: 'settings', component: Settings }
    ]
  }
]
```

---

**Previous:** [[Lifecycle Hooks]]
