function promiseRace(promises) {  
  return new Promise((resolve, reject) => {  
    // Пустой массив - промис висит вечно  
    if (promises.length === 0) {  
      return;  
    }  
  
    // Первый завершившийся промис (успешно или с ошибкой) определяет результат  
    promises.forEach(promise => {  
      Promise.resolve(promise)  
        .then(resolve)   // первый успешный  
        .catch(reject);  // первый с ошибкой  
    });  
  });  
}  
  
// Использование:  
// const fastest = await promiseRace([fetch('/api/1'), fetch('/api/2')]);
