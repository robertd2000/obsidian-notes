Intersection Observer API (IOA) позволяет приложению асинхронно наблюдать за пересечением элемента (target) с его родителем (root) или областью просмотра (viewport). Другими словами, этот API обеспечивает вызов определенной функции каждый раз при пересечении целевого элемента с root или viewport.  
  
Примеры использования:  
  

- «ленивая» или отложенная загрузка изображений
- бесконечная прокрутка страницы
- получение информации о видимости рекламы для целей расчета стоимости показов
- запуск процесса или анимации, находящихся в поле зрения пользователя

  
  
Для начала работы с IOA необходимо с помощью конструктора создать объект-наблюдатель с двумя параметрами — функцией обратного вызова и настройками:  
  

```
// настройкиlet options = {    root: document.querySelector('.scroll-list'),    rootMargin: '5px',    threshold: 0.5}// функция обратного вызоваlet callback = function(entries, observer){    ...}// наблюдательlet observer = new IntersectionObserver(callback, options)
```

  
Настройки:  
  

- root — элемент, который выступает в роли области просмотра для target (предок целевого элемента или null для viewport)
- rootMargin — отступы вокруг root (margin в CSS, по умолчанию все отступы равны 0)
- threshold — число или массив чисел, указывающий допустимый процент пересечения target и root

  
Далее создается целевой элемент, за которым наблюдает observer:  
  

```
let target = document.querySelector('.list-item')observer.observe(target)
```

  
Вызов callback возвращает объект, содержащий записи об изменениях, произошедших с целевым элементом:  
  

```
let callback = (entries, observer) => {    entries.forEach(entry => {        // entry (запись) - изменение        //   entry.boundingClientRect        //   entry.intersectionRatio        //   entry.intersectionRect        //   entry.isIntersecting        //   entry.rootBounds        //   entry.target        //   entry.time    })}
```

  
В сети полно информации по теории, но довольно мало материалов по практике использования IOA. Я решил немного восполнить этот пробел.  
  

### Примеры

  

#### «Ленивая» (отложенная) загрузка изображений

  
Задача: загружать (показывать) изображения по мере прокрутки страницы пользователем.  
  
Код:  
  

```
// ждем полной загрузки страницыwindow.onload = () => {    // устанавливаем настройки    const options = {        // родитель целевого элемента - область просмотра        root: null,        // без отступов        rootMargin: '0px',        // процент пересечения - половина изображения        threshold: 0.5    }    // создаем наблюдатель    const observer = new IntersectionObserver((entries, observer) => {        // для каждой записи-целевого элемента        entries.forEach(entry => {            // если элемент является наблюдаемым            if (entry.isIntersecting) {                const lazyImg = entry.target                // выводим информацию в консоль - проверка работоспособности наблюдателя                console.log(lazyImg)                // меняем фон контейнера                lazyImg.style.background = 'deepskyblue'                // прекращаем наблюдение                observer.unobserve(lazyImg)            }        })    }, options)    // с помощью цикла следим за всеми img на странице    const arr = document.querySelectorAll('img')    arr.forEach(i => {        observer.observe(i)    })}
```

  
Результат:  
![](https://habrastorage.org/r/w1560/webt/wc/bw/gj/wcbwgjht9d-l_jg9jhvkhrfz8ja.png)  
  
Фон контейнера, выходящего за пределы области просмотра, белый.  
  
![](https://habrastorage.org/r/w1560/webt/oy/4a/il/oy4ail-hjedm98mczcp8q4xqphc.png)  
  
При пересечении с областью просмотра наполовину, фон меняется на небесно-голубой.  
  
→ [Codepen](https://codepen.io/igor_agapov/pen/jOPdEwN)  
  
→ [Github](https://github.com/harryheman/intersection-observer-examples/blob/master/intersection-example-1.html)