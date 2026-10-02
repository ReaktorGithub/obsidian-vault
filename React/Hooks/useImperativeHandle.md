
`useImperativeHandle` - хук React, который позволяет **управлять тем, что родитель получит через `ref` от дочернего компонента**.

Обычно `ref` дает доступ к DOM-элементу. С `useImperativeHandle` можно вместо этого открыть родителю **свои методы**.

### Пример

```
const Input = forwardRef((props, ref) => {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current.focus();
    },
    clear() {
      inputRef.current.value = '';
    }
  }));

  return <input ref={inputRef} />;
});
```

Родитель:

```
const inputRef = useRef(null);

return (
  <>
    <Input ref={inputRef} />

    <button onClick={() => inputRef.current.focus()}>
      Focus
    </button>

    <button onClick={() => inputRef.current.clear()}>
      Clear
    </button>
  </>
);
```

