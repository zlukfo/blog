---
title: 
description: 
permalink: 
tags:
  - Битрикс24
  - "#javascript"
draft: false
date:
---
Чтобы получить результат запроса по Rest API, например такой

``` javascript
BX24.callMethod("disk.file.get",{id: id_file}, callback)
```
можно пойти двумя путями. 

## Вариант 1 
Третий параметр вызова - callback функция, в которую по завершению на вход передается результат запроса. И далее в теле функции этот результат обрабатывается. Именно такой вариант приведен в качестве примера в документации Б24. Неудобства и проблемы возникают, когда мы хотим вытащить результат запроса за пределы callback. Например, такой код не сработает

``` javascript
let filepath = BX24.callMethod("disk.file.get",{id: id_file}, callback)
console.log(filepath)
```
Потому что в этом потоке команд JS не будет дожидаться когда BX24 обработает REST-запрос и вернет результат, чтобы присвоить его переменной. Выполнение потока команд пойдет дальше и в консоль выведется `undefined`

Нам нужно дождаться получения результата от REST-запроса и только потом пойти дальше. Для этого используются промисы

## Вариант 2

Идея такая. REST-запрос оборачивается в `Promise`

``` javascript
let promiseCallback = (id_file) => {
  return new Promise(
    (resolve, reject) => {
	    // Сам запрос
        BX24.callMethod("disk.file.get",{id: id_file}, function(result){
		    // Тело callback-функции
            if (result.error()) {
                reject(result.error());
            }
            else {
                resolve(result.data()["DOWNLOAD_URL"]);
            }
       })
   }
 )
}
```

Здесь вся соль в теле callback-функции. В ней есть две встроенные в Promise функции `resolve` - вызывается если промис сработал без ошибки и `reject` - если с ошибкой. В качестве параметра этим функциям передается то, что вернет наша функция `promiseCallback`. Т.е. `resolve`  и `reject` - это своеобразные `return`-ы для промисов.

Соответственно, чтобы правильно обернуть REST в промис нам нужно в теле callback
1) прописать условия при которых выполняются `resolve`  и `reject`
2) в параметрах этим функциям передать возвращаемое значение

Использовать этот код можно так

``` javascript
const filepath = await promiseCallback(642)
console.log(filepath)
```

Теперь все работает как надо