async function asyncForeach(array, callback) {
  for (const item of array) {
    await new Promise(resolve => {
      callback(item, resolve);
    });
  }
}

asyncForeach(['a', 'b', 'c'], function (item, next) {
  setTimeout(function () {
    console.log(item);
    next();
  }, 1000);
});

Способ 2:

```
function asyncForeach(array, callback) {
  const iterator = array[Symbol.iterator]();

  const next = () => {
    const { value, done } = iterator.next();

    if (done) {
      return;
    }

    callback(value, next);
  };

  next();
}
```

