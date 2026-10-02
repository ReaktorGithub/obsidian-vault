`useLayoutEffect` - хук для выполнения эффекта **синхронно после изменения DOM, но до отрисовки браузером экрана**.

`useLayoutEffect` выполняется **после commit React и изменения DOM, но до отрисовки браузером (Paint)**.

Если привязать к **Critical Rendering Path**:

**DOM → CSSOM → Render Tree → Layout → Paint**

В контексте React:

**React Render → Commit (изменение DOM) → `useLayoutEffect` → Browser Paint → `useEffect`**

То есть `useLayoutEffect` блокирует **Paint**, пока его код не выполнится.

