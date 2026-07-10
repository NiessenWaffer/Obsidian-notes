# Props

Props are **custom attributes** for passing data from a **parent component** down to a **child component**.

## Declaring Props

```js
app.component('UserCard', {
  props: ['name', 'age'],
  template: `<p>{{ name }} is {{ age }} years old</p>`
})
```

## Passing Props

```html
<UserCard name="Wilcon" :age="25" />
```

**Static vs Dynamic:**
- Static: `name="Wilcon"` (passes a string)
- Dynamic: `:age="25"` or `:user="userObj"` (binds JS expression)

## Props Are Read-Only

Never mutate a prop inside the child. If you need to modify it, emit an event or use a local data copy.

```js
// ❌ Bad
props: ['count'],
mounted() { this.count++ }

// ✅ Good
props: ['count'],
data() {
  return { localCount: this.count }
}
```

## Prop Types & Validation

```js
props: {
  name: {
    type: String,
    required: true
  },
  age: {
    type: Number,
    default: 0,
    validator(value) {
      return value >= 0
    }
  },
  tags: Array,
  onDelete: Function
}
```

## Prop Passing Flow

```
Parent (owns data)
  │
  │  :user="currentUser"
  ▼
Child (receives via props)
  │
  │  $emit('update', newData)
  ▼
Parent (handles event)
```

This is called **one-way data flow** — predictable and easy to debug.

---

**Next:** [[Lifecycle Hooks]]
**Previous:** [[Components]]
