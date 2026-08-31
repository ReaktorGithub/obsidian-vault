function asyncTimeout(fn, delay) {  
  return async function(...args) {  
    return Promise.race([  
      fn(...args),  
      new Promise((_, reject) =>  
        setTimeout(() => reject(new Error('Timeout exceeded')), delay)  
      )  
    ]);  
  };  
}  
  
// Альтернативное решение с AbortController (более продвинутое):  
// function asyncTimeout(fn, delay) {  
//   return async function(...args) {  
//     const controller = new AbortController();  
//     const timeoutId = setTimeout(() => controller.abort(), delay);  
//     //     try {  
//       const result = await fn(...args, { signal: controller.signal });  
//       clearTimeout(timeoutId);  
//       return result;  
//     } catch (error) {  
//       clearTimeout(timeoutId);  
//       if (error.name === 'AbortError') {  
//         throw new Error('Timeout exceeded');  
//       }  
//       throw error;  
//     }  
//   };  
// }
