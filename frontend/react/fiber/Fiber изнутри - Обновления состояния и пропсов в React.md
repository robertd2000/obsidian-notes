https://habr.com/ru/articles/663792/

## Введение

В своей предыдущей статье [Fiber изнутри: погружение в новый алгоритм согласования React](https://habr.com/ru/post/662549/) я заложил фундамент, необходимый для понимания технических деталей процесса обновления, который я опишу в этой статье.

Там я описал основные структуры данных и концепции, которые я буду использовать в текущей статье, в частности fiber-узлы, текущее и work in progress деревья, побочные эффекты и список эффектов. Я также представил высокоуровневый обзор основного алгоритма и объяснил разницу между фазами `render` и `commit`. Если вы еще не читали ту статью, я рекомендую вам начать с нее.

Я также познакомил вас с примером приложения с кнопкой, которая просто увеличивает число, отображаемое на экране:

![](https://habrastorage.org/getpro/habr/upload_files/c0d/07d/415/c0d07d415e1585e818a118820d4e2f33.gif)

Вы можете поиграть с этим примером [здесь](https://stackblitz.com/edit/react-jwqn64). Он реализован как простой компонент, который возвращает два дочерних элемента `button` и `span` из метода `render`. Когда вы нажимаете на кнопку, состояние компонента обновляется внутри обработчика. Это приводит к обновлению текста элемента `span`:

```js
class ClickCounter extends React.Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState((state) => {
            return { count: state.count + 1 };
        });
    }

    componentDidUpdate() {}

    render() {
        return [
            <button key="1" onClick={this.handleClick}>Update counter</button>,
            <span key="2">{this.state.count}</span>
        ];
    }
}
```

Здесь я также добавил метод жизненного цикла `componentDidUpdate` к компоненту. Это необходимо для демонстрации того, как React добавляет [effects](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e) для вызова этого метода на этапе `commit`.

В этой статье я хочу показать вам, как React обрабатывает обновления состояния и строит список эффектов. Мы рассмотрим, что происходит в высокоуровневых функциях в фазах `render` и `commit`.

В частности, мы увидим, как в [completeWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberCompleteWork.js#L532) React:

- обновляет свойство `count` в `state` компонента `ClickCounter`
    
- вызывает метод `render` для получения списка дочерних элементов и выполняет сравнение
    
- обновляет props элемента `span`.
    

И как [commitRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L523) React:

- обновляет свойство `textContent` элемента `span`
    
- вызывает метод жизненного цикла `componentDidUpdate`.
    

Но перед этим давайте быстро посмотрим, как планируется работа, когда мы вызываем `setState` в обработчике клика.

**Обратите внимание, что вам не нужно знать ничего из этого, чтобы использовать React. Эта статья о том, как React работает внутри.**

## Планирование обновлений

Когда мы нажимаем на кнопку, срабатывает событие `click`, и React выполняет обратный вызов, который мы передаем в пропсе кнопки. В нашем приложении он просто увеличивает счетчик и обновляет состояние:

```js
class ClickCounter extends React.Component {
    ...
    handleClick() {
        this.setState((state) => {
            return { count: state.count + 1 };
        });
    }
}
```

Каждый React компонент имеет связанный с ним `updater`, который действует как мост между компонентами и ядром React. Это позволяет реализовать `setState` по-разному в ReactDOM, React Native, рендеринге на стороне сервера и утилитах тестирования.

В этой статье мы рассмотрим реализацию объекта updater в ReactDOM, который использует Fiber reconciler. Для компонента `ClickCounter` это [classComponentUpdater](https://github.com/facebook/react/blob/6938dcaacbffb901df27782b7821836961a5b68d/packages/react-reconciler/src/ReactFiberClassComponent.js#L186). Он отвечает за получение экземпляра Fiber, постановку обновлений в очередь и планирование работы.

Когда обновления ставятся в очередь, они, по сути, просто добавляются в очередь обновлений для обработки на Fiber узле. В нашем случае [Fiber узел](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e), соответствующий компоненту `ClickCounter`, будет иметь следующую структуру:

```js
{
    stateNode: new ClickCounter,
    type: ClickCounter,
    updateQueue: {
        baseState: { count: 0 },
        firstUpdate: {
            next: {
                payload: (state) => { return { count: state.count + 1 } }
            }
        },
        ...
    },
    ...
}
```

Как вы видите, функция в `updateQueue.firstUpdate.next.payload` - это обратный вызов, который мы передали в `setState` в компоненте `ClickCounter`. Он представляет первое обновление, которое должно быть обработано на этапе `render`.

## Обработка обновлений fiber-узла ClickCounter

[Глава о цикле работы](https://indepth.dev/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react/) в моей предыдущей статье объясняет роль глобальной переменной `nextUnitOfWork`. В частности, там говорится, что эта переменная хранит ссылку на fiber-узел из `workInProgress` дерева, которому предстоит выполнить определенную работу. Когда React обходит fiber-дерево, он использует эту переменную, чтобы узнать, есть ли еще какой-нибудь fiber-узел с незавершенной работой.

Начнем с предположения, что метод `setState` был вызван. React добавляет обратный вызов из `setState` в `updateQueue` на fiber-узле `ClickCounter` и планирует работу. React вступает в фазу `render`. Он начинает обход с самого верхнего fiber-узла `HostRoot`, используя функцию [renderRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1132). Однако он обходит (пропускает) уже обработанные узлы Fiber, пока не найдет узел с незавершенной работой. На данный момент есть только один fiber-узел, над которым еще нужно поработать. Это fiber-узел `ClickCounter`.

Вся работа, выполненная на клонированной копии этого fiber-узла, сохраняется в поле `alternate`. Если альтернативный узел еще не создан, React создает копию в функции [createWorkInProgress](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L326) перед обработкой обновлений. Предположим, что переменная `nextUnitOfWork` содержит ссылку на альтернативный fiber-узел `ClickCounter`.

## beginWork

Во-первых, наш fiber попадает в функцию [beginWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489).

> Поскольку эта функция выполняется для каждого fiber-узла в дереве, это хорошее место для установки точки останова, если вы хотите отладить `render` фазу. Я часто так делаю и проверяю тип fiber-узла, чтобы определить нужный мне узел.

Функция `beginWork` - это, по сути, большой оператор `switch`, который определяет тип работы, которую необходимо выполнить для fiber-узла с помощью тега (свойство tag), а затем выполняет соответствующую функцию для выполнения этой работы. В случае `CountClicks` это классовый компонент, поэтому будет взята эта ветвь:

```js
function beginWork(current$$1, workInProgress, ...) {
    ...
    switch (workInProgress.tag) {
        ...
        case FunctionalComponent: {...}
        case ClassComponent:
        {
            ...
            return updateClassComponent(current$$1, workInProgress, ...);
        }
        case HostComponent: {...}
        case ...:
    }
}
```

и мы попадаем в функцию [updateClassComponent](https://github.com/facebook/react/blob/1034e26fe5e42ba07492a736da7bdf5bf2108bc6/packages/react-reconciler/src/ReactFiberBeginWork.js#L428). В зависимости от того, является ли это первым рендерингом компонента, возобновлением работы или React обновлением, React либо создает экземпляр и монтирует компонент, либо просто обновляет его:

```js
function updateClassComponent(current, workInProgress, Component, ...) {
    ...
    const instance = workInProgress.stateNode;
    let shouldUpdate;
    if (instance === null) {
        ...
        // При первом проходе нам понадобится сконструировать экземпляр
        constructClassInstance(workInProgress, Component, ...);
        mountClassInstance(workInProgress, Component, ...);
        shouldUpdate = true;
    } else if (current === null) {
        // При возобновлении у нас уже будет экземпляр, который мы можем использовать повторно
        shouldUpdate = resumeMountClassInstance(workInProgress, Component, ...);
    } else {
        shouldUpdate = updateClassInstance(current, workInProgress, ...);
    }
    return finishClassComponent(current, workInProgress, Component, shouldUpdate, ...);
}
```

## Обработка обновлений fiber-узла ClickCounter

У нас уже есть экземпляр компонента `ClickCounter`, поэтому мы попадаем в [updateClassInstance](https://github.com/facebook/react/blob/6938dcaacbffb901df27782b7821836961a5b68d/packages/react-reconciler/src/ReactFiberClassComponent.js#L976). Именно здесь React выполняет большую часть работы для классовых компонентов. Вот наиболее важные операции, выполняемые в функции, в порядке их выполнения:

- вызов хука `UNSAFE_componentWillReceiveProps()` (устарел)
    
- обработка обновлений в `updateQueue` и генерация нового состояния
    
- вызов `getDerivedStateFromProps` с этим новым состоянием и получение результата
    
- вызов `shouldComponentUpdate`, чтобы убедиться, что компонент хочет обновиться; если `false`, пропустить весь процесс рендеринга, включая вызов `render` для этого компонента и его дочерних компонентов; в противном случае продолжить обновление
    
- вызов `UNSAFE_componentWillUpdate` (устарел)
    
- добавление эффекта для запуска хука жизненного цикла `componentDidUpdate`
    

> Хотя эффект вызова `componentDidUpdate` добавлен в `render` фазе, метод будет выполнен в следующей `commit` фазе.

- обновление `state` и `props` на экземпляре компонента
    

`state` и `props` должны быть обновлены в экземпляре компонента до вызова метода `render`, поскольку вывод метода `render` обычно зависит от `state` и `props`. Если этого не сделать, то метод будет возвращать один и тот же результат каждый раз.

Вот упрощенная версия функции:

```js
function updateClassInstance(current, workInProgress, ctor, newProps, ...) {
    const instance = workInProgress.stateNode;
    const oldProps = workInProgress.memoizedProps;
    instance.props = oldProps;
    if (oldProps !== newProps) {
        callComponentWillReceiveProps(workInProgress, instance, newProps, ...);
    }
    let updateQueue = workInProgress.updateQueue;
    if (updateQueue !== null) {
        processUpdateQueue(workInProgress, updateQueue, ...);
        newState = workInProgress.memoizedState;
    }
    applyDerivedStateFromProps(workInProgress, ...);
    newState = workInProgress.memoizedState;
    const shouldUpdate = checkShouldComponentUpdate(workInProgress, ctor, ...);
    if (shouldUpdate) {
        instance.componentWillUpdate(newProps, newState, nextContext);
        workInProgress.effectTag |= Update;
        workInProgress.effectTag |= Snapshot;
    }
    instance.props = newProps;
    instance.state = newState;
    return shouldUpdate;
}
```

Я удалил некоторый вспомогательный код в приведенном выше фрагменте. Например, прежде чем вызывать методы жизненного цикла или добавлять эффекты для их запуска, React проверяет, реализует ли компонент метод с помощью оператора `typeof`. Вот, например, как React проверяет наличие метода `componentDidUpdate` перед добавлением эффекта:

```js
if (typeof instance.componentDidUpdate === 'function') {
    workInProgress.effectTag |= Update;
}
```

Итак, теперь мы знаем, какие операции выполняются для fiber-узла компонента `ClickCounter` на render фазе. Давайте теперь посмотрим, как эти операции изменяют значения на fiber-узлах. Когда React начинает работу, fiber-узел компонента `ClickCounter` выглядит следующим образом:

```js
{
    effectTag: 0,
    elementType: class ClickCounter,
    firstEffect: null,
    memoizedState: { count: 0 },
    type: class ClickCounter,
    stateNode: {
        state: { count: 0 }
    },
    updateQueue: {
        baseState: { count: 0 },
        firstUpdate: {
            next: {
                payload: (state, props) => {…}
            }
        },
        ...
    }
}
```

После завершения работы мы получаем fiber-узел, который выглядит следующим образом:

```js
{
    effectTag: 4,
    elementType: class ClickCounter,
    firstEffect: null,
    memoizedState: { count: 1 },
    type: class ClickCounter,
    stateNode: {
        state: { count: 1 }
    },
    updateQueue: {
        baseState: { count: 1 },
        firstUpdate: null,
        ...
    }
}
```

_Обратите внимание на различия в значениях свойств_.

После применения обновления значение свойства `count` изменяется на `1` в `memoizedState` и `baseState` в `updateQueue`. React также обновил состояние в экземпляре компонента `ClickCounter`.

На данный момент у нас больше нет обновлений в очереди, поэтому `firstUpdate` равно `null`. И что важно, у нас есть изменения в свойстве `effectTag`. Оно больше не `0`, его значение `4`. В двоичном формате это `100`, что означает, что установлен третий бит, который как раз и является битом для `Update` [side-effect tag](https://github.com/facebook/react/blob/b87aabdfe1b7461e7331abb3601d9e6bb27544bc/packages/shared/ReactSideEffectTags.js):

```js
export const Update = 0b00000000100;
```

Итак, в заключение, при работе над родительским fiber-узлом `ClickCounter`, React вызывает методы жизненного цикла премутации, обновляет состояние и определяет соответствующие побочные эффекты.

## Согласование дочерних элементов fiber-узла ClickCounter

После этого React приступает к [finishClassComponent](https://github.com/facebook/react/blob/340bfd9393e8173adca5380e6587e1ea1a23cefa/packages/react-reconciler/src/ReactFiberBeginWork.js#L355). Здесь React вызывает метод `render` на экземпляре компонента и применяет свой алгоритм диффинга (сравнения) к дочерним элементам, возвращаемым компонентом. Высокоуровневый обзор описан [в документации](https://reactjs.org/docs/reconciliation.html#the-diffing-algorithm). Вот соответствующая часть:

> При сравнении двух элементов React DOM одного типа, React просматривает атрибуты обоих, сохраняет один и тот же базовый узел DOM и обновляет только измененные атрибуты.

Однако если копнуть глубже, то можно узнать, что на самом деле сравниваются fiber-узлы с React-элементами. Но я не буду сейчас вдаваться в подробности, поскольку процесс довольно сложный. Я напишу отдельную статью, которая будет посвящена именно процессу согласования дочерних элементов.

> Если вам не терпится узнать подробности самостоятельно, ознакомьтесь с функцией reconcileChildrenArray, поскольку в нашем приложении метод `render` возвращает массив React элементов.

На этом этапе важно понять две вещи. **Во-первых**, когда React проходит через процесс согласования дочерних элементов, **он создает или обновляет узлы Fiber для дочерних элементов React**, возвращенных из метода `render`. Функция `finishClassComponent` возвращает ссылку на первого ребенка текущего узла Fiber. Она будет присвоена `nextUnitOfWork` и обработана позже в цикле работ. **Во-вторых**, React **обновляет пропсы дочерних компонентов** в рамках работы, выполняемой для родителя. Для этого он использует данные из элементов React, возвращаемых из метода `render`.

Например, вот как выглядит fiber-узел, соответствующий элементу `span`, до того, как React выверит дочерние элементы для fiber-узла `ClickCounter`:

```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    key: "2",
    memoizedProps: { children: 0 },
    pendingProps: { children: 0 },
    ...
}
```

Как вы можете видеть, свойство `children` в обоих `memoizedProps` и `pendingProps` равно `0`. Вот структура элемента React, возвращаемого из `render` для элемента `span`:

```js
{
    $$typeof: Symbol(react.element),
    key: "2",
    props: { children: 1 },
    ref: null,
    type: "span"
}
```

Как вы можете видеть, **есть разница между пропсами** в fiber-узле и возвращаемым React-элементом. Внутри функции [**createWorkInProgress**](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L326)**`**, которая используется для создания альтернативных fiber-узлов, **React скопирует обновленные свойства из элемента React в fiber-узел**.

Таким образом, после того как React завершит сверку дочерних элементов для компонента `ClickCounter`, fiber-узел `span` получит обновленные `pendingProps`. Они будут соответствовать значению в элементе `span` React:

```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    key: "2",
    memoizedProps: { children: 0 },
    pendingProps: { children: 1 },
    ...
}
```

Позже, когда React будет выполнять работу для fiber-узла `span`, он скопирует их в `memoizedProps` и добавит эффекты для обновления DOM.

Вот и вся работа, которую React выполняет для fiber-узла `ClickCounter` на этапе render. Поскольку кнопка является первым дочерним компонентом `ClickCounter`, она будет назначена переменной `nextUnitOfWork`. С ней ничего нельзя сделать, поэтому React перейдет к ее сиблингу, которым является fiber-узел `span`. Согласно алгоритму [описанному здесь](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e), это происходит в функции `completeUnitOfWork`.

## Обработка обновлений для fiber-узла span

Итак, переменная `nextUnitOfWork` теперь указывает на альтернативу fiber-узла `span`, и React начинает работать над ним. Аналогично шагам, выполненным для `ClickCounter`, мы начинаем с функции [beginWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489).

Поскольку наш узел `span` имеет тип `HostComponent`, на этот раз в операторе switch React берет эту ветвь:

```js
function beginWork(current$$1, workInProgress, ...) {
    ...
    switch (workInProgress.tag) {
        case FunctionalComponent: {...}
        case ClassComponent: {...}
        case HostComponent:
            return updateHostComponent(current, workInProgress, ...);
        case ...:
    }
}
```

и попадает в функцию [updateHostComponent](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L686). Вы можете увидеть параллель с функцией `updateClassComponent`, вызываемой для компонентов класса. Для функционального компонента это будет `updateFunctionComponent` и так далее. Все эти функции вы можете найти в файле [ReactFiberBeginWork.js](https://github.com/facebook/react/blob/1034e26fe5e42ba07492a736da7bdf5bf2108bc6/packages/react-reconciler/src/ReactFiberBeginWork.js).

## Согласование дочерних элементов для fiber-узла span

В нашем случае для fiber-узла `span` в `updateHostComponent` не происходит ничего важного.

## Завершение работы для fiber-узла span

После завершения `beginWork` узел попадает в функцию `completeWork`. Но перед этим React необходимо обновить `memoizedProps` на fiber-узле span. Вы можете помнить, что при согласовании дочерних элементов для компонента `ClickCounter`, React обновил `pendingProps` у fiber-узла `span`:

```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    key: "2",
    memoizedProps: { children: 0 },
    pendingProps: { children: 1 },
    ...
}
```

Поэтому, как только `beginWork` завершается для fiber-узла `span`, React обновляет `pendingProps` для соответствия `memoizedProps`:

```js
function performUnitOfWork(workInProgress) {
    ...
    next = beginWork(current$$1, workInProgress, nextRenderExpirationTime);
    workInProgress.memoizedProps = workInProgress.pendingProps;
    ...
}
```

Затем он вызывает функцию `completeWork`, которая по сути является большим оператором `switch`, подобным тому, который мы видели в `beginWork`:

```js
function completeWork(current, workInProgress, ...) {
    ...
    switch (workInProgress.tag) {
        case FunctionComponent: {...}
        case ClassComponent: {...}
        case HostComponent: {
            ...
            updateHostComponent(current, workInProgress, ...);
        }
        case ...:
    }
}
```

Поскольку наш fiber-узел `span` является `HostComponent`, он запускает функцию [updateHostComponent](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L686). В этой функции React в основном делает следующее:

- подготавливает обновления DOM
    
- добавляет их в `updateQueue` fiber-узла `span`
    
- добавляет эффект для обновления DOM
    

До выполнения этих операций fiber-узел `span` выглядит следующим образом:

```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    effectTag: 0,
    updateQueue: null,
    ...
}
```

и когда работа завершена, это выглядит следующим образом:

```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    effectTag: 4,
    updateQueue: ["children", "1"],
    ...
}
```

Обратите внимание на разницу в полях `effectTag` и `updateQueue`. Теперь это не `0`, а `4`. В двоичном формате это `100`, что означает, что установлен третий бит, который как раз и является битом для тега побочного эффекта `Update`. Это единственная работа, которую React должен выполнить для этого узла во время следующей фазы commit. В поле `updateQueue` хранится полезная нагрузка, которая будет использована для обновления.

Как только React обработает `ClickCounter` и его дочерние узлы, он завершает фазу `render`. Теперь он может назначить завершенное альтернативное дерево свойству `finishedWork` у `FiberRoot`. Это новое дерево, которое должно быть выведено на экран. Оно может быть обработано сразу после фазы `render` или получено позже, когда браузер предоставит React время.

## Effects list (список эффектов)

В нашем случае, поскольку узел `span` и компонент `ClickCounter` имеют побочные эффекты, React добавит ссылку на fiber-узел `span` в свойство `firstEffect` компонента `HostFiber`.

React строит список эффектов в функции [compliteUnitOfWork](https://github.com/facebook/react/blob/d5e1bf07d086e4fc1998653331adecddcd0f5274/packages/react-reconciler/src/ReactFiberScheduler.js#L999). Вот как выглядит дерево Fiber с эффектами для обновления текста узла `span` и вызовами хуков на `ClickCounter`:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/99b/e49/bfa/99be49bfa225ef9e8783e2d90a55483f.png)

А вот линейный список узлов с эффектами:

![](https://habrastorage.org/r/w1560/getpro/habr/upload_files/03e/872/be8/03e872be817c7689771cafed6ddf97eb.png)

## Commit фаза

Эта фаза начинается с функции [completeRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L2306). Прежде чем приступить к работе, она устанавливает свойство `finishedWork` для `FiberRoot` в `null`:

```js
root.finishedWork = null;
```

В отличие от первой фазы `render`, фаза `commit` всегда синхронна, поэтому она может безопасно обновить `HostRoot`, чтобы указать, что работа по фиксации изменений началась.

На фазе `commit` React обновляет DOM и вызывает метод жизненного цикла постмутации `componentDidUpdate`. Для этого он просматривает список эффектов, составленный во время предыдущей фазы `render`, и применяет их.

У нас есть следующие эффекты, определенные в фазе `render` для наших узлов `span` и `ClickCounter`:

```js
{ type: ClickCounter, effectTag: 5 }
{ type: 'span', effectTag: 4 }
```

Значение тега эффекта для `ClickCounter` равно `5` **** или `101` в двоичном виде и определяет работу `Update` , которая в основном транслируется в метод жизненного цикла `componentDidUpdate` в случае классовых компонентов. Наименьший значащий бит также устанавливается, чтобы сигнализировать, что вся работа была завершена для этого fiber-узла в фазе `render`.

Значение тега эффекта для `span` равно `4` или `100` в двоичном формате и определяет работу `update` для обновления DOM компонента узла. В случае элемента `span`, React должен будет обновить `textContent` у элемента.

## Применение эффектов

Давайте посмотрим, как React применяет эти эффекты. Функция [commitRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L523), которая используется для применения эффектов, состоит из 3 подфункций:

```js
function commitRoot(root, finishedWork) {
    commitBeforeMutationLifecycles();
    commitAllHostEffects();
    root.current = finishedWork;
    commitAllLifeCycles();
}
```

Каждая из этих подфункций реализует цикл, который итерирует список эффектов и проверяет их типы. Когда он находит эффект, относящийся к цели функции, он применяет его. В нашем случае она вызовет метод жизненного цикла `componentDidUpdate` для компонента `ClickCounter` и обновит текст элемента `span`.

Первая функция [commitBeforeMutationLifeCycles](https://github.com/facebook/react/blob/fefa1269e2a67fa5ef0992d5cc1d6114b7948b7e/packages/react-reconciler/src/ReactFiberCommitWork.js#L183) ищет эффект [Snapshot](https://github.com/facebook/react/blob/b87aabdfe1b7461e7331abb3601d9e6bb27544bc/packages/shared/ReactSideEffectTags.js#L25) и вызывает метод `getSnapshotBeforeUpdate`. Но, поскольку мы не реализовали метод на компоненте `ClickCounter`, React не добавил эффект на этапе `render`. Поэтому в нашем случае эта функция ничего не делает.

## Обновления DOM

Далее React переходит к функции [commitAllHostEffects](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L376). Здесь React изменит текст на элементе `span` с `0` на `1`. Для fiber-узла `ClickCounter` ничего делать не нужно, потому что узлы, соответствующие классовым компонентам, не имеют никаких обновлений DOM.

Суть функции в том, что она выбирает нужный тип эффекта и применяет соответствующие операции. В нашем случае нам нужно обновить текст на элементе `span`, поэтому здесь мы берем ветвь `Update`:

```js
function updateHostEffects() {
    switch (primaryEffectTag) {
        case Placement: {...}
        case PlacementAndUpdate: {...}
        case Update:
        {
            var current = nextEffect.alternate;
            commitWork(current, nextEffect);
            break;
        }
        case Deletion: {...}
    }
}
```

Спустившись в `commitWork`, мы в конечном итоге попадем в функцию [updateDOMProperties](https://github.com/facebook/react/blob/8a8d973d3cc5623676a84f87af66ef9259c3937c/packages/react-dom/src/client/ReactDOMComponent.js#L326). Она принимает полезную нагрузку `updateQueue`, которая была добавлена на этапе `render` к узлу Fiber, и обновляет свойство `textContent` элемента `span`:

```js
function updateDOMProperties(domElement, updatePayload, ...) {
    for (let i = 0; i < updatePayload.length; i += 2) {
        const propKey = updatePayload[i];
        const propValue = updatePayload[i + 1];
        if (propKey === STYLE) { ...}
        else if (propKey === DANGEROUSLY_SET_INNER_HTML) {...}
        else if (propKey === CHILDREN) {
            setTextContent(domElement, propValue);
        } else {...}
    }
}
```

После выполнения обновлений DOM, React присваивает дерево `finishedWork` в `HostRoot`. Это устанавливает альтернативное дерево в качестве текущего:

```js
root.current = finishedWork;
```

## Вызов хуков жизненного цикла после мутации

Последняя оставшаяся функция - [**commitAllLifecycles**](https://github.com/facebook/react/blob/d5e1bf07d086e4fc1998653331adecddcd0f5274/packages/react-reconciler/src/ReactFiberScheduler.js#L479). Здесь React вызывает методы жизненного цикла после мутации. Во время фазы `render` React добавил эффект `Update` к компоненту `ClickCounter`. Это один из эффектов, который ищет функция `commitAllLifecycles` и вызывает метод `componentDidUpdate`:

```js
function commitAllLifeCycles(finishedRoot, ...) {
    while (nextEffect !== null) {
        const effectTag = nextEffect.effectTag;
        if (effectTag & (Update | Callback)) {
            const current = nextEffect.alternate;
            commitLifeCycles(finishedRoot, current, nextEffect, ...);
        }
        if (effectTag & Ref) {
            commitAttachRef(nextEffect);
        }
        nextEffect = nextEffect.nextEffect;
    }
}
```

Функция также обновляет [refs](https://reactjs.org/docs/refs-and-the-dom.html), но поскольку у нас их нет, эта функциональность не будет использоваться. Метод вызывается в функции [commitLifeCycles](https://github.com/facebook/react/blob/e58ecda9a2381735f2c326ee99a1ffa6486321ab/packages/react-reconciler/src/ReactFiberCommitWork.js#L351):

```js
function commitLifeCycles(finishedRoot, current, ...) {
    ...
    switch (finishedWork.tag) {
        case FunctionComponent: {...}
        case ClassComponent: {
            const instance = finishedWork.stateNode;
            if (finishedWork.effectTag & Update) {
                if (current === null) {
                    instance.componentDidMount();
                } else {
                    ...
                    instance.componentDidUpdate(prevProps, prevState, ...);
                }
            }
        }
        case HostComponent: {...}
        case ...:
    }
}
```

Вы также можете видеть, что в этой функции React вызывает метод `componentDidMount` для компонентов, которые были отрендерены в первый раз.