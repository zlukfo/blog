---
title: 
description: 
permalink: 
tags:
  - javascript
draft: true
date:
---
## 1.
Можно вынести данные (переменные, функции и т.д.) в отдельный файл-скрипт js. Если перед объявлением данных добавить `export`, то их можно импортировать в другой файл-скрипт 
**first.js**
```js
export function func(){
	...
}
```
Второй вариант написания - объявить нужные данные а затем указать какие из них могут быть экспортированы
```js
function func () {
...
}
export {func, func2, ...}
```


**second.js**
```js
import {'func'} from './first.js';
...`
```
или  в скрипт внутри html-страницы
**index.html**
```html
<html>
	<body></body>
	<script type="module">
		import {func} from '/script/first.js'
	</script>
</html>
```
Второй вариант импорта
```js
import * as mod from '/script/first.js';
mod.func();_
```
## 2.
Модуль может включать не только объявление данных но и последовательность выполняемых операций. При этом, если модуль загрузить несколько раз - последовательность операций выполнится только единожды.
**firts.js**
```js
console.log("Модуль выполнен")
```
**index.html**
```html
<html>
    <body></body>
    <script type="module">
        import '/script/first.js'
        console.log("Первый блок отаботал")
    </script>
    <script type="module">
        import '/script/first.js'
        console.log("Второй блок отаботал")
    </script>
</html>
```
**результат**
```
Модуль выполнен
Первый блок отаботал
Второй блок отаботал
```

## 3.
Из предыдущего пункта следует одно важное практическое назначение импорта модуля. Первый импорт часто используют для инициализации конфигурации или создания нужной структуры данных. А последующие импорты - для доступа к ним
**first.js**
```js
export let admin = {name: "John"};
```
**index.html**
```html
<html>
    <body></body>
    <script type="module">
        import {admin} from '/script/first.js'
        console.log(admin)
        admin.name = 'Peter'
    </script>
    <script type="module">
        import {admin} from '/script/first.js'
        console.log(admin)
    </script>
</html>
```
**результат**
```
{name: 'John'}
{name: 'Peter'}
```

## 4.
Есть важная особенность загрузки и выполнения модуля (скрипта с параметром `type="module"`).
Модуль всегда выполняется в отложенном режиме - сначала выполняется полная загрузка **_html-страницы_** и только после этого выполняется модуль. Продемонстрировать можно на примере
**first.js**
```js
console.log('Модуль загрузился и выполнился')
```
**index.html**
```html
<html>
    <script type="module">
        import '/script/first.js'
    </script>
    <script src="https://javascript.info/article/script-async-defer/long.js?speed=1"></script>
    <script>
        console.log('Встроенный скрипт выполнен')
    </script>
    <body></body>
</html>
```
Но, если наш модуль не связан с DOM-элементами страницы, он может быть выполнен не дожидаясь полной загрузки html-страницы. Для этого нужно указать параметр `async`
```html
<script async type="module">
...
</script>
```
## 5.
Модули можно загружать динамически, например когда файл модуля заранее не известен и определяется по ходу выполнения скрипта.
Динамический импорт работает в обычных скриптах, он не требует указания `script type="module"`

**first.js**
```js
console.log('Первый модуль загрузился')
```
**second,js**
```js
console.log('Второй модуль загрузился')
```
**index.html**
```html
<html>
    <script>
        let module = `/script/${prompt("Какой модуль загружать?")}.js`;
        function yes (){
            console.log('Успех !!!')
        }
        let no = () => {console.log('Неудача (((')}
        import(module)
          .then(obj  => yes() )
          .catch(err => no())
    </script>
</html>
```

Динамически модули можно загружать внутри асинхронной функции - `let module = await import(modulePath)`


