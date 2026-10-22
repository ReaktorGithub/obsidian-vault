function promiseAny(promises) {  
  return new Promise((resolve, reject) => {  
    if (promises.length === 0) {  
      reject(new Error('All promises were rejected'));  
      return;  
    }  
  
    let rejectedCount = 0;  
    const errors = [];  
  
    promises.forEach((promise, index) => {  
      Promise.resolve(promise)  
        .then(resolve) // первый успешный сразу резолвит всё  
        .catch(error => {  
          errors[index] = error;  
          rejectedCount++;  
  
          // Реджектим только если ВСЕ упали  
          if (rejectedCount === promises.length) {  
            reject(new Error('All promises were rejected'));  
          }  
        });  
    });  
  });  
}  
  
// Использование:  
// const result = await promiseAny([  
//   fetch('/api/server1'),  
//   fetch('/api/server2'),  
//   fetch('/api/server3')  
// ]);  
// Вернёт результат от самого быстрого успешного сервера
