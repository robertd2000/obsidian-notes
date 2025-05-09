По умолчанию CSS рассматривается как ресурс, блокирующий рендеринг, а это означает, что браузер не будет отображать обработанный контент до тех пор, пока не будет создан CSSOM. Следите за тем, чтобы ваш CSS был компактным, доставляйте его как можно быстрее и используйте типы мультимедиа и запросы для разблокировки рендеринга.

При [построении дерева рендеринга](https://web.dev/articles/critical-rendering-path/render-tree-construction?hl=ru) мы увидели, что критический путь рендеринга требует как DOM, так и CSSOM для построения дерева рендеринга. Это создает важные последствия для производительности: **и HTML, и CSS являются ресурсами, блокирующими рендеринг.** HTML очевиден, поскольку без DOM нам нечего было бы отображать, но требования CSS могут быть менее очевидными. Что произойдет, если мы попытаемся отобразить типичную страницу, не блокируя рендеринг с помощью CSS?

### Краткое содержание

- По умолчанию CSS рассматривается как ресурс, блокирующий рендеринг.
- Типы мультимедиа и медиа-запросы позволяют нам помечать некоторые ресурсы CSS как неблокирующие рендеринг.
- Браузер загружает все ресурсы CSS независимо от блокирующего или неблокирующего поведения.

![Нью-Йорк Таймс с CSS](https://web.dev/static/articles/critical-rendering-path/render-blocking-css/image/nytimes-css-de9a8a85f18fa.png?hl=ru)

Нью-Йорк Таймс с CSS

![Нью-Йорк Таймс без CSS](https://web.dev/static/articles/critical-rendering-path/render-blocking-css/image/nytimes-without-css-6d823cc9c8ea4.png?hl=ru)

The New York Times без CSS (FOUC)

Приведенный выше пример, показывающий веб-сайт NYTimes с CSS и без него, демонстрирует, почему рендеринг блокируется до тех пор, пока CSS не доступен — без CSS страница практически непригодна для использования. Опыт справа часто называют «Вспышкой нестилизованного контента» (FOUC). Браузер блокирует рендеринг до тех пор, пока у него не будет и DOM, и CSSOM.

> **_CSS — это ресурс, блокирующий рендеринг. Передайте его клиенту как можно скорее и быстрее, чтобы оптимизировать время первого рендеринга._**

Однако что, если у нас есть некоторые стили CSS, которые используются только при определенных условиях, например, когда страница печатается или проецируется на большой монитор? Было бы хорошо, если бы нам не приходилось блокировать рендеринг на этих ресурсах.

CSS «медиа-типы» и «медиа-запросы» позволяют нам решать следующие варианты использования:

```
<link href="style.css" rel="stylesheet" />
<link href="print.css" rel="stylesheet" media="print" />
<link href="other.css" rel="stylesheet" media="(min-width: 40em)" />
```

[Медиа-запрос](https://web.dev/learn/design/media-queries?hl=ru) состоит из типа мультимедиа и нуля или более выражений, которые проверяют условия определенных медиа-функций. Например, наше первое объявление таблицы стилей не предоставляет тип носителя или запрос, поэтому оно применяется во всех случаях; то есть это всегда блокировка рендеринга. С другой стороны, второе объявление таблицы стилей применяется только тогда, когда содержимое печатается — возможно, вы хотите изменить макет, изменить шрифты и т. д., и, следовательно, этому объявлению таблицы стилей не нужно блокировать рендеринг страницу при первой загрузке. Наконец, последнее объявление таблицы стилей предоставляет «медиа-запрос», который выполняется браузером: если условия совпадают, браузер блокирует рендеринг до тех пор, пока таблица стилей не будет загружена и обработана.

Используя медиа-запросы, мы можем адаптировать нашу презентацию к конкретным случаям использования, например отображению или печати, а также к динамическим условиям, таким как изменения ориентации экрана, события изменения размера и многое другое. **При объявлении ресурсов таблицы стилей обратите пристальное внимание на тип мультимедиа и запросы; они сильно влияют на производительность критического пути рендеринга.**

Давайте рассмотрим несколько практических примеров:

```
<link href="style.css" rel="stylesheet" />
<link href="style.css" rel="stylesheet" media="all" />
<link href="portrait.css" rel="stylesheet" media="orientation:portrait" />
<link href="print.css" rel="stylesheet" media="print" />
```

- Первое объявление блокирует рендеринг и соответствует всем условиям.
- Второе объявление также является блокировкой рендеринга: «all» — это тип по умолчанию, поэтому, если вы не укажете какой-либо тип, ему неявно будет установлено значение «all». Следовательно, первое и второе объявления фактически эквивалентны.
- Третье объявление содержит динамический медиа-запрос, который оценивается при загрузке страницы. В зависимости от ориентации устройства во время загрузки страницы Portrait.css может блокировать или не блокировать рендеринг.
- Последнее объявление применяется только во время печати страницы, поэтому оно не блокирует рендеринг при первой загрузке страницы в браузере.

Наконец, обратите внимание, что «блокировка рендеринга» относится только к тому, должен ли браузер поддерживать первоначальный рендеринг страницы на этом ресурсе. В любом случае браузер по-прежнему загружает ресурс CSS, хотя и с более низким приоритетом для неблокирующих ресурсов.