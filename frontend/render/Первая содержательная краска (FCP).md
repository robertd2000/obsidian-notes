**Примечание.** Первая отрисовка контента (FCP) — это важный, ориентированный на пользователя показатель для измерения [воспринимаемой скорости загрузки,](https://web.dev/articles/user-centric-performance-metrics?hl=ru#types_of_metrics) поскольку он отмечает первую точку на временной шкале загрузки страницы, когда пользователь может видеть что-либо на экране. Быстрый FCP помогает убедить пользователя в том, что что-то [происходит](https://web.dev/articles/user-centric-performance-metrics?hl=ru#defining_metrics) .
https://web.dev/articles/fcp?hl=ru
## Что такое ФКП?

First Contentful Paint (FCP) измеряет время с момента первого перехода пользователя на страницу до момента отображения какой-либо части содержимого страницы на экране. Для этой метрики «контент» относится к тексту, изображениям (включая фоновые изображения), элементам `<svg>` или небелым элементам `<canvas>` .

**Ключевой момент** : важно отметить, что FCP включает в себя любое время выгрузки с предыдущей страницы, время установки соединения, время перенаправления и [время до первого байта (TTFB),](https://web.dev/articles/ttfb?hl=ru) которые могут быть существенными при измерении в полевых условиях и могут привести к различиям между полевые и лабораторные измерения.

![Хронология FCP с google.com](https://web.dev/static/articles/fcp/image/fcp-timeline-googlecom.png?hl=ru)

На этой временной шкале загрузки FCP происходит во втором кадре, потому что именно тогда на экране отображаются первые элементы текста и изображения.

На временной шкале загрузки, изображенной на предыдущем изображении, FCP происходит во втором кадре, когда на экран выводятся первые элементы текста и изображения.

Вы заметите, что хотя часть контента и была отрисована, не весь контент отрисован. Это важное различие между _первой_ отрисовкой контента и [_самой большой_ отрисовкой контента (LCP)](https://web.dev/articles/lcp?hl=ru) , целью которой является измерение момента завершения загрузки основного содержимого страницы.

### Что такое хороший показатель FCP?

Чтобы обеспечить хорошее взаимодействие с пользователем, сайты должны стремиться к тому, чтобы первая прорисовка контента составляла **1,8 секунды** или меньше. Чтобы гарантировать достижение этой цели для большинства ваших пользователей, хорошим порогом для измерения является **75-й процентиль** загрузки страниц, сегментированный по мобильным и настольным устройствам.

![Хорошие значения FCP составляют 1,8 секунды или меньше, плохие значения превышают 3,0 секунды, а все, что находится между ними, требует улучшения.](https://web.dev/static/articles/fcp/image/good-fcp-values-are-18-s-421f9e1a2cc56.svg?hl=ru)

Хорошие значения FCP составляют 1,8 секунды или меньше. Плохие значения превышают 3,0 секунды.

## Как измерить FCP

FCP можно измерить [в лаборатории](https://web.dev/articles/user-centric-performance-metrics?hl=ru#lab) или [в полевых условиях](https://web.dev/articles/user-centric-performance-metrics?hl=ru#field) , и он доступен в следующих инструментах:

### Полевые инструменты

- [Статистика PageSpeed](https://pagespeed.web.dev/?hl=ru)
- [Отчет об опыте использования Chrome](https://developer.chrome.com/docs/crux?hl=ru)
- [Search Console (отчет о скорости)](https://webmasters.googleblog.com/2019/11/search-console-speed-report.html)
- [JavaScript-библиотека `web-vitals`](https://github.com/GoogleChrome/web-vitals)

### Лабораторные инструменты

- [Маяк](https://developer.chrome.com/docs/lighthouse/overview?hl=ru)
- [Инструменты разработчика Chrome](https://developer.chrome.com/docs/devtools?hl=ru)
- [Статистика PageSpeed](https://pagespeed.web.dev/?hl=ru)

### Измерьте FCP в JavaScript

Чтобы измерить FCP в JavaScript, вы можете использовать [Paint Timing API](https://w3c.github.io/paint-timing/) . В следующем примере показано, как создать [`PerformanceObserver`](https://developer.mozilla.org/docs/Web/API/PerformanceObserver) , который прослушивает запись `paint` с именем `first-contentful-paint` и регистрирует ее на консоли.

```
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntriesByName('first-contentful-paint')) {
    console.log('FCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'paint', buffered: true});
```

**Предупреждение:** этот код показывает, как записать запись `first-contentful-paint` в консоль, но измерение FCP в JavaScript сложнее. Читайте дальше для более подробной информации.

В предыдущем фрагменте кода зарегистрированная запись `first-contentful-paint` сообщит вам, когда был нарисован первый элемент с содержимым. Однако в некоторых случаях эта запись недействительна для измерения FCP.

В следующем разделе перечислены различия между тем, что сообщает API, и тем, как рассчитывается метрика.

#### Различия между метрикой и API

- API отправит запись `first-contentful-paint` для страниц, загруженных на фоновую вкладку, но эти страницы следует игнорировать при расчете FCP (время первой отрисовки следует учитывать только в том случае, если страница все время находилась на переднем плане).
- API не сообщает о записях `first-contentful-paint` когда страница восстанавливается из [обратного/прямого кэша](https://web.dev/articles/bfcache?hl=ru#impact_on_core_web_vitals) , но в этих случаях следует измерять FCP, поскольку пользователи воспринимают их как отдельные посещения страниц.
- API [может не сообщать о времени отрисовки из iframe из разных источников](https://w3c.github.io/paint-timing/#:%7E:text=cross-origin%20iframes) , но для правильного измерения FCP следует учитывать все кадры. Подкадры могут использовать API для передачи данных о времени отрисовки родительскому кадру для агрегирования.
- API измеряет FCP с момента начала навигации, но для [предварительно обработанных страниц](https://developer.chrome.com/docs/web-platform/prerender-pages?hl=ru) FCP следует измерять с [`activationStart`](https://developer.mozilla.org/docs/Web/API/PerformanceNavigationTiming/activationStart) , поскольку это соответствует времени FCP, ощущаемому пользователем.

Вместо того, чтобы запоминать все эти тонкие различия, разработчики могут использовать [библиотеку JavaScript `web-vitals`](https://github.com/GoogleChrome/web-vitals) для измерения FCP, которая обрабатывает эти различия за вас (где это возможно — обратите внимание, что проблема с iframe не рассматривается):

```
import {onFCP} from 'web-vitals';

// Measure and log FCP as soon as it's available.
onFCP(console.log);
```

Вы можете обратиться к [исходному коду `onFCP()`](https://github.com/GoogleChrome/web-vitals/blob/main/src/onFCP.ts) для получения полного примера измерения FCP в JavaScript.

**Примечание.** В некоторых случаях (например, в iframe из разных источников) невозможно измерить FCP в JavaScript. Подробности смотрите в разделе [ограничений](https://github.com/GoogleChrome/web-vitals#limitations) библиотеки `web-vitals` .

## Как улучшить FCP

Чтобы узнать, как улучшить FCP для конкретного сайта, вы можете запустить аудит производительности Lighthouse и обратить внимание на любые конкретные [возможности](https://developer.chrome.com/docs/lighthouse/performance/?hl=ru#opportunities) или [диагностику,](https://developer.chrome.com/docs/lighthouse/performance/?hl=ru#diagnostics) которые предлагает аудит.

Чтобы узнать, как улучшить FCP в целом (для любого сайта), обратитесь к следующим руководствам по производительности:

- [Устраните ресурсы, блокирующие рендеринг](https://developer.chrome.com/docs/lighthouse/performance/render-blocking-resources?hl=ru)
- [Минимизировать CSS](https://developer.chrome.com/docs/lighthouse/performance/unminified-css?hl=ru)
- [Удалить неиспользуемый CSS](https://developer.chrome.com/docs/lighthouse/performance/unused-css-rules?hl=ru)
- [Удалить неиспользуемый JavaScript](https://developer.chrome.com/docs/lighthouse/performance/unused-javascript?hl=ru)
- [Предварительное подключение к необходимым источникам](https://developer.chrome.com/docs/lighthouse/performance/uses-rel-preconnect?hl=ru)
- [Сокращение времени ответа сервера (TTFB)](https://web.dev/articles/ttfb?hl=ru)
- [Избегайте перенаправления нескольких страниц](https://developer.chrome.com/docs/lighthouse/performance/redirects?hl=ru)
- [Предварительная загрузка запросов ключей](https://developer.chrome.com/docs/lighthouse/performance/uses-rel-preload?hl=ru)
- [Избегайте огромной полезной нагрузки в сети](https://developer.chrome.com/docs/lighthouse/performance/total-byte-weight?hl=ru)
- [Обслуживайте статические ресурсы с помощью эффективной политики кэширования](https://developer.chrome.com/docs/lighthouse/performance/uses-long-cache-ttl?hl=ru)
- [Избегайте чрезмерного размера DOM](https://developer.chrome.com/docs/lighthouse/performance/dom-size?hl=ru)
- [Минимизируйте критическую глубину запроса](https://developer.chrome.com/docs/lighthouse/performance/critical-request-chains?hl=ru)
- [Убедитесь, что текст остается видимым во время загрузки веб-шрифта.](https://developer.chrome.com/docs/lighthouse/performance/font-display?hl=ru)
- [Следите за тем, чтобы количество запросов было небольшим, а размеры переводов — небольшими.](https://developer.chrome.com/docs/lighthouse/performance/resource-summary?hl=ru)