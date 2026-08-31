function promiseAll(promises) {  
  return new Promise((resolve, reject) => {  
    // Обработка пустого массива  
    if (promises.length === 0) {  
      resolve([]);  
      return;  
    }  
  
    const results = [];  
    let completed = 0;  
  
    promises.forEach((promise, index) => {  
      // Используем Promise.resolve для обработки не-промисов  
      Promise.resolve(promise)  
        .then(result => {  
          results[index] = result; // сохраняем порядок  
          completed++;  
  
          // Резолвим только когда все готовы  
          if (completed === promises.length) {  
            resolve(results);  
          }  
        })  
        .catch(reject); // первая ошибка реджектит всё  
    });  
  });  
}  
  
// Использование:  
// const results = await promiseAll([fetch('/api/1'), fetch('/api/2')]);
