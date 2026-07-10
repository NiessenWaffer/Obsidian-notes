# The Options API Shape

**Date:** 2026-07-10
**Prerequisite:** [[03 - Reactivity]]
**Next:** [[05 - Template Bindings]]

---

## Why Separate Pieces?

After [[03 - Reactivity]], you know Vue tracks data changes and updates the DOM. But where do you put your logic? Why isn't everything just in one object?

The Options API organizes your component into clear sections:

```js
export default {
  data() {
    return {
      count: 0,
      items: []
    }
  },
  computed: {
    doubleCount() {
      return this.count * 2
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  watch: {
    count(newVal, oldVal) {
      console.log(`Count changed from ${oldVal} to ${newVal}`)
    }
  }
}
```

Each section has a **specific job**:

| Section | Job | Runs when? |
|---------|-----|------------|
| `data` | Reactive state | Once, on component creation |
| `computed` | Derived values (cached) | When dependencies change |
| `methods` | Functions (on demand) | When you call them |
| `watch` | Side effects | When watched property changes |

---

## How `this` Ties Everything Together

Inside a Vue component, `this` refers to the component instance. That's why you can write `this.count` in a method — it reaches into `data()` and grabs the reactive `count`.

```js
export default {
  data() {
    return {
      firstName: 'Niesen',
      lastName: 'Wilcon'
    }
  },
  computed: {
    fullName() {
      // 'this' gives access to everything in the component
      return this.firstName + ' ' + this.lastName
    }
  },
  methods: {
    greet() {
      // 'this' works the same way here
      alert('Hello, ' + this.fullName)
    }
  }
}
```

**Rule:** You never import or reference your own data/computed/methods — `this` connects them all.

---

## Where to Put Logic (When You're Staring at a Blank `.vue` File)

This is the practical part. When you're building something in your Laravel/Vue/Buefy project, here's how to decide:

### "I need to store data the user can see or change"
→ `data()`

```js
data() {
  return {
    showModal: false,
    selectedItem: null,
    searchQuery: ''
  }
}
```

### "I need a value derived from other data"
→ `computed`

```js
computed: {
  filteredItems() {
    return this.items.filter(item =>
      item.name.includes(this.searchQuery)
    )
  }
}
```

### "I need to do something when the user clicks or submits"
→ `methods`

```js
methods: {
  async saveItem() {
    await axios.post('/api/items', this.item)
    this.showModal = false
  }
}
```

### "I need to react when a piece of data changes"
→ `watch`

```js
watch: {
  searchQuery(newVal) {
    // Maybe you need to call an API when search changes
    this.fetchResults(newVal)
  }
}
```

---

## Real Example: Your Approval Workflow

Here's how a typical Vue component in your Laravel/Vue/Buefy project might be structured:

```js
export default {
  data() {
    return {
      approvals: [],
      loading: false,
      selectedStatus: 'pending',
      showModal: false,
      currentApproval: null
    }
  },

  computed: {
    filteredApprovals() {
      return this.approvals.filter(a => a.status === this.selectedStatus)
    },
    hasApprovals() {
      return this.filteredApprovals.length > 0
    }
  },

  methods: {
    async fetchApprovals() {
      this.loading = true
      try {
        const response = await axios.get('/api/approvals')
        this.approvals = response.data
      } catch (error) {
        console.error(error)
      } finally {
        this.loading = false
      }
    },

    openApproval(approval) {
      this.currentApproval = approval
      this.showModal = true
    },

    async approveItem(id) {
      await axios.post(`/api/approvals/${id}/approve`)
      await this.fetchApprovals()
      this.showModal = false
    }
  },

  watch: {
    selectedStatus() {
      // Reactivity in action — status changes, computed recalculates
    }
  },

  created() {
    this.fetchApprovals()
  }
}
```

**Notice the flow:**
1. `data()` holds the state (approvals, loading, modal visibility)
2. `computed` derives filtered lists based on state
3. `methods` handles user actions (fetch, open, approve)
4. `watch` could react to specific changes (like search input)
5. `created()` runs the initial API call

---

## The Mental Model

Think of a Vue component as a **self-contained unit**:

```
┌─────────────────────────────────────┐
│           Component                 │
│                                     │
│  data() ─────► What exists         │
│      │                              │
│      ▼                              │
│  computed ───► What's derived      │
│      │                              │
│      ▼                              │
│  methods ────► What happens        │
│      │                              │
│      ▼                              │
│  watch ──────► What reacts         │
│                                     │
└─────────────────────────────────────┘
```

Everything inside the component can access everything else via `this`. No imports, no globals — just `this.data`, `this.computed`, `this.methods`.

---

## Common Mistake: Mixing Up Sections

```js
export default {
  data() {
    return {
      items: []
    }
  },
  // ❌ WRONG: Putting logic in data that should be in computed
  data() {
    return {
      items: [],
      filteredItems: this.items.filter(i => i.active)  // NOT REACTIVE, DON'T DO THIS
    }
  }
}
```

**Fix:**
```js
export default {
  data() {
    return {
      items: []
    }
  },
  computed: {
    filteredItems() {
      return this.items.filter(i => i.active)  // ✅ Reactive, recalculates when items change
    }
  }
}
```

---

## Summary

| You need to... | Put it in... | Why |
|----------------|--------------|-----|
| Store reactive state | `data()` | Vue tracks it automatically |
| Derive values from state | `computed` | Cached, recalculates when dependencies change |
| Handle user actions | `methods` | Runs on demand, no caching |
| React to specific changes | `watch` | Side effects when a property changes |
| Access other options | Use `this` | `this` connects everything in the component |

---

## Connection to Your Real Work

Every time you write a Vue component for your Laravel app, you're using this shape:

- **Data tables:** `data()` holds the rows, `computed` filters/sorts them, `methods` handles pagination
- **Forms:** `data()` holds form fields, `methods` submits, `computed` validates
- **Modals:** `data()` holds `showModal`, `methods` opens/closes, `computed` determines content

Understanding the shape means you'll always know where to put new logic.

---

## Research Task (Before Lesson 3)

**Look at one of your existing Vue components** (maybe from your Item Importation or approval workflow).

Try to identify:
- What's in `data()` that should be in `computed`?
- What's in `methods` that could be in `computed`?
- Are there any functions that are just storing data that could be in `data()` instead?

Don't change anything yet — just observe. When you come back, we'll move to [[05 - Template Bindings]].
