function promiseAllSettled(promises) {  
  return new Promise((resolve) => {  
    // Пустой массив  
    if (promises.length === 0) {  
      resolve([]);  
      return;  
    }  
  
    const results = [];  
    let completed = 0;  
  
    promises.forEach((promise, index) => {  
      Promise.resolve(promise)  
        .then(value => {  
          results[index] = { status: 'fulfilled', value };  
          completed++;  
  
          if (completed === promises.length) {  
            resolve(results);  
          }  
        })  
        .catch(reason => {  
          results[index] = { status: 'rejected', reason };  
          completed++;  
  
          if (completed === promises.length) {  
            resolve(results);  
          }  
        });  
    });  
  });  
}  
  
// Использование:  
// const results = await promiseAllSettled([fetch('/api/1'), fetch('/api/2')]);
