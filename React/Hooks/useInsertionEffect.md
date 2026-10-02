`useInsertionEffect` - специальный хук React для **вставки стилей в DOM до того, как React выполнит layout effects**.

В основном он нужен **библиотекам CSS-in-JS**, а не обычным компонентам.

```
useInsertionEffect(() => {
  const style = document.createElement('style');
  style.textContent = '.button { color: red; }';

  document.head.appendChild(style);

  return () => {
    document.head.removeChild(style);
  };
}, []);
```

Порядок эффектов:

```
useInsertionEffect
        ↓
DOM mutations
        ↓
useLayoutEffect
        ↓
useEffect
```

**На собеседовании:**

> `useInsertionEffect` используется преимущественно CSS-in-JS библиотеками для синхронной вставки стилей до выполнения layout effects. В обычной разработке компонентов применяется редко.

`useInsertionEffect` выполняется **очень рано в commit-фазе React — до `useLayoutEffect`**, чтобы CSS-in-JS библиотеки успели вставить стили до layout.

Упрощённо:

**React Render → DOM mutations → `useInsertionEffect` → `useLayoutEffect` → Paint → `useEffect`**

В контексте **Critical Rendering Path** его задача — **вставить CSS до того, как браузер выполнит Layout/Paint**, чтобы браузер рассчитывал layout уже с актуальными стилями.

Важно: `useInsertionEffect` не является отдельным этапом CRP браузера. Это React-хук, который запускается внутри commit-фазы **перед layout effects**.
