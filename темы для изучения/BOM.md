## [BOM (Browser Object Model)](https://learn.javascript.ru/browser-environment#bom-browser-object-model)

Объектная модель браузера (Browser Object Model, BOM) – это дополнительные объекты, предоставляемые браузером (окружением), чтобы работать со всем, кроме документа.

Например:

- Объект [navigator](https://developer.mozilla.org/ru/docs/Web/API/Window/navigator) даёт информацию о самом браузере и операционной системе. Среди множества его свойств самыми известными являются: `navigator.userAgent` – информация о текущем браузере, и `navigator.platform` – информация о платформе (может помочь в понимании того, в какой ОС открыт браузер – Windows/Linux/Mac и так далее).
- Объект [location](https://developer.mozilla.org/ru/docs/Web/API/Window/location) позволяет получить текущий URL и перенаправить браузер по новому адресу.

Вот как мы можем использовать объект `location`:

[](https://learn.javascript.ru/browser-environment# "выполнить")

[](https://learn.javascript.ru/browser-environment# "открыть в песочнице")

``` `alert``(`location`.`href`)``;` `// показывает текущий URL` `if` `(``confirm``(``"Перейти на Wikipedia?"``)``)` `{`   location`.`href `=` `"https://wikipedia.org"``;` `// перенаправляет браузер на другой URL` `}` ```

Функции `alert/confirm/prompt` тоже являются частью BOM: они не относятся непосредственно к странице, но представляют собой методы объекта окна браузера для коммуникации с пользователем.

Спецификации

BOM является частью общей [спецификации HTML](https://html.spec.whatwg.org/).

	Да, вы всё верно услышали. Спецификация HTML по адресу [https://html.spec.whatwg.org](https://html.spec.whatwg.org/) не только про «язык HTML» (теги, атрибуты), она также покрывает целое множество объектов, методов и специфичных для каждого браузера расширений DOM. Это всё «HTML в широком смысле». Для некоторых вещей есть отдельные спецификации, перечисленные на [https://spec.whatwg.org](https://spec.whatwg.org/).

