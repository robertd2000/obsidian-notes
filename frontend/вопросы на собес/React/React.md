
##### зачем нужен
React — это популярная JavaScript-библиотека для создания пользовательских интерфейсов (UI), разработанная Facebook (Meta). Вот зачем он нужен и какие проблемы решает:

---

###### 🔥 **Основные причины использовать React**
1. **Компонентный подход**  
   - Разбиение интерфейса на переиспользуемые компоненты (кнопки, формы, списки).  
   - Пример:  
     ```jsx
     function Button({ children }) {
       return <button className="btn">{children}</button>;
     }
     ```

2. **Вирутальный DOM**  
   - React эффективно обновляет только изменившиеся части интерфейса, а не всю страницу.  
   - Это ускоряет работу приложений.

3. **Одностраничные приложения (SPA)**  
   - Плавные переходы между "страницами" без перезагрузки.  
   - Интеграция с React Router:  
     ```jsx
     <Routes>
       <Route path="/" element={<Home />} />
       <Route path="/about" element={<About />} />
     </Routes>
     ```

4. **Управление состоянием**  
   - Локальное состояние (`useState`):  
     ```jsx
     const [count, setCount] = useState(0);
     ```  
   - Глобальное состояние (Redux, MobX, Context API).

5. **Богатая экосистема**  
   - React Native (мобильные приложения), Next.js (SSR), Gatsby (статические сайты).

---

###### 🛠 **Какие проблемы решает React?**
| Проблема | Решение в React |
|----------|----------------|
| Сложность обновления DOM | Виртуальный DOM |
| Повторяющийся код | Компоненты + JSX |
| Медленная разработка | Готовые UI-библиотеки (MUI, Ant Design) |
| Плохая структура проекта | Четкие правила (хуки, компоненты) |

---

###### 🌟 **Где используют React?**
1. **Соцсети**: Facebook, Instagram, Twitter (частично)
2. **Стриминги**: Netflix, Disney+
3. **Аналитика**: Airbnb, Uber
4. **Админки** и SaaS-продукты

---

###### 💡 **Пример кода: счетчик**
```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Вы кликнули {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми меня
      </button>
    </div>
  );
}
```
- `useState` — хук для управления состоянием.
- JSX — синтаксис для описания UI.

---

###### 📊 **Плюсы и минусы**
| ✅ Плюсы | ❌ Минусы |
|----------|----------|
| Быстрая разработка | Кривая обучения (хуки, JSX) |
| Большое комьюнити | Требует дополнительных библиотек для роутинга (React Router) |
| Подходит для сложных UI | Не SEO-дружественный из коробки (решается Next.js) |

---

###### 🚀 **Когда выбрать React?**
- Если нужно **динамическое SPA** (соцсети, дашборды).
- Для **переиспользуемых компонентов** в крупных проектах.
- Когда важна **производительность** и горячее обновление (HMR).

Для простых лендингов лучше подойдут статические генераторы (Gatsby, Astro).
##### виртуальный ДОМ
https://reactdev.ru/archive/react16/faq-internals/
https://ru.legacy.reactjs.org/docs/faq-internals.html
https://habr.com/ru/companies/macloud/articles/558682/

Виртуальный DOM (VDOM) — это концепция программирования, в которой идеальное или «виртуальное» представление пользовательского интерфейса хранится в памяти и синхронизируется с «настоящим» DOM при помощи библиотеки, такой как ReactDOM. Этот процесс называется [согласованием](https://reactdev.ru/archive/react16/reconciliation/).

Такой подход и делает API React декларативным: вы указываете, в каком состоянии должен находиться пользовательский интерфейс, а React добивается, чтобы DOM соответствовал этому состоянию. Это абстрагирует манипуляции с атрибутами, обработку событий и ручное обновление DOM, которые в противном случае пришлось бы использовать при разработке приложения.

Поскольку «виртуальный DOM» – это скорее паттерн, чем конкретная технология, этим термином иногда обозначают разные понятия. В мире React «виртуальный DOM» обычно ассоциируется с [React-элементами](https://reactdev.ru/archive/react16/rendering-elements/), поскольку они являются объектами, представляющими пользовательский интерфейс. Тем не менее, React также использует внутренние объекты, называемые «волокнами» (fibers), чтобы хранить дополнительную информацию о дереве компонентов. Их также можно считать частью реализации «виртуального DOM» в React.

###### Виртуальный DOM (Virtual DOM) в React

**Виртуальный DOM** — это ключевая оптимизационная концепция React, которая делает обновления интерфейса быстрыми и эффективными. Вот как это работает:

---

###### 🌟 **Основная идея**
React создает легковесную **"виртуальную" копию реального DOM** в памяти. При изменениях:
1. Сначала обновляется Virtual DOM
2. Затем React **сравнивает** (diffing) новую и предыдущую версии Virtual DOM
3. В реальный DOM вносятся **только необходимые изменения** (reconciliation)

```jsx
// Пример: React обновляет только текст, а не всю кнопку
function Counter() {
  const [count, setCount] = useState(0);
  return <button>Clicked {count} times</button>;
}
```

---

###### 🔍 **Как именно это работает?**
1. **Создание Virtual DOM**  
   При рендере компонента React строит дерево объектов (Virtual DOM) — упрощенное представление реального DOM.

2. **Обновление при изменениях**  
   При изменении состояния (`setState`) создается **новая версия** Virtual DOM.

3. **Diffing (сравнение)**  
   React определяет разницу между старой и новой версиями с помощью алгоритма **Reconciliation**.

4. **Минимальные обновления**  
   В реальный DOM вносятся **только те изменения**, которые действительно необходимы.

```javascript
// Псевдокод: как может выглядеть Virtual DOM
const virtualDom = {
  type: 'div',
  props: { className: 'header' },
  children: [
    { type: 'h1', props: {}, children: 'Hello' }
  ]
};
```

---

###### ⚡ **Преимущества Virtual DOM**
| **Плюс** | **Пример** |
|----------|-----------|
| ⚡ Быстрота | Не тратится время на прямое обновление тяжелого DOM |
| 🧩 Эффективность | Изменяются только нужные элементы (не весь документ) |
| 🛡 Стабильность | Меньше риска ошибок при ручном управлении DOM |

---

###### 🆚 **Virtual DOM vs Real DOM**
| **Критерий** | **Real DOM** | **Virtual DOM** |
|--------------|-------------|-----------------|
| Вес | Тяжелый (полное дерево) | Легкий (только JS-объекты) |
| Обновление | Медленное (перерисовка) | Быстрое (diffing) |
| Управление | Вручную | Автоматически через React |

```javascript
// Реальный DOM (медленный)
document.getElementById('app').innerHTML = '<div>New content</div>';

// Virtual DOM (быстрый)
ReactDOM.render(<App />, document.getElementById('root'));
```

---

###### 🛠 **Пример оптимизации**
Без Virtual DOM при изменении состояния перерисовывался бы **весь интерфейс**:
```html
<!-- Было -->
<div class="header">
  <h1>Hello</h1>
</div>

<!-- Стало бы после обновления (даже если изменился только текст) -->
<div class="header">
  <h1>Hello World</h1>
</div>
```

С Virtual DOM React поймет, что изменился только текст внутри `<h1>`, и обновит **именно его**.

---

###### 💡 **Когда Virtual DOM не нужен?**
1. Для **статичных сайтов** (лендингов) — избыточно.
2. В **простых приложениях** — можно использовать нативный DOM.
3. Если нужна **максимальная производительность** (в этом случае лучше WebAssembly или Canvas).

---

###### **Как React сравнивает изменения?** (Reconciliation)
1. **Разные типы элементов** → Полная перестройка дерева.  
   ```jsx
   // Будет полное обновление:
   <div>
     <Counter />
   </div>
   → 
   <span>
     <Counter />
   </span>
   ```

2. **Одинаковые типы** → Обновляются только атрибуты.  
   ```jsx
   <div className="old" />
   → 
   <div className="new" />  // Обновится только класс
   ```

3. **Ключи (`key`)** — помогают React определять, какие элементы изменились в списках.

---

###### 📊 **Производительность Virtual DOM**
- **Обычные приложения**: Виртуальный DOM дает заметный прирост скорости.
- **Сложные UI**: Возможны задержки (решается через `React.memo`, `useMemo`).
- **Экстремальные случаи**: Для анимаций/игр иногда лучше нативный DOM.

```jsx
// Оптимизация через React.memo
const MemoButton = React.memo(function Button({ children }) {
  return <button>{children}</button>;
});
```

---

###### **Итог**
Virtual DOM — это **"умный посредник"** между вашим кодом и браузерным DOM, который:
- 🚀 Ускоряет рендеринг
- 🛠 Упрощает разработку
- 🔄 Автоматизирует обновления

Без него современные SPA-приложения были бы намного медленнее!
##### коммит/рендер

<big>Прежде чем ваши компоненты будут отображены на экране, они должны быть отрисованы React. Понимание этапов этого процесса поможет вам задуматься о том, как выполняется ваш код, и объяснить его поведение.</big>

!!!tip "Вы узнаете"

    -   Что означает рендеринг в React
    -   Когда и почему React отображает компонент
    -   Шаги, связанные с отображением компонента на экране
    -   Почему рендеринг не всегда приводит к обновлению DOM

Представьте, что ваши компоненты - это повара на кухне, собирающие вкусные блюда из ингредиентов. В этом сценарии React - это официант, который выполняет запросы клиентов и приносит им их заказы. Этот процесс запроса и подачи пользовательского интерфейса состоит из трех этапов:

1.  **Триггирование** рендеринга (доставка заказа гостя на кухню).
2.  **Рендеринг** компонента (подготовка заказа на кухне)
3.  **Коммитирование** в DOM (размещение заказа на столе)

###### Шаг 1: Запуск рендеринга {#step-1-trigger-a-render}

Есть две причины для рендеринга компонента:

1.  Это **начальный рендеринг компонента**.
2.  Состояние компонента (или одного из его предков) **было обновлено**.

###### Начальный рендер {#initial-render}

Когда ваше приложение запускается, вам необходимо вызвать начальный рендеринг. Фреймворки и песочницы иногда скрывают этот код, но он выполняется вызовом [`createRoot`](../reference/react-dom/client/createRoot.md) с целевым узлом DOM, а затем вызовом его метода `render` с вашим компонентом:

=== "index.js"

    ```js
    import Image from './Image.js';
    import { createRoot } from 'react-dom/client';

    const root = createRoot(document.getElementById('root'));
    root.render(<Image />);
    ```

=== "Image.js"

    ```js
    export default function Image() {
    	return (
    		<img
    			src="https://i.imgur.com/ZF6s192.jpg"
    			alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    		/>
    	);
    }
    ```

=== "CodeSandbox"

    <iframe src="https://codesandbox.io/embed/lvpwwg?view=Editor+%2B+Preview" style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;" title="react.dev" allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking" sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts" ></iframe>

Попробуйте закомментировать вызов `root.render()` и увидите, как компонент исчезнет!

###### Рендеринг при обновлении состояния {#re-renders-when-state-updates}

После того, как компонент был первоначально отрисован, вы можете инициировать дальнейшие рендеры, обновляя его состояние с помощью функции [`set`](../reference/react/useState.md) Обновление состояния компонента автоматически ставит его в очередь на рендер. (Вы можете представить это как посетитель ресторана, который после первого заказа заказывает чай, десерт и всевозможные вещи, в зависимости от состояния его жажды или голода).

###### Шаг 2: React рендерит ваши компоненты {#step-2-react-renders-your-components}

После запуска рендеринга React вызывает ваши компоненты, чтобы определить, что отобразить на экране. **"Рендеринг" - это обращение React к вашим компонентам.**

-   **При первом рендере** React вызывает корневой компонент.
-   **При последующих рендерах** React будет вызывать функциональный компонент, обновление состояния которого вызвало рендер.

Этот процесс рекурсивен: если обновленный компонент возвращает какой-то другой компонент, React будет рендерить _этот_ компонент следующим, и если этот компонент тоже что-то возвращает, он будет рендерить _этот_ компонент следующим, и так далее. Этот процесс будет продолжаться до тех пор, пока не останется вложенных компонентов и React не будет точно знать, что должно быть отображено на экране.

В следующем примере React вызовет `Gallery()` и `Image()` несколько раз:

"Gallery.js"

    ```js
    export default function Gallery() {
    	return (
    		<section>
    			<h1>Inspiring Sculptures</h1>
    			<Image />
    			<Image />
    			<Image />
    		</section>
    	);
    }

    function Image() {
    	return (
    		<img
    			src="https://i.imgur.com/ZF6s192.jpg"
    			alt="'Floralis Genérica' by Eduardo Catalano"
    		/>
    	);
    }
    ```

=== "index.js"

    ```js
    import Gallery from './Gallery.js';
    import { createRoot } from 'react-dom/client';

    const root = createRoot(document.getElementById('root'));
    root.render(<Gallery />);
    ```

=== "CodeSandbox"

    <iframe src="https://codesandbox.io/embed/5zgl9d?view=Editor+%2B+Preview&module=%2Fsrc%2FGallery.js" style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;" title="react.dev" allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking" sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts" ></iframe>

-   **Во время первоначального рендеринга** React [создаст DOM-узлы](https://developer.mozilla.org/docs/Web/API/Document/createElement) для тегов `<section>`, `<h1>` и трех `<img>`.
-   Во время повторного рендеринга React вычислит, какие из их свойств, если таковые имеются, изменились с момента предыдущего рендеринга. Он ничего не будет делать с этой информацией до следующего шага, фазы фиксации.

!!!warning "Внимание"

    Рендеринг всегда должен быть [чистым вычислением](keeping-components-pure.md):

    -   **Одинаковые параметры, одинаковый результат.** При одинаковых параметрах компонент всегда должен возвращать одинаковый JSX. (Когда кто-то заказывает салат с помидорами, он не должен получить салат с луком!)
    -   **Он занимается своими делами.** Он не должен изменять никакие объекты или переменные, существовавшие до рендеринга. (Один заказ не должен изменять другой заказ).

    В противном случае вы можете столкнуться с непонятными ошибками и непредсказуемым поведением по мере усложнения вашей кодовой базы. При разработке в "строгом режиме" React вызывает функцию каждого компонента дважды, что может помочь выявить ошибки, вызванные нечистыми функциями.

!!!note "Оптимизация производительности"

    Поведение по умолчанию, при котором отображаются все компоненты, вложенные в обновленный компонент, не является оптимальным с точки зрения производительности, если обновленный компонент находится очень высоко в дереве. Если вы столкнулись с проблемой производительности, есть несколько способов ее решения, описанных в разделе [Оптимизация производительности](https://ru.legacy.reactjs.org/docs/optimizing-performance.html).

    **Не оптимизируйте преждевременно!**

###### Шаг 3: React фиксирует изменения в DOM {#step-3-react-commits-changes-to-the-dom}

После рендеринга (вызова) ваших компонентов React изменит DOM.

-   **Для первоначального рендеринга,** React будет использовать [`appendChild()`](https://developer.mozilla.org/docs/Web/API/Node/appendChild) DOM API для размещения всех созданных им узлов DOM на экране.
-   **Для повторного рендеринга,** React будет применять минимально необходимые операции (вычисляемые во время рендеринга!), чтобы DOM соответствовал последнему выводу рендеринга.

**React изменяет узлы DOM, только если есть разница между рендерами.** Например, вот компонент, который рендерится с различными пропсами, передаваемыми от его родителя каждую секунду. Обратите внимание, как вы можете добавить текст в `<input>`, обновляя его `value`, но текст не исчезает при повторном рендеринге компонента:

=== "Clock.js"

    ```js
    export default function Clock({ time }) {
    	return (
    		<>
    			<h1>{time}</h1>
    			<input />
    		</>
    	);
    }
    ```

=== "CodeSandbox"

    <iframe src="https://codesandbox.io/embed/xlwst2?view=Editor+%2B+Preview&module=%2Fsrc%2FClock.js" style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;" title="react.dev" allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking" sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts" ></iframe>

Это работает, потому что во время последнего шага React только обновляет содержимое `<h1>` с новым `time`. Он видит, что `<input>` появляется в JSX в том же месте, что и в прошлый раз, поэтому React не трогает `<input>` или его `value`!

###### Браузерная отрисовка {#epilogue-browser-paint}

После того как рендеринг завершен и React обновил DOM, браузер перерисовывает экран. Хотя этот процесс известен как "браузерный рендеринг", мы будем называть его "рисованием", чтобы избежать путаницы в документации.

!!!note "Итого"

    -   Любое обновление экрана в приложении React происходит в три этапа:
    	1.  Триггер
    	2.  Рендеринг
    	3.  Коммит
    -   Вы можете использовать режим Strict Mode для поиска ошибок в ваших компонентах
    -   React не трогает DOM, если результат рендеринга такой же, как и в прошлый раз

##### состояние

##### useMemo, useCallback, memo

##### useEffect

##### context

##### состояние - что будет если передать функцию в useState

##### React Fiber Architecture
  https://github.com/acdlite/react-fiber-architecture

###### Introduction

React Fiber is an ongoing reimplementation of React's core algorithm. It is the culmination of over two years of research by the React team.

The goal of React Fiber is to increase its suitability for areas like animation, layout, and gestures. Its headline feature is **incremental rendering**: the ability to split rendering work into chunks and spread it out over multiple frames.

Other key features include the ability to pause, abort, or reuse work as new updates come in; the ability to assign priority to different types of updates; and new concurrency primitives.
 ###### About this document

Fiber introduces several novel concepts that are difficult to grok solely by looking at code. This document began as a collection of notes I took as I followed along with Fiber's implementation in the React project. As it grew, I realized it may be a helpful resource for others, too.

I'll attempt to use the plainest language possible, and to avoid jargon by explicitly defining key terms. I'll also link heavily to external resources when possible.

Please note that I am not on the React team, and do not speak from any authority. **This is not an official document**. I have asked members of the React team to review it for accuracy.

This is also a work in progress. **Fiber is an ongoing project that will likely undergo significant refactors before it's completed.** Also ongoing are my attempts at documenting its design here. Improvements and suggestions are highly welcome.

My goal is that after reading this document, you will understand Fiber well enough to [follow along as it's implemented](https://github.com/facebook/react/commits/master/src/renderers/shared/fiber), and eventually even be able to contribute back to React.

###### Prerequisites

I strongly suggest that you are familiar with the following resources before continuing:

- [React Components, Elements, and Instances](https://facebook.github.io/react/blog/2015/12/18/react-components-elements-and-instances.html) - "Component" is often an overloaded term. A firm grasp of these terms is crucial.
- [Reconciliation](https://facebook.github.io/react/docs/reconciliation.html) - A high-level description of React's reconciliation algorithm.
- [React Basic Theoretical Concepts](https://github.com/reactjs/react-basic) - A description of the conceptual model of React without implementation burden. Some of this may not make sense on first reading. That's okay, it will make more sense with time.
- [React Design Principles](https://facebook.github.io/react/contributing/design-principles.html) - Pay special attention to the section on scheduling. It does a great job of explaining the _why_ of React Fiber.

###### Review

Please check out the prerequisites section if you haven't already.

Before we dive into the new stuff, let's review a few concepts.

###### What is reconciliation?
reconciliation

The algorithm React uses to diff one tree with another to determine which parts need to be changed.

update

A change in the data used to render a React app. Usually the result of `setState`. Eventually results in a re-render.

The central idea of React's API is to think of updates as if they cause the entire app to re-render. This allows the developer to reason declaratively, rather than worry about how to efficiently transition the app from any particular state to another (A to B, B to C, C to A, and so on).

Actually re-rendering the entire app on each change only works for the most trivial apps; in a real-world app, it's prohibitively costly in terms of performance. React has optimizations which create the appearance of whole app re-rendering while maintaining great performance. The bulk of these optimizations are part of a process called **reconciliation**.

Reconciliation is the algorithm behind what is popularly understood as the "virtual DOM." A high-level description goes something like this: when you render a React application, a tree of nodes that describes the app is generated and saved in memory. This tree is then flushed to the rendering environment — for example, in the case of a browser application, it's translated to a set of DOM operations. When the app is updated (usually via `setState`), a new tree is generated. The new tree is diffed with the previous tree to compute which operations are needed to update the rendered app.

Although Fiber is a ground-up rewrite of the reconciler, the high-level algorithm [described in the React docs](https://facebook.github.io/react/docs/reconciliation.html) will be largely the same. The key points are:

- Different component types are assumed to generate substantially different trees. React will not attempt to diff them, but rather replace the old tree completely.
- Diffing of lists is performed using keys. Keys should be "stable, predictable, and unique."

###### Reconciliation versus rendering

The DOM is just one of the rendering environments React can render to, the other major targets being native iOS and Android views via React Native. (This is why "virtual DOM" is a bit of a misnomer.)

The reason it can support so many targets is because React is designed so that reconciliation and rendering are separate phases. The reconciler does the work of computing which parts of a tree have changed; the renderer then uses that information to actually update the rendered app.

This separation means that React DOM and React Native can use their own renderers while sharing the same reconciler, provided by React core.

Fiber reimplements the reconciler. It is not principally concerned with rendering, though renderers will need to change to support (and take advantage of) the new architecture.

###### Scheduling

scheduling

the process of determining when work should be performed.

work

any computations that must be performed. Work is usually the result of an update (e.g. `setState`).

React's [Design Principles](https://facebook.github.io/react/contributing/design-principles.html#scheduling) document is so good on this subject that I'll just quote it here:

> In its current implementation React walks the tree recursively and calls render functions of the whole updated tree during a single tick. However in the future it might start delaying some updates to avoid dropping frames.
> 
> This is a common theme in React design. Some popular libraries implement the "push" approach where computations are performed when the new data is available. React, however, sticks to the "pull" approach where computations can be delayed until necessary.
> 
> React is not a generic data processing library. It is a library for building user interfaces. We think that it is uniquely positioned in an app to know which computations are relevant right now and which are not.
> 
> If something is offscreen, we can delay any logic related to it. If data is arriving faster than the frame rate, we can coalesce and batch updates. We can prioritize work coming from user interactions (such as an animation caused by a button click) over less important background work (such as rendering new content just loaded from the network) to avoid dropping frames.

The key points are:

- In a UI, it's not necessary for every update to be applied immediately; in fact, doing so can be wasteful, causing frames to drop and degrading the user experience.
- Different types of updates have different priorities — an animation update needs to complete more quickly than, say, an update from a data store.
- A push-based approach requires the app (you, the programmer) to decide how to schedule work. A pull-based approach allows the framework (React) to be smart and make those decisions for you.

React doesn't currently take advantage of scheduling in a significant way; an update results in the entire subtree being re-rendered immediately. Overhauling React's core algorithm to take advantage of scheduling is the driving idea behind Fiber.

---

Now we're ready to dive into Fiber's implementation. The next section is more technical than what we've discussed so far. Please make sure you're comfortable with the previous material before moving on.

###### What is a fiber?


We're about to discuss the heart of React Fiber's architecture. Fibers are a much lower-level abstraction than application developers typically think about. If you find yourself frustrated in your attempts to understand it, don't feel discouraged. Keep trying and it will eventually make sense. (When you do finally get it, please suggest how to improve this section.)

Here we go!

---

We've established that a primary goal of Fiber is to enable React to take advantage of scheduling. Specifically, we need to be able to

- pause work and come back to it later.
- assign priority to different types of work.
- reuse previously completed work.
- abort work if it's no longer needed.

In order to do any of this, we first need a way to break work down into units. In one sense, that's what a fiber is. A fiber represents a **unit of work**.

To go further, let's go back to the conception of [React components as functions of data](https://github.com/reactjs/react-basic#transformation), commonly expressed as

```
v = f(d)
```

It follows that rendering a React app is akin to calling a function whose body contains calls to other functions, and so on. This analogy is useful when thinking about fibers.

The way computers typically track a program's execution is using the [call stack](https://en.wikipedia.org/wiki/Call_stack). When a function is executed, a new **stack frame** is added to the stack. That stack frame represents the work that is performed by that function.

When dealing with UIs, the problem is that if too much work is executed all at once, it can cause animations to drop frames and look choppy. What's more, some of that work may be unnecessary if it's superseded by a more recent update. This is where the comparison between UI components and function breaks down, because components have more specific concerns than functions in general.

Newer browsers (and React Native) implement APIs that help address this exact problem: `requestIdleCallback` schedules a low priority function to be called during an idle period, and `requestAnimationFrame` schedules a high priority function to be called on the next animation frame. The problem is that, in order to use those APIs, you need a way to break rendering work into incremental units. If you rely only on the call stack, it will keep doing work until the stack is empty.

Wouldn't it be great if we could customize the behavior of the call stack to optimize for rendering UIs? Wouldn't it be great if we could interrupt the call stack at will and manipulate stack frames manually?

That's the purpose of React Fiber. Fiber is reimplementation of the stack, specialized for React components. You can think of a single fiber as a **virtual stack frame**.

The advantage of reimplementing the stack is that you can [keep stack frames in memory](https://www.facebook.com/groups/2003630259862046/permalink/2054053404819731/) and execute them however (and _whenever_) you want. This is crucial for accomplishing the goals we have for scheduling.

Aside from scheduling, manually dealing with stack frames unlocks the potential for features such as concurrency and error boundaries. We will cover these topics in future sections.

In the next section, we'll look more at the structure of a fiber.

###### Structure of a fiber


_Note: as we get more specific about implementation details, the likelihood that something may change increases. Please file a PR if you notice any mistakes or outdated information._

In concrete terms, a fiber is a JavaScript object that contains information about a component, its input, and its output.

A fiber corresponds to a stack frame, but it also corresponds to an instance of a component.

Here are some of the important fields that belong to a fiber. (This list is not exhaustive.)

###### `type` and `key`

The type and key of a fiber serve the same purpose as they do for React elements. (In fact, when a fiber is created from an element, these two fields are copied over directly.)

The type of a fiber describes the component that it corresponds to. For composite components, the type is the function or class component itself. For host components (`div`, `span`, etc.), the type is a string.

Conceptually, the type is the function (as in `v = f(d)`) whose execution is being tracked by the stack frame.

Along with the type, the key is used during reconciliation to determine whether the fiber can be reused.

###### `child` and `sibling`

These fields point to other fibers, describing the recursive tree structure of a fiber.

The child fiber corresponds to the value returned by a component's `render` method. So in the following example

```js
function Parent() {
  return <Child />
}
```

The child fiber of `Parent` corresponds to `Child`.

The sibling field accounts for the case where `render` returns multiple children (a new feature in Fiber!):

```js
function Parent() {
  return [<Child1 />, <Child2 />]
}
```

The child fibers form a singly-linked list whose head is the first child. So in this example, the child of `Parent` is `Child1` and the sibling of `Child1` is `Child2`.

Going back to our function analogy, you can think of a child fiber as a [tail-called function](https://en.wikipedia.org/wiki/Tail_call).

###### `return`

The return fiber is the fiber to which the program should return after processing the current one. It is conceptually the same as the return address of a stack frame. It can also be thought of as the parent fiber.

If a fiber has multiple child fibers, each child fiber's return fiber is the parent. So in our example in the previous section, the return fiber of `Child1` and `Child2` is `Parent`.

###### `pendingProps` and `memoizedProps`

Conceptually, props are the arguments of a function. A fiber's `pendingProps` are set at the beginning of its execution, and `memoizedProps` are set at the end.

When the incoming `pendingProps` are equal to `memoizedProps`, it signals that the fiber's previous output can be reused, preventing unnecessary work.

###### `pendingWorkPriority`

A number indicating the priority of the work represented by the fiber. The [ReactPriorityLevel](https://github.com/facebook/react/blob/master/src/renderers/shared/fiber/ReactPriorityLevel.js) module lists the different priority levels and what they represent.

With the exception of `NoWork`, which is 0, a larger number indicates a lower priority. For example, you could use the following function to check if a fiber's priority is at least as high as the given level:

```js
function matchesPriority(fiber, priority) {
  return fiber.pendingWorkPriority !== 0 &&
         fiber.pendingWorkPriority <= priority
}
```

_This function is for illustration only; it's not actually part of the React Fiber codebase._

The scheduler uses the priority field to search for the next unit of work to perform. This algorithm will be discussed in a future section.

###### `alternate`


flush

To flush a fiber is to render its output onto the screen.

work-in-progress

A fiber that has not yet completed; conceptually, a stack frame which has not yet returned.

At any time, a component instance has at most two fibers that correspond to it: the current, flushed fiber, and the work-in-progress fiber.

The alternate of the current fiber is the work-in-progress, and the alternate of the work-in-progress is the current fiber.

A fiber's alternate is created lazily using a function called `cloneFiber`. Rather than always creating a new object, `cloneFiber` will attempt to reuse the fiber's alternate if it exists, minimizing allocations.

You should think of the `alternate` field as an implementation detail, but it pops up often enough in the codebase that it's valuable to discuss it here.

###### `output`

host component

The leaf nodes of a React application. They are specific to the rendering environment (e.g., in a browser app, they are `div`, `span`, etc.). In JSX, they are denoted using lowercase tag names.

Conceptually, the output of a fiber is the return value of a function.

Every fiber eventually has output, but output is created only at the leaf nodes by **host components**. The output is then transferred up the tree.

The output is what is eventually given to the renderer so that it can flush the changes to the rendering environment. It's the renderer's responsibility to define how the output is created and updated.

###### Future sections

That's all there is for now, but this document is nowhere near complete. Future sections will describe the algorithms used throughout the lifecycle of an update. Topics to cover include:

- how the scheduler finds the next unit of work to perform.
- how priority is tracked and propagated through the fiber tree.
- how the scheduler knows when to pause and resume work.
- how work is flushed and marked as complete.
- how side-effects (such as lifecycle methods) work.
- what a coroutine is and how it can be used to implement features like context and layout.

###### Related Videos

- [What's Next for React (ReactNext 2016)](https://youtu.be/aV1271hd9ew)
  

