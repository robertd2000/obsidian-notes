
##### **Какие типы данных поддерживает TypeScript?**

- **Ответ:** TypeScript поддерживает:

    - Примитивные типы: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.

    - Сложные типы: `array`, `tuple`, `enum`, `object`, `function`.

    - Специальные типы: `any`, `unknown`, `void`, `never`.

##### interface vs type

- `type` может использоваться для создания псевдонимов для любых типов, включая примитивы, объединения и пересечения.

- `interface` используется только для описания формы объектов и может быть расширен с помощью `extends`.

##### type guards

Понимание механизмов, рассматриваемых в этой главе, научит определять конструкции, которые часто применяются на практике и способны сделать код более понятным и выразительным.

###### Защитники Типа - общее

Помимо того, что _TypeScript_ имеет достаточно мощную систему выявления ошибок на этапе компиляции, разработчики языка, не останавливаясь на достигнутом, безостановочно работают над сведением их к нулю. Существенным шагом к достижению цели было добавление компилятору возможности, активируемой при помощи флага `--strictNullChecks`, запрещающей неявные операции, в которых участвует значение `null` и `undefined`. Простыми словами, компилятор научили во время анализа кода выявлять ошибки, способные возникнуть при выполнении операций, в которых фигурируют значения `null` или `undefined`.

Простейшим примером является операция получения элемента из dom-дерева при помощи метода `querySelector()`, который в обычном _нерекомендуемом_ режиме (с неактивной опцией `--strictNullChecks`) возвращает значение, совместимое с типом `Element`.

```ts
const stage: Element = document.querySelector('#stage');
```

Но в строгом _рекомендуемом_ режиме (с активной опцией `--strictNullChecks`) метод `querySelector()` возвращает объединенный тип `Element | null`, поскольку искомое значение может попросту не существовать.

```ts
const stage: Element | null = document.querySelector(
  '#stage'
);
```

Не будет лишним напомнить, что на самом деле метод `querySelector` возвращает тип `Element | null` независимо от режима. Дело в том, что в обычном режиме тип `null` совместим с любыми типами. То есть, в случае отсутствия элемента в dom-дереве операция присваивания значения `null` переменной с типом `Element` не приведет к возникновению ошибки.

```ts
// lib.es6.d.ts
interface NodeSelector {
  querySelector(selectors: string): Element | null;
}
```

Возвращаясь к примеру с получением элемента из dom-дерева стоит сказать, что в строке кода, в которой происходит подписка элемента на событие, на этапе компиляции все равно возникнет ошибка, даже в случае, если элемент существует. Дело в том, что компилятор _TypeScript_ не позволит вызвать метод `addEventListener`, поскольку для него объект, на который ссылается переменная, принадлежит к типу `Element` ровно настолько же, насколько он принадлежит к типу `null`.

```ts
const stage: Element | null = document.querySelector(
  '#stage'
);
stage.addEventListener('click', stage_clickHandler); // тип переменной stage Element или Null?

function stage_clickHandler(event: MouseEvent): void {}
```

Именно из-за этой особенности или другими словами, неоднозначности, которую вызывает тип `Union`, в _TypeScript_, появился механизм, называемый _защитниками типа_ (`Type Guards`).

_Защитники типа_ — это правила, которые помогают выводу типов определить суженный диапазон типов для значения, принадлежащего к типу `Union`. Другими словами, разработчику предоставлен механизм, позволяющий с помощью выражений составить логические условия, проанализировав которые, вывод типов сможет сузить диапазон типов до указанного и выполнить над ним требуемые операции.

Понятно, что ничего не понятно. Поэтому, прежде чем продолжить разбирать определение по шагам, нужно рассмотреть простой пример, способный зафиксировать картинку в сознании.

Представим два класса, `Bird` и `Fish`, в обоих из которых реализован метод `voice`. Кроме этого, в классе `Bird` реализован метод `fly`, а в классе `Fish` — метод `swim`. Далее представим функцию с единственным параметром, принадлежащим к объединению типов `Bird` и `Fish`. В теле этой функции без труда получится выполнить операцию вызова метода `voice` у её параметра, так как этот метод объявлен и в типе `Bird`, и в типе `Fish`. Но при попытке вызвать метод `fly` или `swim` возникает ошибка, так как эти методы не являются общими для обоих типов. Компилятор попросту находится в подвешенном состоянии и не способен самостоятельно определиться.

```ts
class Bird {
  public fly(): void {}
  public voice(): void {}
}

class Fish {
  public swim(): void {}
  public voice(): void {}
}

function move(animal: Bird | Fish): void {
  animal.voice(); // Ok

  animal.fly(); // Error
  animal.swim(); // Error
}
```

Для того, чтобы облегчить работу компилятору, _TypeScript_ предлагает процесс сужения множества типов, составляющих тип `Union`, до заданного диапазона, а затем закрепляет его за конкретной областью видимости в коде. Но, прежде чем диапазон типов будет вычислен и ассоциирован с областью, разработчику необходимо составить условия, включающие в себя признаки, недвусмысленно указывающие на принадлежность к нужным типам.

Из-за того, что анализ происходит на основе логических выражений, область, за которой закрепляется суженый диапазон типов, ограничивается областью, выполняемой при истинности условия.

Стоит заметить, что от признаков, участвующих в условии, зависит место, в котором может находиться выражение, а от типов, составляющих множество типа `Union`, зависит способ составления логического условия.

###### Сужение диапазона множества типов на основе типа данных

При необходимости составления условия, в основе которого лежат допустимые с точки зрения _JavaScript_ типы, прибегают к помощи уже знакомых операторов `typeof` и `instanceof`.

К помощи оператора `typeof` прибегают тогда, когда хотят установить принадлежность к типам `number`, `string`, `boolean`, `object`, `function`, `symbol` или `undefined`. Если значение принадлежит к производному от объекта типу, то установить его принадлежность к типу, определяемого классом и находящегося в иерархии наследования, можно при помощи оператора `instanceof`.

Как уже было сказано, с помощью операторов `typeof` и `instanceof` составляется условие, по которому компилятор может вычислить, к какому конкретно типу или диапазону будет относиться значение в определяемой условием области.

```ts
// Пример для оператора typeof

type ParamType =
  | number
  | string
  | boolean
  | object
  | Function
  | symbol
  | undefined;

function identifier(param: ParamType): void {
  param; // param: number | string | boolean | object | Function | symbol | undefined

  if (typeof param === 'number') {
    param; // param: number
  } else if (typeof param === 'string') {
    param; // param: string
  } else if (typeof param === 'boolean') {
    param; // param: boolean
  } else if (typeof param === 'object') {
    param; // param: object
  } else if (typeof param === 'function') {
    param; // param: Function
  } else if (typeof param === 'symbol') {
    param; // param: symbol
  } else if (typeof param === 'undefined') {
    param; // param: undefined
  }

  param; // param: number | string | boolean | object | Function | symbol | undefined
}
```

```ts
// Пример для оператора instanceof

class Animal {
  constructor(public type: string) {}
}

class Bird extends Animal {}
class Fish extends Animal {}
class Insect extends Animal {}

function f(param: Animal | Bird | Fish | Insect): void {
  param; // param: Animal | Bird | Fish | Insect

  if (param instanceof Bird) {
    param; // param: Bird
  } else if (param instanceof Fish) {
    param; // param: Fish
  } else if (param instanceof Insect) {
    param; // param: Insect
  }

  param; // param: Animal | Bird | Fish | Insect
}
```

Если значение принадлежит к типу `Union`, а выражение состоит из двух операторов, `if` и `else`, значение, находящееся в операторе `else`, будет принадлежать к диапазону типов, не участвующих в условии `if`.

```ts
// Пример для оператора typeof

function f0(param: number | string | boolean): void {
  param; // param: number | string | boolean

  if (
    typeof param === 'number' ||
    typeof param === 'string'
  ) {
    param; // param: number | string
  } else {
    param; // param: boolean
  }

  param; // param: number | string | boolean
}

function f1(param: number | string | boolean): void {
  param; // param: number | string | boolean

  if (typeof param === 'number') {
    param; // param: number
  } else {
    param; // param: string | boolean
  }

  param; // param: number | string | boolean
}
```

```ts
// Пример для оператора instanceof

class Animal {
  constructor(public type: string) {}
}

class Bird extends Animal {}
class Fish extends Animal {}
class Insect extends Animal {}
class Bug extends Insect {}

function f0(param: Bird | Fish | Insect): void {
  param; // param: Bird | Fish | Insect

  if (param instanceof Bird) {
    param; // param: Bird
  } else if (param instanceof Fish) {
    param; // param: Fish
  } else {
    param; // param: Insect
  }

  param; // param: Bird | Fish | Insect
}

function f1(
  param: Animal | Bird | Fish | Insect | Bug
): void {
  param; // param: Animal | Bird | Fish | Insect | Bug
  if (param instanceof Bird) {
    param; // param: Bird
  } else if (param instanceof Fish) {
    param; // param: Fish
  } else {
    param; // param: Animal | Insect | Bug
  }

  param; // param: Animal | Bird | Fish | Insect | Bug
}
```

Кроме того, условия можно поместить в тернарный оператор. В этом случае область, на которую распространяется сужение диапазона типов, ограничивается областью, содержащей условное выражение.

Представьте функцию, которой в качестве единственного аргумента можно передать как значение, принадлежащее к типу `T`, так и функциональное выражение, возвращающее значение, принадлежащее к типу `T`. Для того чтобы было проще работать со значением параметра, его нужно сохранить в локальную переменную, принадлежащую к типу `T`. Но прежде компилятору нужно помочь конкретизировать тип данных, к которому принадлежит значение.

Условие, как и раньше, можно было бы поместить в конструкцию `if`/`else`, но в таких случаях больше подходит тернарный условный оператор. Создав условие, в котором значение проверяется на принадлежность к типу, отличному от типа `T`, разработчик укажет компилятору, что при выполнении условия тип параметра будет ограничен типом `Function`, тем самым создав возможность вызвать параметр как функцию. Иначе значение, хранимое в параметре, принадлежит к типу `T`.

```ts
// Пример для оператора typeof

function f(param: string | (() => string)): void {
  param; // param: string | (() => string)

  let value: string =
    typeof param !== 'string' ? param() : param;

  param; // param: string | (() => string)
}
```

```ts
// Пример для оператора instanceof

class Animal {
  constructor(public type: string = 'type') {}
}

function identifier(param: Animal | (() => Animal)): void {
  param; // param: Animal | (() => Animal)

  let value: Animal = !(param instanceof Animal)
    ? param()
    : param;

  param; // param: Animal | (() => Animal)
}
```

Так как оператор `switch` логически похож на оператор `if`/`else`, то может показаться, что механизм, рассмотренный в этой главе, будет применим и к нему. Но это не так. Вывод типов не умеет различать условия, составленные при помощи операторов `typeof` и `instanceof` в конструкции `switch`.

###### Сужение диапазона множества типов на основе признаков, присущих типу Tagged Union

Помимо определения принадлежности к единичному типу, диапазон типов, составляющих тип `Union`, можно сузить по признакам, характерным для типа `Tagged Union`.

Условия, составленные на основе идентификаторов варианта, можно использовать во всех условных операторах, включая `switch`.

```ts
// Пример для оператора if/else

enum AnimalTypes {
  Animal = 'animal',
  Bird = 'bird',
  Fish = 'fish',
}

class Animal {
  readonly type: AnimalTypes = AnimalTypes.Animal;
}

class Bird extends Animal {
  readonly type: AnimalTypes.Bird = AnimalTypes.Bird;

  public fly(): void {}
}

class Fish extends Animal {
  readonly type: AnimalTypes.Fish = AnimalTypes.Fish;

  public swim(): void {}
}

function move(param: Bird | Fish): void {
  param; // param: Bird | Fish

  if (param.type === AnimalTypes.Bird) {
    param.fly();
  } else {
    param.swim();
  }

  param; // param: Bird | Fish
}
```

```ts
// Пример для тернарного оператора (?:)

function move(param: Bird | Fish): void {
  param; // param: Bird | Fish

  param.type === AnimalTypes.Bird
    ? param.fly()
    : param.swim();

  param; // param: Bird | Fish
}
```

```ts
// Пример для оператора switch

enum AnimalTypes {
  Animal = 'animal',
  Bird = 'bird',
  Fish = 'fish',
}

class Animal {
  readonly type: AnimalTypes = AnimalTypes.Animal;
}

class Bird extends Animal {
  readonly type: AnimalTypes.Bird = AnimalTypes.Bird;

  public fly(): void {}
}

class Fish extends Animal {
  readonly type: AnimalTypes.Fish = AnimalTypes.Fish;

  public swim(): void {}
}

function move(param: Bird | Fish): void {
  param; // param: Bird | Fish

  switch (param.type) {
    case AnimalTypes.Bird:
      param.fly(); // Ok
      break;

    case AnimalTypes.Fish:
      param.swim(); // Ok
      break;
  }

  param; // param: Bird | Fish
}
```

В случаях, когда множество типа `Union` составляют тип `null` и/или `undefined`, а также только один конкретный тип, выводу типов будет достаточно условия, подтверждающего существование значения, отличного от `null` и/или `undefined`. Это очень распространенный случай при активной опции `--strictNullChecks`. Условие, с помощью которого вывод типов сможет установить принадлежность значения к типам, отличным от `null` и/или `undefined`, может использоваться совместно с любыми условными операторами.

```ts
// Пример с оператором if/else

function f(param: number | null | undefined): void {
  param; // param: number | null | undefined

  if (param !== null && param !== undefined) {
    param; // param: number
  }

  // or

  if (param) {
    param; // Param: number
  }

  param; // param: number | null | undefined
}
```

```ts
// Пример с тернарным оператором (?:), оператором нулевого слияния (??, nullish coalescing) и логическим "или" (||)

function f(param: number | null | undefined): void {
  param; // param: number | null | undefined

  var value: number = param ? param : 0;
  var value: number = param ?? 0;
  var value: number = param || 0;

  param; // param: number | null | undefined
}
```

```ts
// Пример с оператором switch

function identifier(
  param: number | null | undefined
): void {
  param; // param: number | null | undefined

  switch (param) {
    case null:
      param; // param: null
      break;

    case undefined:
      param; // param: undefined
      break;

    default: {
      param; // param: number
    }
  }

  param; // param: number | null | undefined
}
```

Кроме этого, при активной опции `--strictNullChecks`, в случаях со значением, принадлежащим к объектному типу, вывод типов может заменить оператор `Not-Null Not-Undefined`. Для этого нужно составить условие, содержащее проверку обращения к членам, в случае отсутствия которых может возникнуть ошибка.

```ts
// Пример с Not-Null Not-Undefined (с учетом активной опции --strictNullChecks)

class Ability {
  public fly(): void {}
}

class Bird {
  public ability: Ability | null = new Ability();
}

function move(animal: Bird | null | undefined): void {
  animal.ability; // Error, Object is possibly 'null' or 'undefined'
  animal!.ability; // Ok
  animal!.ability.fly(); // Error, Object is possibly 'null' or 'undefined'
  animal!.ability!.fly(); // Ok
}
```

```ts
// Пример с защитником типа (с учетом активной опции --strictNullChecks)

class Ability {
  public fly(): void {}
}

class Bird {
  public ability: Ability | null = new Ability();
}

function move(animal: Bird | null | undefined): void {
  if (animal && animal.ability) {
    animal.ability.fly(); // Ok
  }

  // или с помощью оператора optional chaining
  if (animal?.ability) {
    animal.ability.fly(); // Ok
  }
}
```

###### Сужение диапазона множества типов на основе доступных членов объекта

Сужение диапазона типов также возможно на основе доступных (`public`) членов, присущих типам, составляющим диапазон (`Union`). Сделать это можно с помощью оператора `in`.

```ts
class A {
  public a: number = 10;
}
class B {
  public b: string = 'text';
}
class C extends A {}

function f0(p: A | B) {
  if ('a' in p) {
    return p.a; // p: A
  }

  return p.b; // p: B
}

function f1(p: B | C) {
  if ('a' in p) {
    return p.a; // p: C
  }

  return p.b; // p: B
}
```

###### Сужение диапазона множества типов на основе функции, определенной пользователем

Все перечисленные ранее способы работают только в том случае, если проверка происходит в месте, отведенном под условие. Другими словами, с помощью перечисленных до этого момента способов, условие проверки нельзя вынести в отдельный блок кода (функцию). Это могло бы сильно ударить по семантической составляющей кода, а также нарушить принцип разработки программного обеспечения, который призван бороться с повторением кода (_Don’t repeat yourself, DRY_ (не повторяйся)). Но, к счастью для разработчиков, создатели _TypeScript_ реализовали возможность определять пользовательские защитники типа.

В роли пользовательского защитника может выступать функция, функциональное выражение или метод, которые обязательно должны возвращать значения, принадлежащие к типу `boolean`. Для того, чтобы вывод типов понял, что вызываемая функция не является обычной функцией, у функции вместо типа возвращаемого значения указывают предикат (предикат — это логическое выражение, значение которого может быть либо истинным `true`, либо ложным `false`).

Выражение предиката состоит из трех частей и имеет следующий вид `identifier is Type`.

Первым членом выражения является идентификатор, который обязан совпадать с идентификатором одного из параметров, объявленных в сигнатуре функции. В случае, когда предикат указан методу экземпляра класса, в качестве идентификатора может быть указано ключевое слово `this`.

Стоит отдельно упомянуть, что ключевое слово `this` можно указать только в сигнатуре метода, определенного в классе или описанного в интерфейсе. При попытке указать ключевое слово `this` в предикате функционального выражения, не получится избежать ошибки, если это выражение определяется непосредственно в `prototype`, функции конструкторе, либо методе объекта, созданного с помощью литерала.

```ts
// Пример с функцией конструктором

function Constructor() {}

Constructor.prototype.validator = function (): this is Object {
  // Error
  return true;
};
```

```ts
// Пример с литералом объекта

interface IPredicat {
  validator(): this is Object; // Ok
}

var object: IPredicat = {
  // Ok
  validator(): this is Object {
    // Error
    return this;
  },
};

var object: { validator(): this is Object } = {
  // Error
  validator(): this is Object {
    // Error
    return this;
  },
};
```

Ко второму члену выражения относится ключевое слово `is`, которое служит в качестве утверждения. В качестве третьего члена выражения может выступать любой тип данных.

```ts
// Пример предиката функции (function)

function isT1(p1: T1 | T2 | T3): p1 is T1 {
  return p1 instanceof T1;
}

function identifier(p1: T1 | T2 | T3): void {
  if (isT1(p1)) {
    p1; // p1: T1
  }
}
```

```ts
// Пример предиката функционального выражения (functional expression)

const isT2 = (p1: T1 | T2 | T3): p1 is T2 =>
  p1 instanceof T2;

function identifier(p1: T1 | T2 | T3): void {
  if (isT2(p1)) {
    p1; // p1: T2
  }
}
```

```ts
// Пример предиката метода класса (static method)

class Validator {
  public static isT3(p1: T1 | T2 | T3): p1 is T3 {
    return p1 instanceof T3;
  }
}

function identifier(p1: T1 | T2 | T3): void {
  if (Validator.isT3(p1)) {
    p1; // p1: T3
  }
}
```

Условие, на основании которого разработчик определяет принадлежность одного из параметров к конкретному типу данных, не ограничено никакими конкретными правилами. Исходя из результата выполнения условия `true` или `false`, вывод типов сможет установить принадлежность указанного параметра к указанному типу данных.

```ts
class Animal {}
class Bird extends Animal {
  public fly(): void {}
}

class Fish extends Animal {
  public swim(): void {}
}

class Insect extends Animal {
  public crawl(): void {}
}

class AnimalValidator {
  public static isBird(animal: Animal): animal is Bird {
    return animal instanceof Bird;
  }

  public static isFish(animal: Animal): animal is Fish {
    return (animal as Fish).swim !== undefined;
  }

  public static isInsect(animal: Animal): animal is Insect {
    let isAnimalIsNotUndefinedValid: boolean =
      animal !== undefined;
    let isInsectValid: boolean = animal instanceof Insect;

    return isAnimalIsNotUndefinedValid && isInsectValid;
  }
}

function move(animal: Animal): void {
  if (AnimalValidator.isBird(animal)) {
    animal.fly();
  } else if (AnimalValidator.isFish(animal)) {
    animal.swim();
  } else if (AnimalValidator.isInsect(animal)) {
    animal.crawl();
  }
}
```

Последнее, о чем осталось упомянуть, что в случае, когда по условию значение не подходит ни по одному из признаков, вывод типов установит его принадлежность к типу `never`.

```ts
class Animal {
  constructor(public type: string) {}
}

class Bird extends Animal {}
class Fish extends Animal {}

function move(animal: Bird | Fish): void {
  if (animal instanceof Bird) {
    animal; // animal: Bird
  } else if (animal instanceof Fish) {
    animal; // animal: Fish
  } else {
    animal; // animal: never
  }
}
```
##### generics

Из всего, что стало и ещё станет известным о типизированном мире, тем, кто только начинает свое знакомство с ним, тема, посвященная обобщениям (_generics_), может казаться наиболее сложной. Хотя данная тема, как и все остальные, обладает некоторыми нюансами, каждый из которых будет детально разобран, в реальности рассматриваемые в ней механизмы очень просты и схватываются на лету. Поэтому приготовьтесь, к концу главы место, занимаемое множеством вопросов, касающихся обобщений, займет желание сделать все пользовательские конструкции универсальными.

###### Общие понятия

Представьте огромный и дорогущий, высокотехнологичный типографский печатный станок, выполненный в виде монолита, что в свою очередь делает его пригодным для печати только одного номера газеты. То есть для печати сегодняшних новостей необходим один печатный станок, для завтрашних другой и т.д. Подобный станок сравним с _обычным типом_, признаки которого после объявления остаются неизменными при его реализации. Другими словами, если при существовании типа `A`, описание которого включает поле, принадлежащее к типу `number`, потребуется тип, отличие которого будет состоять лишь в принадлежности поля к другому типу, возникнет необходимость в его объявлении.

```ts
// простые типы сравнимы с монолитами

// этот станок предназначен для печати газет под номером A
interface A {
  field: number;
}

// этот станок предназначен для печати газет под номером B
interface B {
  field: string;
}

// и т.д.
```

К счастью, в нашей реальности нашли решение не только относительно печатных станков, но и типов. Нежелание тратить усилия на постоянное описывание монолитных типов послужило причиной зарождения парадигмы _обобщенного программирования_.

_Обобщенное программирование_ (_Generic Programming_) — это подход, при котором алгоритмы могут одинаково работать с данными, принадлежащими к разным типам данных, без изменения декларации (описания типа).

В основе обобщенного программирования лежит такое ключевое понятие как _обобщение_. _Обобщение_ (_Generics_) - это _параметризированный тип_, позволяющий объявлять _параметры типа_, являющиеся временной заменой _конкретных типов_, _конкретизация_ которых будет выполнена в момент создания экземпляра. Параметры типа, при условии соблюдения некоторых правил, можно использовать в большинстве операций, допускающих работу с обычными типами. Все это вместе дает повод сравнивать обобщенный тип с _правильной версией_ печатного станка, чьи заменяемые валы, предназначенные для отпечатывания информации на проходящей через них бумаге, сопоставимы с _параметрами типа_.

В реальности обобщения позволяют сокращать количество преобразований (приведений) и писать многократно используемый код, при этом повышая его типобезопасность.

Этих примеров должно быть достаточно для образования отчетливого образа обобщений. Но, прежде чем продолжить, стоит уточнить значения таких далеко не всем очевидных терминов, как - _обобщенный тип_, _параметризированный тип_ и _универсальная конструкция_.

Для понимания этих терминов необходимо представить чертеж бумажного домика, в который планируется поселить пойманного на пикнике жука. Когда гипотетический жук мысленно располагается вне границ начерченного жилища, сопоставимого с типом, то оно предстает в виде _обобщенного типа_. Когда жук представляется внутри своей будущей обители, то о ней говорят как о _параметризированном типе_. Если же чертеж материализовался, хотя и в форму, представленную обычной коробкой из-под печенья, то её называют _универсальной конструкцией_.

Другими словами, тип, определяющий параметр, обозначается как обобщенный тип. При обсуждении типов, представляемых параметрами типа, необходимо понимать, что они определены в параметризированном типе. Когда объявление обобщенного типа получило реализацию, то такую конструкцию, будь то класс или функция, называют универсальной (универсальный класс, универсальная функция или метод).

###### Обобщения в TypeScript

В _TypeScript_ обобщения могут быть указаны для типов, определяемых с помощью:

- _псевдонимов_ (`type`)
- _интерфейсов_, объявленных с помощью ключевого слова `interface`
- _классов_ (`class`), в том числе _классовых выражений_ (_class expression_)
- _функций_ (`function`), определенных в виде как деклараций (_Function Declaration_), так и выражений (_Function Expression_)
- _методов_ (_method_)

Обобщения объявляются при помощи пары угловых скобок, в которые через запятую заключены _параметры типа_, называемые также _типо-заполнителями_ или _универсальными параметрами_ `Type<T0, T1>`.

```ts
/**[0][1] [2] */
interface Type<T0, T1> {}

/**
 * [0] объявление обобщенного типа Type,
 * определяющего два параметра типа [1][2]
 */
```

Параметры типа могут быть указаны в качестве типа везде, где требуется аннотация типа, за исключением членов класса (static members). Область видимости параметров типа ограничена областью обобщенного типа. Все вхождения параметров типа будут заменены на конкретные типы, переданные в качестве аргументов типа. Аргументы типа указываются в угловых скобках, в которых через запятую указываются конкретные типы данных `Type<number, string>`.

```ts
/**[0]  [1]      [2] */
let value: Type<number, string>;

/**
 * [0] указание обобщенного типа,
 * которому в качестве аргументов
 * указываются конкретные типы
 * number [1] и string [2]
 */
```

Идентификаторы параметров типа должны начинаться с заглавной буквы и, кроме фантазии разработчика, они также ограничены общими для _TypeScript_ правилами. Если логическую принадлежность параметра типа возможно установить без какого-либо труда, как, например, в случае `Array<T>`, кричащего, что параметр типа `T` представляет тип, к которому могут принадлежать элементы этого массива, то идентификаторы параметров типа принято выбирать из последовательности `T`, `S`, `U`, `V` и т.д. Также частая последовательность `T`, `U`, `V`, `S` и т.д.

С помощью `K` и `V` принято обозначать типы, соответствующие `Key`/`Value`, а при помощи `P` — `Property`. Идентификатором `Z` принято обозначать полиморфный тип `this`.

Кроме того, не исключены случаи, в которых предпочтительнее выглядят полные имена, как, например, `RequestService`, `ResponseService`, к которым ещё можно применить _Венгерскую нотацию_ - `TRequestService`, `TResponseService`.

К примеру, увидев в автодополнении редактора тип `Array<T>`, в голову тут же приходит верный вариант, что массив будет содержать элементы, принадлежащие к указанному типу `T`. Но, увидев `Animal<T, S>`, можно никогда не догадаться, что эти типы данных будут указаны в аннотации типа полей `id` и `arial`. В этом случае было бы гораздо предпочтительней дать говорящие имена `Animal<AnimalID, AnimalArial>` или даже `Animal<TAnimalID, TAnimalArial>`, что позволит внутри тела параметризированного типа `Animal` отличать его параметры типа от конкретных объявлений.

Указывается обобщение сразу после идентификатора типа. Это правило остается неизменным даже в тех случаях, когда идентификатор отсутствует (как в случае с безымянным классовым или функциональным выражением), или же и вовсе не предусмотрен (стрелочная функция).

```ts
type Identifier<T> = {};

interface Identifier<T> {}

class Identifier<T> {
  public identifier<T>(): void {}
}

let identifier = class<T> {};

function identifier<T>(): void {}

let identifier = function <T>(): void {};

let identifier = <T>() => {};
```

Но прежде чем приступить к детальному рассмотрению, нужно уточнить, что правила для функций, функциональных выражений и методов - идентичны. Правила для классов ничем не отличаются от правил для классовых выражений. Исходя из этого, все дальнейшие примеры будут приводиться исключительно на классах и функциях.

В случае, когда обобщение указано псевдониму типа (`type`), область видимости параметров типа ограничена самим выражением.

```ts
type T1<T> = { f1: T };
```

Область видимости параметров типа при объявлении функции и функционального выражения, включая стрелочное, а также методов, ограничивается их сигнатурой и телом. Другими словами, параметр типа можно использовать в качестве типа при объявлении параметров, возвращаемого значения, а также в допустимых выражениях (аннотация типа, приведение типа и т.д.), расположенных в теле.

```ts
function f1<T>(p1: T): T {
  let v1: T;

  return v1;
}
```

При объявлении классов (в том числе и классовых выражений) и интерфейсов, область видимости параметров типа ограничивается областью объявления и телом.

```ts
interface IT1<T> {
  f1: T;
}

class T1<T> {
  public f1: T;
}
```

В случаях, когда класс/интерфейс расширяет другой класс/интерфейс, который объявлен как обобщенный, потомок обязан указать типы для своего предка. Потомок в качестве аргумента типа своему предку может указать не только конкретный тип, но и тип, представляемый собственными параметрами типа.

```ts
interface IT1<T> {}

interface IT3<T> extends IT1<T> {}
interface IT2 extends IT1<string> {}

class T1<T> {}

class T2<T> extends T1<T> implements IT1<T> {}
class T3 extends T1<string> implements IT1<string> {}
```

Если класс/интерфейс объявлен как обобщенный, а внутри него объявлен обобщенный метод, имеющий идентичный параметр типа, то последний в своей области видимости будет перекрывать первый (более конкретно это поведение будет рассмотрено позднее).

```ts
interface IT1<T> {
  m2<T>(p1: T): T;
}

class T1<T> {
  public m1<T>(p1: T): T {
    let v1: T;

    return p1;
  }
}
```

Принадлежность параметра типа к конкретному типу данных устанавливается в момент передачи аргументов типа. При этом конкретные типы данных указываются в паре угловых скобок, а количество конкретных типов должно соответствовать количеству обязательных параметров типа.

```ts
class Animal<T> {
  constructor(readonly id: T) {}
}

var bird: Animal<string> = new Animal('bird'); // Ok
var bird: Animal<string> = new Animal(1); // Error
var fish: Animal<number> = new Animal(1); // Ok
```

Если обобщенный тип указывается в качестве типа данных, то он обязан содержать аннотацию обобщения (исключением являются параметры типа по умолчанию, которые рассматриваются далее в главе).

```ts
class Animal<T> {
  constructor(readonly id: T) {}
}

var bird: Animal = new Animal<string>('bird'); // Error
var bird: Animal<string> = new Animal<string>('bird'); // Ok
```

Когда все обязательные параметры типа используются в параметрах конструктора, при создании экземпляра класса аннотацию обобщения можно опускать. В таком случае вывод типов определит принадлежность к типам по устанавливаемым значениям. Если параметры являются необязательными и значение не будет передано, то вывод типов определит принадлежность параметров типа к типу данных `unknown`.

```ts
class Animal<T> {
  constructor(readonly id?: T) {}
}

let bird: Animal<string> = new Animal('bird'); // Ok -> bird: Animal<string>
let fish = new Animal('fish'); // Ok -> fish: Animal<string>
let insect = new Animal(); // Ok -> insect: Animal<unknown>
```

Относительно обобщенных типов существуют такие понятия, как _открытый_ (open) и _закрытый_ (closed) тип. Обобщенный тип в момент определения называется _открытым_.

```ts
class T0<T, U> {} //  T0 - открытый тип
```

Кроме того, обобщенные типы, указанные в аннотации, у которых хотя бы один из аргументов типа является параметром типа, также являются открытыми типами.

```ts
class T1<T> {
  public f: T0<number, T>; // T0 - открытый тип
}
```

И наоборот, если все аргументы типа принадлежат к конкретным типам, то такой обобщенный тип является _закрытым_ типом.

```ts
class T1<T> {
  public f1: T0<number, string>; // T0 - закрытый тип
}
```

Те же самые правила применимы и к функциям, но за одним исключением — вывод типов для примитивных типов определяет принадлежность параметров типа к литеральным типам данных.

```ts
function action<T>(value?: T): T | undefined {
  return value;
}

action<number>(0); // function action<number>(value?: number | undefined): number | undefined
action(0); // function action<0>(value?: 0 | undefined): 0 | undefined

action<string>('0'); // function action<string>(value?: string | undefined): string | undefined
action('0'); // function action<"0">(value?: "0" | undefined): "0" | undefined

action(); // function action<unknown>(value?: unknown): unknown
```

Если параметры типа не участвуют в операциях при создании экземпляра класса и при этом аннотация обобщения не была указана явно, вывод типа теряет возможность установить принадлежность к типу по значению и поэтому устанавливает его принадлежность к типу `unknown`.

```ts
class Animal<T> {
  public name: T;

  constructor(readonly id: string) {}
}

let bird: Animal<string> = new Animal('bird#1');
bird.name = 'bird';
// Ok -> bird: Animal<string>
// Ok -> (property) Animal<string>.name: string

let fish = new Animal<string>('fish#1');
fish.name = 'fish';
// Ok -> fish: Animal<string>
// Ok -> (property) Animal<string>.name: string

let insect = new Animal('insect#1');
insect.name = 'insect';
// Ok -> insect: Animal<unknown>
// Ok -> (property) Animal<unknown>.name: unknown
```

И опять, эти же правила верны и для функций.

```ts
function action<T>(value?: T): T | undefined {
  return value;
}

action<string>('0'); // function action<string>(value?: string | undefined): string | undefined
action('0'); // function action<"0">(value?: "0" | undefined): "0" | undefined
action(); // function action<unknown>(value?: unknown): unknown
```

В случаях, когда обобщенный класс содержит обобщенный метод, параметры типа метода будут затенять параметры типа класса.

```ts
type ReturnParam<T, U> = { a: T; b: U };

class GenericClass<T, U> {
  public defaultMethod<T>(a: T, b?: U): ReturnParam<T, U> {
    return { a, b };
  }

  public genericMethod<T>(a: T, b?: U): ReturnParam<T, U> {
    return { a, b };
  }
}

let generic: GenericClass<
  string,
  number
> = new GenericClass();
generic.defaultMethod('0', 0);
generic.genericMethod<boolean>(true, 0);
generic.genericMethod('0');

// Ok -> generic: GenericClass<string, number>
// Ok -> (method) defaultMethod<string>(a: string, b?: number): ReturnParam<string, number>
// Ok -> (method) genericMethod<boolean>(a: boolean, b?: number): ReturnParam<boolean, number>
// Ok -> (method) genericMethod<string>(a: string, b?: number): ReturnParam<string, number>
```

Стоит заметить, что в _TypeScript_ нельзя создавать экземпляры типов, представляемых параметрами типа.

```ts
interface CustomConstructor<T> {
  new (): T;
}

class T1<T extends CustomConstructor<T>> {
  public getInstance(): T {
    return new T(); // Error
  }
}
```

Кроме того, два типа, определяемые классом или функцией, считаются идентичными вне зависимости от того, являются они обобщенными или нет.

```ts
type T1 = {};
type T1<T> = {}; // Error -> Duplicate identifier

class T2<T> {}
class T2 {} // Error -> Duplicate identifier

class T3 {
  public m1<T>(): void {}
  public m1(): void {} // Error -> Duplicate method
}

function f1<T>(): void {}
function f1(): void {} // Error -> Duplicate function
```

###### Параметры типа - extends (generic constraints)

Помимо того, что параметры типа можно указывать в качестве конкретного типа, они также могут расширять другие типы, в том числе и другие параметры типа. Такой механизм требуется, когда значения внутри обобщенного типа должны обладать ограниченным набором признаков. Ключевое слово `extends` размещается левее расширяемого типа и правее идентификатора параметра типа `<T extends Type>`. В качестве расширяемого типа может быть указан как конкретный тип данных, так и другой параметр типа. При чем если один параметр типа расширяет другой, нет разницы, в каком порядке они объявляются. Если параметр типа ограничен другим параметром типа, то такое ограничение называют _неприкрытым ограничением типа_ (_naked type constraint_),

```ts
class T1<T extends number> {}
class T2<T extends number, U extends T> {} // неприкрытое ограничение типа
class T3<U extends T, T extends number> {}
```

Механизм расширения требуется в тех случаях, в которых параметр типа должен обладать заданными характеристиками, необходимыми для выполнения конкретных операций над этим типом.

Для примера рассмотрим случай, когда в коллекции `T` (`Collection<T>`) объявлен метод получения элемента по имени (`getItemByName`).

```ts
class Collection<T> {
  private itemAll: T[] = [];

  public add(item: T): void {
    this.itemAll.push(item);
  }

  public getItemByName(name: string): T | undefined {
    return this.itemAll.find((item) => item.name === name); // Error -> Property 'name' does not exist on type 'T'
  }
}
```

При операции поиска в массиве возникнет ошибка. Это происходит по причине того, что в типе `T` не описано свойство `name`. Для того чтобы ошибка исчезла, тип `T` должен расширить тип, в котором описано необходимое свойство `name`. В таком случае предпочтительней будет вариант объявления интерфейса `IName` с последующим его расширением.

```ts
interface IName {
  name: string;
}

class Collection<T extends IName> {
  private itemAll: T[] = [];

  public add(item: T): void {
    this.itemAll.push(item);
  }

  public getItemByName(name: string): T | undefined {
    return this.itemAll.find((item) => item.name === name); // Ok
  }
}

abstract class Animal {
  constructor(readonly name: string) {}
}

class Bird extends Animal {}
class Fish extends Animal {}

let birdCollection: Collection<Bird> = new Collection();
birdCollection.add(new Bird('raven'));
birdCollection.add(new Bird('owl'));

let raven: Bird = birdCollection.getItemByName('raven'); // Ok

let fishCollection: Collection<Fish> = new Collection();
fishCollection.add(new Fish('shark'));
fishCollection.add(new Fish('barracuda'));

let shark: Fish = fishCollection.getItemByName('shark'); // Ok
```

Пример, когда параметр типа расширяет другой параметр типа, будет рассмотрен немного позднее.

Также нелишним будет заметить, что когда параметр типа расширяет другой тип, то в качестве аргумента типа можно будет передать только совместимый с ним тип.

```ts
interface Bird {
  fly(): void;
}
interface Fish {
  swim(): void;
}

interface IEgg<T extends Bird> {
  child: T;
}

let v1: IEgg<Bird>; // Ok
let v2: IEgg<Fish>; // Error -> Type 'Fish' does not satisfy the constraint 'Bird'
```

Кроме того, расширять можно любые подходящие для этого типы, полученные любым доступным путем.

```ts
interface IAnimal {
  name: string;
  age: number;
}

let animal: IAnimal;

class Bird<T extends typeof animal> {} // T extends IAnimal
class Fish<K extends keyof IAnimal> {} // K extends "name" | "age"
class Insect<
  V extends IAnimal[K],
  K extends keyof IAnimal
> {} // V extends string | number
class Reptile<
  T extends number | string,
  U extends number & string
> {}
```

Помимо прочего, одна важная и неочевидная особенность связана с параметром типа, расширяющего `any`. Может показаться, что в таком случае над параметром типа будет возможно производить любые операции, допускаемые типом `any`. Но в реальности это не так. Поскольку `any` предполагает выполнение над собой любых операций, то для повышения типобезопасности подобное поведение для типов, представляемых параметрами типа, было отменено.

```ts
class ClassType<T extends any> {
  private f0: any = {}; // Ok
  private field: T = {}; // Error [0]

  constructor() {
    this.f0.notExistsMethod(); // Ok [1]
    this.field.notExistsMethod(); // Error [2]
  }
}

/**
 * Поскольку параметр типа, расширяющий тип any
 * подрывает типобезопасность программы, то вывод
 * типов такой параметр расценивает как принадлежащий
 * к типу unknown, запрещающий любые операции над собой.
 *
 * [0] тип unknown не совместим с объектным типом {}.
 * [1] Ok на этапе компиляции и Error во время выполнения.
 * [2] тип unknown не описывает метода notExistsMethod().
 */
```

###### Параметры типа - значение по умолчанию = (generic parameter defaults)

Помимо прочего _TypeScript_ позволяет указывать для параметров типа значение по умолчанию.

Значение по умолчанию указывается с помощью оператора равно `=`, слева от которого располагается параметр типа, а справа конкретный тип, либо другой параметр типа `T = Type`. Параметры, которым заданы значения по умолчанию, являются необязательными параметрами. Необязательные параметры типа должны быть перечислены строго после обязательных. Если параметр типа указывается в качестве типа по умолчанию, то ему самому должно быть задано значение по умолчанию, либо он должен расширять другой тип.

```ts
class T1<T = string> {} // Ok
class T2<T = U, U> {} // Error -> необязательное перед обязательным
class T3<T = U, U = number> {} // Ok

class T4<T = U, U extends number> {} // Error -> необязательное перед обязательным
class T5<U extends number, T = U> {} // Ok.
```

Кроме того, можно совмещать механизм установки значения по умолчанию и механизм расширения типа. В этом случае оператор равно `=` указывается после расширяемого типа.

```ts
class T1<T extends T2 = T3> {}
```

В момент, когда тип `T` расширяет другой тип, он получает признаки этого типа. Именно поэтому для параметра типа, расширяющего другой тип, в качестве типа по умолчанию можно указывать только совместимый с ним тип.

Чтобы было проще понять, нужно представить два класса, один из которых расширяет другой. В этом случае переменной с типом суперкласса можно в качестве значения присвоить объект его подкласса, но — не наоборот.

```ts
class Animal {
  public name: string;
}

class Bird extends Animal {
  public fly(): void {}
}

let bird: Animal = new Bird(); // Ok
let animal: Bird = new Animal(); // Error
```

Тот же самый механизм используется для параметров типа.

```ts
class Animal {
  public name: string;
}

class Bird extends Animal {
  public fly(): void {}
}

class T1<T extends Animal = Bird> {} // Ok
// -------(   Animal   ) = Bird

class T2<T extends Bird = Animal> {} // Error
// -------(   Bird   ) = Animal
```

Необходимо сделать акцент на том, что вывод типов обращает внимание на необязательные параметры типа только при работе с аргументами этого обобщенного типа. Чтобы было более понятно, вспомним ещё раз, что механизм ограничения параметров типа с помощью ключевого слова `extends`. Признаки типа, расположенного правее ключевого слова `extends`, рассматриваются не только при сопоставлении аргументов типа, но и при выполнении операций над типом, представленным параметром типа. Простыми словами, вывод типов берет во внимание расширенный тип как снаружи (аргумент типа), так и внутри (параметр типа) обобщенного типа.

```ts
/**
 * T расширяет string...
 */
class A<T extends string> {
  constructor(value?: T) {
    /**
     * ...что заставляет вывод типов рассматривать
     * значение, принадлежащее к нему в качестве строкового
     * как внутри...
     */

    if (value) {
      value.charAt(0); // Ok -> ведь value наделено признаками, присущими типу string
    }
  }
}

// ...так и снаружи
let a0 = new A(); // Ok -> let a0: A<string> string потому что параметр типа ограничен им
let a1 = new A(`ts`); // Ok -> let a1: A<"ts"> literal string потому что он совместим со стринг, но более конкретен
let a2 = new A(0); // Error ->  потому что number не совместим с ограничивающим аргумент типа типом string
```

Тип, который указывается параметру типа в качестве типа по умолчанию, вообще ничего не ограничивает.

```ts
// тип string устанавливается типу T в качестве типа по умолчанию...
class B<T = string> {
  constructor(value?: T) {
    if (value) {
      // ...что не оказывает никакого ограничения ни внутри...
      value.charAt(0); // Error -> тип T не имеет определение метода charAt
    }
  }
}

// ...ни снаружи
let b0 = new B(); // Ok -> let b0: B<string>
let b1 = new B(`ts`); // Ok -> let b1: B<string>
let b2 = new B(0); // Ok -> let b2: B<number>
```

При отсутствии аргументов типа был бы выведен тип `unknown`, а не тип, указанный по умолчанию.

```ts
// с типом по умолчанию
class B<T = string> {
  constructor(value?: T) {}
}

// без типа по умолчанию
class С<T> {
  constructor(value?: T) {}
}

let b = new B(); // Ok -> let b: B<string>
let с = new С(); // Ok -> let с: С<unknown>
```

Не будет лишним также рассмотреть отличия этих двух механизмов при работе вывода типов.

```ts
// ограничение типа T типом string
declare class A<T extends string> {
  constructor(value?: T);
}

// тип string устанавливается типу T в качестве типа по умолчанию
declare class B<T = string> {
  constructor(value?: T);
}

let a0 = new A(); // Ok -> let a0: A<string>
let b0 = new B(); // Ok -> let b0: B<string>

let a1 = new A(`ts`); // Ok -> let a1: A<"ts">
let b1 = new B(`ts`); // Ok -> let b1: B<string>

let a2 = new A<string>(`ts`); // Ok -> let a2: A<string>
let b2 = new B<string>(`ts`); // Ok -> let b2: B<string>

let a3 = new A<number>(0); // Error
let b3 = new B<number>(0); // Ok -> let b3: B<number>
```

###### Параметры типа - как тип данных

Параметры типа, указанные в угловых скобках при объявлении обобщенного типа, изначально не принадлежат ни к одному типу. Несмотря на это, компилятор расценивает параметры типа как совместимые с такими типами, как `any` и `never`, а также с самим собой.

```ts
function f0<T>(p: any): T {
  // Ok, any совместим с T
  return p;
}

function f1<T>(p: never): T {
  // Ok, never совместим с T
  return p;
}

function f2<T>(p: T): T {
  // Ok, T совместим с T
  return p;
}
```

Если обобщенная коллекция в качестве аргумента типа получает тип объединение (`Union`), то все её элементы будут принадлежать к типу объединения. Простыми словами, элемент из такой коллекции не будет, без явного преобразования, совместим ни с одним из вариантов, составляющих тип объединение.

```ts
interface IName {
  name: string;
}

interface IAnimal extends IName {}

abstract class Animal implements IAnimal {
  constructor(readonly name: string) {}
}

class Bird extends Animal {
  public fly(): void {}
}

class Fish extends Animal {
  public swim(): void {}
}

class Collection<T extends IName> {
  private itemAll: T[] = [];

  public add(item: T): void {
    this.itemAll.push(item);
  }

  public getItemByName(name: string): T | undefined {
    return this.itemAll.find((item) => item.name === name); // Ok
  }
}

let collection: Collection<Bird | Fish> = new Collection();
collection.add(new Bird('bird'));
collection.add(new Fish('fish'));

var bird: Bird = collection.getItemByName('bird'); // Error -> Type 'Bird | Fish' is not assignable to type 'Bird'
var bird: Bird = collection.getItemByName('bird') as Bird; // Ok
```

Но операцию приведения типов можно поместить (сокрыть) прямо в метод самой коллекции и тем самым упростить её использование. Для этого метод должен быть обобщенным, а его параметр типа, указанный в качестве возвращаемого из функции, расширять параметр типа самой коллекции.

```ts
// ...

class Collection<T extends IName> {
  private itemAll: T[] = [];

  public add(item: T): void {
    this.itemAll.push(item);
  }

  // 1. параметр типа U должен расширять параметр типа T
  // 2. возвращаемый тип указан как U
  // 3. возвращаемое значение нуждается в явном преобразовании к типу U
  public getItemByName<U extends T>(name: string): U | undefined {
    return this.itemAll.find(
      (item) => item.name === name
    ) as U | undefined; // Ok
  }
}

let collection: Collection<Bird | Fish> = new Collection();
collection.add(new Bird('bird'));
collection.add(new Fish('fish'));

var bird: Bird | undefined = collection.getItemByName('bird'); // Ok
var birdOrFish = collection.getItemByName('bird'); // Bad, var birdOrFish: Bird | Fish
var bird = collection.getItemByName<Bird>('bird'); // Ok, var bird: Bird
```

Сокрытие приведения типов прямо в методе коллекции повысило “привлекательность” кода. Но, все же, в случаях, когда элемент коллекции присваивается конструкции без явной аннотации типа, появляется потребность вызывать обобщенный метод с аргументами типа.

Кроме того, нужно не забывать, что два разных объявления параметров типа несовместимы, даже если у них идентичные идентификаторы.

```ts
class Identifier<T> {
  array: T[] = [];

  method<T>(param: T): void {
    this.array.push(param); // Error, T объявленный в сигнатуре функции не совместим с типом T, объявленным в сигнатуре класса
  }
}
```
##### union / intersection

- - **Union types (`|`):** Позволяют указать, что переменная может быть одного из нескольких типов.

        typescript

        let value: string | number;

    - **Intersection types (`&`):** Позволяют объединить несколько типов в один.

        typescript

        type A = { a: string };

        type B = { b: number };

        type C = A & B; // { a: string, b: number }

  

---

##### any vs unknown

`any` — это тип, который отключает проверку типов для переменной. Его следует использовать только в крайних случаях, когда тип неизвестен или сложно определить. А также для обратной совместимости с библиотеками на js.

`unknown` — это тип, который требует явного приведения перед использованием. В отличие от `any`, он безопаснее, так как не позволяет выполнять операции без проверки типа.

  

##### **Абстрактный класс**

Абстрактный класс — это класс, который не может быть инстанциирован напрямую. Он используется как шаблон для других классов и может содержать абстрактные методы (без реализации).


##### Readonly, Partial, Required, Pick, Record

Чтобы сделать повседневные будни разработчика немного легче, _TypeScript_ реализовал несколько предопределенных сопоставимых типов, как - `Readonly<T>`, `Partial<T>`, `Required<T>`, `Pick<T, K>` и `Record<K, T>`. За исключением `Record<K, T>`, все они являются так называемыми _гомоморфными типами_ (homomorphic types). Простыми словами, _гомоморфизм_ — это возможность изменять функционал, сохраняя первоначальные свойства всех операций. Если на данный момент это кажется сложным, то текущая глава покажет, что за данным термином не скрывается ничего сложного. Кроме того, в ней будет подробно рассмотрен каждый из перечисленных типов.

###### `Readonly<T>` (сделать члены объекта только для чтения)

Сопоставимый тип `Readonly<T>` добавляет каждому члену объекта модификатор `readonly`, делая их тем самым доступными только для чтения.

```ts
// lib.es6.d.ts

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

Наиболее частое применение данного типа можно встретить при определении функций и методов, параметры которых принадлежат к объектным типам. Поскольку объектные типы передаются по ссылке, то с высокой долей вероятности случайное изменение члена объекта может привести к непредсказуемым последствиям.

```ts
interface IPerson {
  name: string;
  age: number;
}

/**
 * Функция, параметр которой не
 * защищен от случайного изменения.
 *
 * Поскольку объектные типы передаются
 * по ссылке, то с высокой долей вероятности
 * случайное изменение поля name нарушит ожидаемый
 * ход выполнения программы.
 */
function mutableAction(person: IPerson) {
  person.name = 'NewName'; // Ok
}

/**
 * Надежная функция, защищающая свои
 * параметры от изменения, не требуя описания
 * нового неизменяемого типа.
 */
function immutableAction(person: Readonly<IPerson>) {
  person.name = 'NewName'; // Error -> Cannot assign to 'name' because it is a read-only property.
}
```

Тип сопоставления `Readonly<T>` является гомоморфным и добавляя свой модификатор `readonly` не влияет на уже существующие модификаторы. Сохранение исходным типом своих первоначальных характеристик (в данном случае — модификаторы), делает сопоставленный тип `Readonly<T>` гомоморфным.

```ts
interface IPerson {
  gender?: string;
}

type Person = Readonly<IPerson>; // type Person = { readonly gender?: string; }
```

В качестве примера можно привести часто встречающейся на практике случай, в котором универсальный интерфейс описывает объект, предназначенный для работы с данными. Поскольку в львиной доле данные представляются объектными типами, интерфейс декларирует их как неизменяемые, что в дальнейшем при его реализации избавит разработчика от типизации конструкций и тем самым сэкономит для него время на более увлекательные задачи.

```ts
/**
 * Интерфейс необходим для описания экземпляра
 * провайдеров, с которыми будет сопряжено
 * приложение. Кроме того, интерфейс описывает
 * поставляемые данные как только для чтения,
 * что в будущем может сэкономить время.
 */
interface IDataProvider<OutputData, InputData = null> {
  getData(): Readonly<OutputData>;
}

/**
 * Абстрактный класс описание, определяющий
 * поле data, доступный только потомкам как
 * только для чтения. Это позволит предотвратить
 * случайное изменение данных в классах потомках.
 */
abstract class DataProvider<InputData, OutputData = null>
  implements IDataProvider<InputData, OutputData> {
  constructor(protected data?: Readonly<OutputData>) {}

  abstract getData(): Readonly<InputData>;
}

interface IPerson {
  firstName: string;
  lastName: string;
}

interface IPersonDataProvider {
  name: string;
}

class PersonDataProvider extends DataProvider<
  IPerson,
  IPersonDataProvider
> {
  getData() {
    /**
     * Работая в теле потомков DataProvider,
     * будет не так просто случайно изменить
     * данные, доступные через ссылку this.data
     */
    let [firstName, lastName] = this.data.name.split(` `);
    let result = { firstName, lastName };

    return result;
  }
}

let provider = new PersonDataProvider({
  name: `Ivan Ivanov`,
});
```

###### `Partial<T>` (сделать все члены объекта необязательными)

Сопоставимый тип `Partial<T>` добавляет членам объекта модификатор `?:`, делая их таким образом необязательными.

```ts
// lib.es6.d.ts

type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

Тип сопоставления `Partial<T>` является гомоморфным и не влияет на существующие модификаторы, а лишь расширяет модификаторы конкретного типа.

```ts
interface IPerson {
  readonly name: string; // поле помечено как только для чтения
}

/**
 * добавлен необязательный модификатор
 * и при этом сохранен модификатор readonly
 *
 * type Person = {
 *  readonly name?: string;
 * }
 */
type Person = Partial<IPerson>;
```

Представьте приложение, зависящее от конфигурации, которая как полностью, так и частично, может быть переопределена пользователем. Поскольку работоспособность приложения завязана на конфигурации, члены, определенные в типе, представляющем её, должны быть обязательными. Но поскольку пользователь может переопределить лишь часть конфигурации, функция, выполняющая её слияние с конфигурацией по умолчанию, не может указать в аннотации типа уже определенный тип, так как его члены обязательны. Описывать новый тип слишком утомительно. В таких случаях необходимо прибегать к помощи `Partial<T>`.

```ts
interface IConfig {
  domain: string;
  port: '80' | '90';
}

const DEFAULT_CONFIG: IConfig = {
  domain: `https://domain.com`,
  port: '80',
};

function createConfig(config: IConfig): IConfig {
  return Object.assign({}, DEFAULT_CONFIG, config);
}

/**
 * Error -> Поскольку в типе IConfig все
 * поля обязательные, данную функцию
 * не получится вызвать с частичной конфигурацией.
 */
createConfig({
  port: '80',
});

function createConfig(config: Partial<IConfig>): IConfig {
  return Object.assign({}, DEFAULT_CONFIG, config);
}

/**
 * Ok -> Тип Partial<T> сделал все члены,
 * описанные в IConfig необязательными,
 * поэтому пользователь может переопределить
 * конфигурацию частично.
 */
createConfig({
  port: '80',
});
```

###### `Required<T>` (сделать все необязательные члены обязательными)

Сопоставимый тип `Required<T>` удаляет все необязательные модификаторы `?:`, приводя члены объекта к обязательным. Достигается это путем удаления необязательных модификаторов при помощи механизма _префиксов - и +_ рассматриваемого в главе [Оператор keyof, Lookup Types, Mapped Types, Mapped Types - префиксы + и -](042.md)).

```ts
type Required<T> = {
  [P in keyof T]-?: T[P];
};
```

Тип сопоставления `Required<T>` является полной противоположностью типу сопоставления `Partial<T>`.

```ts
interface IConfig {
  domain: string;
  port: '80' | '90';
}

/**
 * Partial добавил членам IConfig
 * необязательный модификатор ->
 *
 * type T0 = {
 *  domain?: string;
 *  port?: "80" | "90";
 * }
 */
type T0 = Partial<IConfig>;

/**
 * Required удалил необязательные модификаторы
 * у типа T0 ->
 *
 * type T1 = {
 *  domain: string;
 *  port: "80" | "90";
 * }
 */
type T1 = Required<T0>;
```

Тип сопоставления `Required<T>` является гомоморфным и не влияет на модификаторы, отличные от необязательных.

```ts
interface IT {
  readonly a?: number;
  readonly b?: string;
}

/**
 * Модификаторы readonly остались
 * на месте ->
 *
 * type T0 = {
 *  readonly a: number;
 *  readonly b: string;
 * }
 */
type T0 = Required<IT>;
```

###### Pick (отфильтровать объектный тип)

Сопоставимый тип `Pick<T, K>` предназначен для фильтрации объектного типа, ожидаемого в качестве первого параметра типа. Фильтрация происходит на основе ключей, представленных множеством литеральных строковых типов, ожидаемых в качестве второго параметра типа.

```ts
// lib.es6.d.ts

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

Простыми словами, результатом преобразования `Pick<T, K>` будет являться тип, состоящий из членов первого параметра, идентификаторы которых указаны во втором параметре.

```ts
interface IT {
  a: number;
  b: string;
  c: boolean;
}

/**
 * Поле "с" отфильтровано ->
 *
 * type T0 = { a: number; b: string; }
 */
type T0 = Pick<IT, 'a' | 'b'>;
```

Стоит заметить, что в случае указания несуществующих ключей возникнет ошибка.

```ts
interface IT {
  a: number;
  b: string;
  c: boolean;
}

/**
 * Error ->
 *
 * Type '"a" | "U"' does not satisfy the constraint '"a" | "b" | "c"'.
 * Type '"U"' is not assignable to type '"a" | "b" | "c"'.
 */
type T1 = Pick<IT, 'a' | 'U'>;
```

Тип сопоставления `Pick<T, K>` является гомоморфным и не влияет на существующие модификаторы, а лишь расширяет модификаторы конкретного типа.

```ts
interface IT {
  readonly a?: number;
  readonly b?: string;
  readonly c?: boolean;
}

/**
 * Модификаторы readonly и ? сохранены ->
 *
 * type T2 = { readonly a?: number; }
 */
type T2 = Pick<IT, 'a'>;
```

Примером, который самым первым приходит в голову, является функция `pick`, в задачу которой входит создавать новый объект путем фильтрации членов существующего.

```ts
function pick<T, K extends string & keyof T>(
  object: T,
  ...keys: K[]
) {
  return Object.entries(object) // преобразуем объект в массив [идентификатор, значение]
    .filter(([key]: Array<K>) => keys.includes(key)) // фильтруем
    .reduce(
      (result, [key, value]) => ({
        ...result,
        [key]: value,
      }),
      {} as Pick<T, K>
    ); // собираем объект из прошедших фильтрацию членов
}

let person = pick(
  {
    a: 0,
    b: ``,
    c: true,
  },
  `a`,
  `b`
);

person.a; // Ok
person.b; // Ok
person.c; // Error -> Property 'c' does not exist on type 'Pick<{ a: number; b: string; c: boolean; }, "a" | "b">'.
```

###### Record<K, T> (динамически определить поле в объектном типе)

Сопоставимый тип `Record<K, T>` предназначен для динамического определения полей в объектном типе. Данный тип определяет два параметра типа. В качестве первого параметра ожидается множество ключей, представленных множеством `string` или `Literal String` - `Record<"a", T>` или `Record<"a" | "b", T>`. В качестве второго параметра ожидается конкретный тип данных, который будет ассоциирован с каждым ключом.

```ts
// lib.es6.d.ts

type Record<K extends string, T> = {
  [P in K]: T;
};
```

Самый простой пример, который первым приходит в голову, это замена индексных сигнатур.

```ts
/**
 * Поле payload определено как объект
 * с индексной сигнатурой, что позволит
 * динамически записывать в него поля.
 */
interface IConfigurationIndexSignature {
  payload: {
    [key: string]: string;
  };
}

/**
 * Поле payload определено как
 * Record<string, string>, что аналогично
 * предыдущему варианту, но выглядит более
 * декларативно.
 */
interface IConfigurationWithRecord {
  payload: Record<string, string>;
}

let configA: IConfigurationIndexSignature = {
  payload: {
    a: `a`,
    b: `b`,
  },
}; // Ok
let configB: IConfigurationWithRecord = {
  payload: {
    a: `a`,
    b: `b`,
  },
}; // Ok
```

Но, в отличие от индексной сигнатуры типа `Record<K, T>` может ограничить диапазон ключей.

```ts
type WwwConfig = Record<'port' | 'domain', string>;

let wwwConfig: WwwConfig = {
  port: '80',
  domain: 'https://domain.com',

  user: 'User', // Error -> Object literal may only specify known properties, and 'user' does not exist in type 'Record<"port" | "domain", string>'.
};
```

В данном случае было бы даже более корректным использовать `Record<K, T>` в совокупности с ранее рассмотренным типом `Partial<T>`.

```ts
type WwwConfig = Partial<Record<'port' | 'domain', string>>;

let wwwConfig: WwwConfig = {
  port: '80',
  // Ok -> поле domain теперь необязательное
  user: 'User', // Error -> Object literal may only specify known properties, and 'user' does not exist in type 'Record<"port" | "domain", string>'.
};
```

Также не будет лишним упомянуть, что поведение данного типа при определении в объекте с предопределенными членами, идентификаторы которых ассоциированы с типами, отличными от типа, указанного в качестве второго параметра, идентично поведению индексной сигнатуры. Напомню, что при попытке определить в объекте члены, идентификаторы которых будут ассоциированы с типами, отличными от указанных в индексной сигнатуре, возникнет ошибка.

```ts
/**
 * Ok -> поле a ассоциированно с таким
 * же типом, что указан в индексной сигнатуре.
 */
interface T0 {
  a: number;

  [key: string]: number;
}

/**
 * Error -> тип поля a не совпадает с типом,
 * указанным в индексной сигнатуре.
 */
interface T1 {
  a: string; // Error -> Property 'a' of type 'string' is not assignable to string index type 'number'.

  [key: string]: number;
}
```

Данный пример можно переписать с использованием типа пересечения.

```ts
interface IValue {
  a: number;
}

interface IDynamic {
  [key: string]: string;
}

type T = IDynamic & IValue;

/**
 * Error ->
 * Type '{ a: number; }' is not assignable to type 'IDynamic'.
 * Property 'a' is incompatible with index signature.
 * Type 'number' is not assignable to type 'string'.
 */
let t: T = {
  a: 0,
};
```

Аналогичное поведение будет и для пересечения, определяемого типом `Record<K, T>`.

```ts
interface IValue {
  a: number;
}

type T = Record<string, string> & IValue;

/**
 * Error ->
 * Type '{ a: number; }' is not assignable to type 'Record<string, string>'.
 * Property 'a' is incompatible with index signature.
 * Type 'number' is not assignable to type 'string'.
 */
let t: T = {
  a: 0,
};
```


##### Exclude, Extract, NonNullable, ReturnType, InstanceType, Omit

Чтобы сэкономить время разработчиков, в систему типов _TypeScript_ были включены несколько часто требующихся условных типов, каждый из которых будет подробно рассмотрен в этой главе.

###### Exclude<T, U> (исключает из T признаки присущие U)

В результате разрешения условный тип `Exclude<T, U>` будет представлять разницу типа `T` относительно типа `U`. Параметры типа `T` и `U` могут быть представлены как единичным типом, так и множеством `union`.

```ts
// @filename: lib.d.ts

type Exclude<T, U> = T extends U ? never : T;
```

Простыми словами, из типа `T` будут исключены признаки (ключи), присущие также и типу `U`.

```ts
let v0: Exclude<number | string, number | boolean>; // let v0: string
let v1: Exclude<number | string, boolean | object>; // let v1: string|number
let v2: Exclude<'a' | 'b', 'a' | 'c'>; // let v2: "b"
```

В случае, если оба аргумента типа принадлежат к одному и тому же типу данных, `Exclude<T, U>` будет представлен типом `never`.

```ts
let v4: Exclude<number | string, number | string>; // let v4: never
```

Его реальную пользу лучше всего продемонстрировать на реализации функции, которая на входе получает два разных объекта, а на выходе возвращает новый объект, состоящий из членов, присутствующих в первом объекте, но отсутствующих во втором. Аналог функции `difference` из широко известной библиотеки _lodash_.

```ts
declare function difference<T, U>(
  a: T,
  b: U
): Pick<T, Exclude<keyof T, keyof U>>;

interface IA {
  a: number;
  b: string;
}
interface IB {
  a: number;
  c: boolean;
}

let a: IA = { a: 5, b: '' };
let b: IB = { a: 10, c: true };

interface IDifference {
  b: string;
}

let v0: IDifference = difference(a, b); // Ok
// Error -> Property 'a' is missing in type 'Pick<IA, "b">' but required in type 'IA'.
let v1: IA = difference(a, b);
// Error -> Type 'Pick ' is missing the following properties from type 'IB': a, c
let v2: IB = difference(a, b);
```

###### Extract<T, U> (общие для двух типов признаки)

В результате разрешения условный тип `Extract<T, U>` будет представлять пересечение типа `T` относительно типа `U`. Оба параметра типа могут быть представлены как обычным типом, так и `union`.

```ts
// @filename: lib.d.ts

type Extract<T, U> = T extends U ? T : never;
```

Простыми словами, после разрешения `Extract<T, U>` будет принадлежать к типу определяемого признаками (ключами), присущими обоим типам. То есть, тип `Extract<T, U>` является противоположностью типа `Exclude<T, U>`.

```ts
let v0: Extract<number | string, number | string>; // let v0: string | number
let v1: Extract<number | string, number | boolean>; // let v1: number
let v2: Extract<'a' | 'b', 'a' | 'c'>; // let v2: "a"
```

В случае, когда общие признаки отсутствуют, тип `Extract<T, U>` будет представлять тип `never`.

```ts
let v3: Extract<number | string, boolean | object>; // let v3: never
```

Условный тип `Extract<T, U>` стоит рассмотреть на примере реализации функции, принимающей два объекта и возвращающей новый объект, состоящий из членов первого объекта, которые также присутствуют и во втором объекте.

```ts
declare function intersection<T, U>(
  a: T,
  b: U
): Pick<T, Extract<keyof T, keyof U>>;

interface IA {
  a: number;
  b: string;
}
interface IB {
  a: number;
  c: boolean;
}

let a: IA = { a: 5, b: '' };
let b: IB = { a: 10, c: true };

interface IIntersection {
  a: number;
}

let v0: IIntersection = intersection(a, b); // Ok
// Error -> Property 'b' is missing in type 'Pick<IA, "a">' but required in type 'IA'.
let v1: IA = intersection(a, b);
// Error -> Property 'c' is missing in type 'Pick<IA, "a">' but required in type 'IB'.
let v2: IB = intersection(a, b);
```

###### `NonNullable<T>` (удаляет типы null и undefined)

Условный тип `NonNullable<T>` служит для исключения из типа признаков типов `null` и `undefined`. Единственный параметр типа может принадлежать как к обычному типу, так и множеству определяемого тип `union`.

```ts
// @filename: lib.d.ts

type NonNullable<T> = T extends null | undefined
  ? never
  : T;
```

Простыми словами, данный тип удаляет из аннотации типа такие типы, как `null` и `undefined`.

```ts
let v0: NonNullable<string | number | null>; // let v0: string | number
let v1: NonNullable<string | undefined | null>; // let v1: string
let v2: NonNullable<string | number | undefined | null>; // let v2: string | number
```

В случае, когда тип, выступающий в роли единственного аргумента типа, принадлежит только к типам `null` и\или `undefined`, `NonNullable<T>` представляет тип `never`.

```ts
let v3: NonNullable<undefined | null>; // let v3: never
```

###### `ReturnType<T>` (получить тип значения, возвращаемого функцией)

Условный тип `ReturnType<T>` служит для установления возвращаемого из функции типа. В качестве параметра типа должен обязательно выступать _функциональный тип_.

```ts
// @filename: lib.d.ts

type ReturnType<
  T extends (...args: any) => any
> = T extends (...args: any) => infer R ? R : any;
```

На практике очень часто требуется получить тип, к которому принадлежит значение, возвращаемое из функции. Единственное, на что стоит обратить внимание, что в случаях, когда тип возвращаемого из функции значения является параметром типа, у которого отсутствуют хоть какие-то признаки, то тип `ReturnType<T>` будет представлен пустым объектным типом `{}`.

```ts
let v0: ReturnType<() => void>; // let v0: void
let v1: ReturnType<() => number | string>; // let v1: string|number
let v2: ReturnType<<T>() => T>; // let v2: {}
let v3: ReturnType<<
  T extends U,
  U extends string[]
>() => T>; // let v3: string[]
let v4: ReturnType<any>; // let v4: any
let v5: ReturnType<never>; // let v5: never
let v6: ReturnType<Function>; // Error
let v7: ReturnType<number>; // Error
```

###### `InstanceType<T>` (получить через тип класса тип его экземпляра)

Условный тип `InstanceType<T>` предназначен для получения типа экземпляра на основе типа, представляющего класс. Параметр типа `T` должен обязательно принадлежать к _типу класса_.

```ts
// @filename: lib.d.ts

type InstanceType<
  T extends new (...args: any) => any
> = T extends new (...args: any) => infer R ? R : any;
```

В большинстве случаев идентификатор класса задействован в приложении в качестве типа его экземпляра.

```ts
class Animal {
  move(): void {}
}

/**
 * Тип Animal представляет объект класса,
 * то есть его экземпляр, полученный при
 * помощи оператора new.
 */
function f(animal: Animal) {
  type Param = typeof Animal;

  // здесь Param представляет экземпляр типа Animal
}
```

Но сложные приложения часто требуют динамического создания своих компонентов. В таких случаях фабричные функции работают не с экземплярами классов, а непосредственно с самими классами.

Стоит напомнить, что в _JavaScript_ классы это всего лишь _синтаксический сахар_ над старой доброй _функцией конструктором_. И, как известно, объект функции конструктора представляет объект класса, содержащего ссылку на прототип, который и представляет экземпляр. Другими словами, в _TypeScript_ идентификатор класса, указанный в аннотации типа, представляет описание прототипа. Чтобы получить тип самого класса, необходимо выполнить над идентификатором класса _запрос типа_.

```ts
class Animal {
  move(): void {}
}

type Instance = Animal;
type Class = typeof Animal;

type MoveFromInstance = Instance['move']; // Ok ->() => void
type MoveFromClass = Class['move']; // Error -> Property 'move' does not exist on type 'typeof Animal'.
```

Таким образом, грамотно вычислить тип экземпляра в фабричной функции можно при помощи типа `InstanceType<T>`.

```ts
class Animal {
  move(): void {}
}

function factory(Class: typeof Animal) {
  type Instance = InstanceType<Class>;

  let instance: Instance = new Class(); // Ok -> let instance: Animal
}
```

Хотя можно прибегнуть и к менее декларативному способу - к запросу типа свойства класса `prototype`.

```ts
function factory(Class: typeof Animal) {
  type Instance = Class['prototype'];

  let instance: Instance = new Class(); // Ok -> let instance: Animal
}
```

И последнее, о чем стоит упомянуть, что результат получения типа непосредственно через `any` и `never` будет представлен ими же. Остальные случаи приведут к возникновению ошибки.

```ts
class Animal {}

let v0: InstanceType<any>; // let v0: any
let v1: InstanceType<never>; // let v1: never
let v2: InstanceType<number>; // Error
```

###### `Parameters<T>` (получить тип размеченного кортежа, описывающий параметры функционального типа)

Расширенный тип `Parameters<T>` предназначен для получения типов, указанных в аннотации параметров функции. В качестве аргумента типа ожидается _функциональный тип_, на основе которого будет получен размеченный кортеж, описывающий параметры этого функционального типа.

```ts
type Parameters<
  T extends (...args: any[]) => any
> = T extends (...args: infer P) => any ? P : never;
```

`Parameters<T>` возвращает типы параметров в виде кортежа.

```ts
function f<T>(
  p0: T,
  p1: number,
  p2: string,
  p3?: boolean,
  p4: object = {}
) {}

/**
 * type FunctionParams = [p0: unknown, p1: number, p2: string, p3?: boolean, p4?: object]
 */
type FunctionParams = Parameters<typeof f>;
```

###### `ConstructorParameters<T>` (получить через тип класса размеченный кортеж, описывающий параметры его конструктора)

Расширенный тип `ConstructorParameters<T>` предназначен для получения типов, указанных в аннотации параметров конструктора.

```ts
type ConstructorParameters<
  T extends new (...args: any[]) => any
> = T extends new (...args: infer P) => any ? P : never;
```

В качестве единственного параметра типа `ConstructorParameters<T>` ожидает тип самого класса, на основе конструктора которого будет получен размеченный кортеж, описывающий параметры этого конструктора.

```ts
class Class<T> {
  constructor(
    p0: T,
    p1: number,
    p2: string,
    p3?: boolean,
    p4: object = {}
  ) {}
}

/**
 * type ClassParams = [p0: unknown, p1: number, p2: string, p3?: boolean, p4?: object]
 */
type ClassParams = ConstructorParameters<typeof Class>;
```

###### Omit<T, K> (исключить из T признаки, ассоциированные с ключами, перечисленными множеством K)

Расширенный тип `Omit<T, K>` предназначен для определения нового типа путем исключения заданных признаков из существующего типа.

```ts
// lib.d.ts

type Omit<T, K extends string | number | symbol> = {
  [P in Exclude<keyof T, K>]: T[P];
};
```

В качестве первого аргумента типа тип `Omit<T, K>` ожидает тип данных, из которого будут исключены признаки, связанные с ключами, переданными в качестве второго аргумента типа.

Простыми словами, к помощи `Omit<T, K>` следует прибегать в случаях необходимости определения типа, представляющего некоторую часть уже существующего типа.

```ts
type Person = {
  firstName: string;
  lastName: string;

  age: number;
};

/**
 * Тип PersonName представляет только часть типа Person
 *
 * type PersonName = {
 *  firstName: string;
 *  lastName: string;
 * }
 */
type PersonName = Omit<Person, 'age'>; // исключение признаков, связанных с полем age, из типа Person
```



##### Infer

##### never

