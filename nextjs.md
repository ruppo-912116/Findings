## 🧠 `useCallback` vs `useMemo` in React

React provides the `useCallback` and `useMemo` hooks to optimize performance by preventing unnecessary recalculations and re-renders. Understanding **when to use which** is key to writing efficient and clean code.

### 🔁 `useCallback`

- **Purpose**: Memoizes a **function**.
- **Use When**:
  - You pass a function to a **child component** to avoid unnecessary renders.
  - A function is **recreated on every render**, and you want to avoid that for performance.
  - You want **referential equality** of a function between renders.

```js
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

---

### 💾 `useMemo`

- **Purpose**: Memoizes a **value** (usually the result of a computation).
- **Use When**:
  - You perform an **expensive calculation**.
  - You want to **avoid recalculating** a derived value on every render.
  - You want to ensure **referential equality** for derived values (e.g., arrays or objects).

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

---

### 📊 Comparison Table

| Feature              | `useCallback`                          | `useMemo`                             |
|----------------------|-----------------------------------------|----------------------------------------|
| Memoizes             | Function                                | Value                                  |
| Returns              | Memoized function                       | Memoized result of a computation       |
| Main Use Case        | Prevent function recreation             | Avoid expensive recalculations         |
| Helps with           | Function reference stability            | Value reference stability              |
| Typical Use          | Passing stable callbacks to children    | Memoizing derived/calculated values    |
