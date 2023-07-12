# Pattern matching meets JS
> sorta.... kinda...

# Несколько слов о ...

**Pattern matching** (aka Сопоставление с примером) - это механизм сравнения значений с определенным примером. Является крайне удобным лаконичным инструментом преимущественно (до недавнего времени) языков программирования (ЯП) претендующих на звание *функциональных* как то [Scala](https://docs.scala-lang.org/tour/pattern-matching.html), [Haskell](https://en.wikibooks.org/wiki/Haskell/Pattern_matching), [OCaml](https://v2.ocaml.org/manual/patterns.html), [Erlang](https://www.erlang.org/docs/26/reference_manual/patterns.html)(где это вообще один из "языкообразующих" механизмов), [CLisp](https://lispcookbook.github.io/cl-cookbook/pattern_matching.html) и др. Но некоторое время назад этот инструмент в том или ином виде начал появляться и других ЯП.

Какую задачу решает pattern matching в программе? Давайте посмотрим на примере из Scala:

```scala
def constantsPatternMatching(constant: Any): String = {
  constant match {
    case 0 => "I'm equal to zero"
    case 4.5d => "I'm a double"
    case false => "I'm the contrary of true"
    case _ => s"I'm unknown and equal to $constant"
  }
}
```

Легко заметить, что данная функция исполняет ту или иную логическую *ветку* в зависимости от того, с чем совпадает `constant`. Другими словами: `pattern matching обеспечивает ветвление логики исполнение`. Именно так - почти как `if` или `switch`, известных во множестве языков программирования, особенно Си-подобных. Однако важно отметить, что в большинстве реализаций в разных ЯП, pattern matching является выражением и возвращает результат исполнения логики в одной из веток. В показанном выше примере результатом сопоставления будет одна из четырёх строк.

Разумеется данный пример можно переписать на любом (или почти любом) языке программирования без использования pattern matching'а, с помощью `if` или `switch`:

```scala
def constantsPatternMatching(constant: Any): String = {  
  if (constant.isInstanceOf[Int] && constant == 0) {  
    return "I'm equal to zero"  
  } else if (constant.isInstanceOf[Double] && constant == 4.5d) {  
    return "I'm a double"  
  } else if (constant.isInstanceOf[Boolean] && constant == false) {  
    return "I'm the contrary of true"  
  } else {  
    return "I'm unknown and equal to" + constant  
  }  
}
```

Обратите внимание насколько более громоздкая конструкция получилась по сравнению с версией использующей сопоставление с образцом. И чем более комплексные проверки заложены для условий ветвления и чем больше количественно условий - тем нагляднее будет разница.

# О чём это заметка?
Сопоставление с образцом выглядит очень привлекательно для использования при решении задач и обычно реализован на уровне ЯП. Однако JavaScript (далее JS) не обладает данным механизмом на нативном уровне(по крайней мере на момент написания заметки).
В этой заметке автор предпринял попытку реализовать подобие механизма сопоставления с образцом с помощью имеющихся в языке конструкций, потому что... почему бы и нет?

В конце заметки будут расположены ссылки на другие реализации механизма сопоставления с образцом в JS от других авторов.

# Внешний вид и внутренний мир

Перед тем как начать, автор хотел бы оставить небольшой дисклеймер, о том что внешний вид или форма, с которой предположительно будет взаимодействовать пользователь, а так же непосредственно реализация механизма сопоставления с образцом не являются единственно верными. Автор использовал те формы и существующие механизмы языка, которые нашёл удобными для демонстрации идеи.


# Начинаем начинать

## Группы для сопоставления

Автор выделил 4 группы сущностей в JS, которые могли бы быть использованы при сопоставлении с образцом: 
- примитивы (`undefined`, `boolean`, `number`, `bigint`, `string`, `symbol`), их обёртки (`String`, `BigInt`, `Symbol`, `Number`, `Boolean`) и `null`
- массивы (`[1,2,3,4,5]`)
- ассоциативные массивы(`{a:1; b:2; c:3}`) и объекты пользовательских классов (`new SomeClass()`)


## Пользовательский интерфейс

Для использования механизма сопоставления с образцом пользователем, можно выделить 3 функции и 2 константы:
Функции:
- функция, принимающая в качестве аргументов: образец, с которым нужно будет сопоставлять, `guard`'ы (дополнительные условия при сопоставлении) и функцию с логикой, которую надо будет выполнить, если произошло совпадение. В виду того, что `case` является ключевым словом в JS - функция будет называться `ca$e` =). Результатом работы функции `ca$e` будет та самая функция с логикой, в случае если сопоставление пройдёт успешно. 
- функция `match`, которая принимает в качестве аргументов "образец" и последовательность из `ca$e` функций. Результатом будет либо результат применения одной из результирующих функций `ca$e`, либо выбросится исключение, если ни один из образцов не подошёл
- функция обработки по умолчанию `el$e`, такая функция передаётся последней, она всегда истинна при сопоставлении и будет выполнена, если ни одна из предыдущих не подошла - её стоит использовать, чтобы избежать исключительной ситуации.
Константы:
- `ANY` - она применяется в качестве *wildcard*-значения - т.е. без разницы, что будет на её месте в сопоставляемой сущности.
- `TAIL` - это константа похожа на `ANY`, только для массивов и распространяется от указанного места и до конца массива.

## Как это должно выглядеть в итоге

```js

let result = 
  match(matchable)(
    ca$e(null)(_ => "null"),
    ca$e(2)(n => `${n} + 2 == 4`),
    ca$e([1, 2, ANY, 4, TAIL], (arr => arr.lehth > 10))(arr => "long array"),
    el$e(elseCase => `matching not found for ${elseCase}`)
)
```

Автор хотел бы напомнить, что внешний вид обусловлен исключительно предпочтениями автора и может быть разным - это не сильно повлияет на логику.


Разберём подробнее внешний вид используемых функций:

```js
match(some_value)(...ca$es)

ca$e(pattern, ...guards)(success_match_function)

el$e(matching_fallback_function)
```

Правила комбинирования следующие:
- `match` используя частичное применение замыкает в себе `some_value` и дальше по очереди применяет её к каждому `ca$e`. `match` может проверять сколько угодно `ca$e` функций.
- `ca$e` первым аргументом принимает образец для сопоставления, а следующие аргументы считаются `guard`'ами. `ca$e` замыкает их и ожидает далее функцию, которая применится в случае успешного сопоставления.
- `el$e` не предполагает никакой логики по сопоставлению и просто принимает функцию, которая применится к сопоставляемой сущности если ни одна из `ca$e` не будет успешной. `el$e` условно можно представить в виде `ca$e(ANY)(matching_fallback_function)`

# К реализации...

## ANY

`ANY` должна быть достаточно простая для использования, и при этом должна содержать значение, которое **вряд ли** можно встретить снаружи. Например, вот такая:

```js
const ANY = "🍵_there's_could_be_any_value_what_you_can_imagine_🍵"
```

> Согласитесь - вы вряд ли ожидаете встретить чашечку чая в реальном коде?

## TAIL

Аналогично с `ANY`:

```js
const TAIL="🍵_there's_tail_of_array_just_don't_care_what's_could_be_🍵"
```

## match

Функция `match` прежде всего должна принимать `item` для дальнейшего сопоставления, и далее, чтобы принять последовательность из `ca$e` функций, `match` возвращает функцию в качестве результата, в которой будет выполняться основная логика: 

```js
function match(item) {
  function match_cases(item, [head_case_func, ...tail_case_funcs]) {
    if (head_case_func == undefined) return null
    return head_case_func(item) || match_cases(item, tail_case_funcs)
  }
  
  return (...cases) => {
    const result_f = match_cases(item, cases)
    if (result_f != null) return result_f(matchable)
    throw Error("Match ERROR!!!");
  }
}
```

Здесь `match_cases` вспомогательная функция, которая рекурсивно применяет `item` к переданным `ca$e` функциям пока первая из них ни вернёт не `null` или пока последовательность `ca$e` функций не опустеет. 

Обратите внимание, что сопоставление `item` не начнётся, пока не переданы `ca$e` функции.

## ~~case~~ ... простите ca$e

Для начала определимся с аргументами и возвращаемыми значениями функции `ca$e` в целом:

```js
function ca$e(case_pattern, ...guards) {
  return (case_func) =>
        (matchable) => {
        // **magic**
        return null            
        }
  }
}
```

Этот вид соответствует описанному выше шаблону 
`ca$e(pattern, ...guards)(success_match_function)`. Последняя возвращаемая функция нужна, для запуска логики сопоставления. `matchable` будет передавать функция [`match`](#match) при вызове `head_case_func`. 

Теперь приступим к реализации части **magic**.
У нас есть шаблон для сопоставления (`case_pattern`) и есть то с чем нужно сопоставлять. Можно для начала сравнить - вдруг они строго говоря равны? И тогда не нужно ничего делать - только проверить `guards` условия и вернуть `case_func` как результат, т.е. сопоставление прошло успешно. 

### Равенство

Напишем несколько вспомогательных функций:

```js
function areTheyStrictEqual(matchable, case_pattern) {
  return matchable === case_pattern
}

function checkGuards(guards, matchable) {
  return guards.every(g => g(matchable))
}
```

И добавим эту логику:
```js
function ca$e(case_pattern, ...guards) {
  return (case_func) =>
            (matchable) => {
              if((areTheyStrictEqual(matchable, case_pattern) ||
                  case_pattern === ANY) &&
                  checkGuards(guards, matchable)) {
              return case_func
          }
        // **rest part of magic**
        return null            
        }
  }
}
```
> Нужно так же учесть случай, когда в качестве образца передают `ANY` 

Уже сейчас мы можем проверить работу на простых значениях:

```js
  
let result = match(1)(
        ca$e(1)(one => `It's work! One!!! ${one}`)
      )

console.log(result) // It's work! One!!! 1
```

Если мы передадим что-то что не совпадает с образцом мы получим исключение:
```js
let result = match(2)(
        ca$e(1)(one => `It's work! One!!! ${one}`)
      )
// Error: Match ERROR!!!
console.log(result)
```

Пока всё ожидаемо. Продолжаем...

### Массивы

Напишем логику по сопоставлению массивов. Сопоставление будет считаться успешным если один и те же элементы массивов расположены в одном и том же порядке. Но для начала нужно выяснить, что `matchable` это массив. Напишем вспомогательную функцию  `areEveryArray` и затем немного уменьшим количество магии:

```js
function areEveryArray(...maybeArrays) {
  return maybeArrays.length > 0 && maybeArrays.every(Array.isArray)
}


function ca$e(case_pattern, ...guards) {
  return (case_func) =>
            (matchable) => {
              if((areTheyStrictEqual(matchable, case_pattern) ||
                case_pattern === ANY) &&
                checkGuards(guards, matchable)) {
                return case_func
              }
              if(areEveryArray(matchable, case_pattern) &&
                checkArraysRec(matchable, case_pattern) &&
                checkGuards(guards, matchable)) {
                return case_func
              }
            // **rest part of magic**
            return null            
            }
  }
}
```

`checkArraysRec` - эта функций, которая будет заниматься сопоставлением именно массивов:
```js
function checkArraysRec(matchable_array , case_pattern_array) {
    if([matchable_array, case_pattern_array].every(a => a.length == 0)) return true //(1)
    let [head_m, ...tail_m ] = matchable_array
    let [head_cp, ...tail_cp] = case_pattern_array
    if(head_cp === TAIL) return true //(2)
    if(head_cp != ANY && !areTheyStrictEqual(head_m, head_cp)) return false //(3)
    return checkArraysRec(tail_m, tail_cp) //(4)
}
```

Пройдёмся по условиям:
1. Если оба массива пустые: сопоставление завершено и не было найдено расхождений - возвращается `true`. Иначе продолжаем сопоставление.
2. Если образец для сопоставления это константа `TAIL`: не важно дальнейшее сравнение - возвращаем `true`. Иначе продолжаем сопоставление.
3. Если образец для сопоставления это не `ANY` и не равно сопоставляемому значению (пока что оставим это простое условие): найдено расхождение и нет смысла продолжать сопоставление - возвращается `false`
4. Если ни одно из предыдущих условий не выполнено - продолжаем сопоставление.

Проверим:

```js
match([1,2,3])(
  ca$e([2,2,3])(arr => `miss`),
  ca$e([1,2,3])(arr => `[1,2,3] == [${arr}]`)
)
// [1,2,3] == [1,2,3]

match([1,2,3])(
  ca$e([ANY,2,3], (([first, ...tail]) => first < 5))(arr => `first is small`),
  ca$e([1,2,3])(arr => `[1,2,3] == [${arr}]`)
)

// first is small

match([1,2,3])(
  ca$e([1, TAIL], (arr => arr.length < 5))(arr => `lenght is less than 5`),
  ca$e([ANY,2,3], (([first, ...tail]) => firts < 5))(arr => `first is small`),
  ca$e([1,2,3])(arr => `[1,2,3] == [${arr}]`)
)
// lenght is less than 5
```

Выглядит неплохо. Далее реализуем логику сопоставления примитивных типов.

### Примитивы
Напомним какие в JS есть примитивные типы, а так же об их классах обёртках: 

|Тип|Результат `typeof` от переменной|Обёртка|
|---|---|---|
|[Null](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#null_type)|`"object"`([почему](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof#typeof_null))|N/A|
|[Undefined](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#undefined_type)|`"undefined"`|N/A|
|[Boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#boolean_type)|`"boolean"`|[`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)|
|[Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#number_type)|`"number"`|[`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)|
|[BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#bigint_type)|`"bigint"`|[`BigInt`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)|
|[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#string_type)|`"string"`|[`String`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)|
|[Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#symbol_type)|`"symbol"`|[`Symbol`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)|

Таким образом, для определения, что перед нами примитив нужно проверить: что `typeof` от переменной является от одним из перечисленных значений во втором столбце из таблицы или что `instanceof` переменной - одно из значений из третьего столбца.

```js
const PRIMITIVE_AND_WRAPPER = {
  "boolean" : Boolean,
  "number" : Number,
  "bigint" : BigInt,
  "string" : String,
  "symbol" : Symbol
}

function isPrimitive(item) {
    return item === null || 
            ["undefined", ...Object.keys(PRIMITIVE_AND_WRAPPER)]
            .includes(typeof item) 
}

function isPrimitiveWrapper(item) {
    return Object.values(PRIMITIVE_AND_WRAPPER)
            .some(w => item instanceof w)
}
```

> `PRIMITIVE_AND_WRAPPER` понадобится в дальнейшем

И объединим эти функции:
```js
function areEveryPrimitive(...maybePrimitives) {
  return maybePrimitives.length > 0 && 
      maybePrimitives
      .every(e => isPrimitive(e) || isPrimitiveWrapper(e))
}
```

Добавим эту логику в функцию `ca$e`:
```js
function ca$e(case_pattern, ...guards) {
  return (case_func) =>
            (matchable) => {
              if((areTheyStrictEqual(matchable, case_pattern) ||
                case_pattern === ANY) &&
                checkGuards(guards, matchable)) {
                return case_func
              }
              if(areEveryArray(matchable, case_pattern) &&
                checkArraysRec(matchable, case_pattern) &&
                checkGuards(guards, matchable)) {
                return case_func
              }
              if(areEveryPrimitive(matchable, case_pattern) &&
                checkPrimitives(matchable, case_pattern) &&
                checkGuards(guards, matchable)) {
                return case_func
              }
              // **rest part of magic**
              return null            
            }
  }
}
```

`checkPrimitives` по аналогии с `checkArraysRec` занимается непосредственно сопоставлением двух значений. Но перед её реализацией необходимо написать несколько вспомогательных функций:

```js
function sameTypes(matchable, case_value) {
    return typeof matchable === typeof case_value
}

function sameWrapperTypes(matchable, case_pattern) {
    return Object.values(PRIMITIVE_AND_WRAPPER)
            .some(w => matchable instanceof w &&
                  case_pattern instanceof w)
}

function arePrimitiveAndWrapperOrViceVersa(matchable, case_pattern) {
    return Object.entries(PRIMITIVE_AND_WRAPPER)
            .some(([pr, wrap]) =>
              (typeof matchable === pr && 
              case_pattern instanceof wrap) ||
              (typeof case_pattern === pr && 
                matchable instanceof wrap))
}

function areMatchableTypes(matchable, case_pattern) {
  return [sameTypes, 
          sameWrapperTypes, 
          arePrimitiveAndWrapperOrViceVersa]
          .some(f => f(matchable, case_pattern))
}
```

В `areMatchableTypes` считаем, что примитивы сопоставимы, если выполняется одно из условий:
- их типы равны. `sameTypes`
- их обёртки равны. `sameWrapperTypes`
- тип сопоставляемого значения связан с обёрткой значения образца или наоборот. `arePrimitiveAndWrapperOrViceVersa`

Теперь напишем реализацию `checkPrimitives`:

```js
function checkPrimitives(matchable, case_pattern) {
  if (case_pattern == ANY || areTheyStrictEqual(matchable, case_pattern)) return true
return areMatchableTypes(matchable, case_pattern) &&
        areTheyStrictEqual(matchable.toString(), case_pattern.toString())
}
```

Кажется, что здесь всё выглядит достаточно просто, кроме последней строки:

```js
areTheyStrictEqual(matchable.toString(), case_pattern.toString())
```

Глядя на неё, у кого-то может возникнуть вопрос "Зачем?". Зачем сравнивать строковое представление примитивов, особенно учитывая, что чуть выше уже была проверка самих значений?

Ну... это обусловлено "особенной" системой типов и их сравнения в JS:
```js
Symbol(1) === Symbol(1) // false
// Но
Symbol(1).toString() === Symbol(1).toString() // true
```

Разумеется, если этот случай не интересен - можно убрать сравнение 
строковых представлений в конце функции.

Проверим:

```js
match(1)(
    ca$e([1, TAIL], (arr => arr.length < 5))(arr => `lenght is less than 5`),
    ca$e("1")(_ => `It's number one but as string`),
    ca$e(new Number(1))(num_one => `It's number one`),
    ca$e(ANY)(any => `It something different`)
)
// `It's number one`
```

> So far so good

Теперь в проверке массива мы можем поменять условие проверки на более корректное:

```js
function checkArraysRec(matchable_array , case_pattern_array) {
  if([matchable_array, case_pattern_array].every(a => a.length == 0)) return true //(1)
    let [head_m, ...tail_m ] = matchable_array
    let [head_cp, ...tail_cp] = case_pattern_array
    if(head_cp === TAIL) return true //(2)
   // if(head_cp != ANY && !areTheyStrictEqual(head_m, head_cp)) return false //(3)
    if(!checkPrimitives(head_m, head_cp)) return false
    return checkArraysRec(tail_m, tail_cp) //(4)
}
```


### Ассоциативные массивы и пользовательские типы

Как определить, что переменная является ассоциативным массивом или она экземпляр пользовательского класса? Можно сказать, что если переменная не относится ни к примитивам ни к массивам (`Array`), то это ассоциативный массив либо экземпляр пользовательского класса. Логично? Логично.

```js
function areEveryComplexStruct(...maybeComplexStruct){
    return maybeComplexStruct.length > 0 &&
            maybeComplexStruct
            .every(i => !(areEveryPrimitive(i) || areEveryArray(i)))
}
```

Добавим так же вспомогательную функцию для проверки совпадения классов:
```js
function sameCustomClass(matchable, case_pattern) {
    return matchable.constructor.name === case_pattern.constructor.name
}
```

Добавим проверку в функции `ca$e` и уберём оттуда остатки магии:
```js
function ca$e(case_pattern, ...guards) {
  return (case_func) =>
            (matchable) => {
              if((areTheyStrictEqual(matchable, case_pattern) ||
                case_pattern === ANY) &&
                checkGuards(guards, matchable)) {
                return case_func
              }
              if(areEveryArray(matchable, case_pattern) &&
                checkArraysRec(matchable, case_pattern) &&
                checkGuards(guards, matchable)) {
                return case_func
              }
              if(areEveryPrimitive(matchable, case_pattern) &&
                checkPrimitives(matchable, case_pattern) &&
                checkGuards(guards, matchable)) {
                return case_func
              }
              
              if(areEveryComplexStruct(matchable, case_pattern) &&
                            checkComplex(matchable, case_pattern) &&
                            checkGuards(guards, matchable)) {
                            return case_func
                        }
              return null            
            }
  }
}
```

`checkComplex` будет похожа своей идеей на `checkArraysRec`:
```js

function checkComplexRec(matchable, [kv_case_pattern, ...tail_case_pattern]) {
    if(kv_case_pattern == undefined) return true
    let [key_case_pattern, value_case_pattern] = kv_case_pattern
    let matchable_value = matchable[key_case_pattern]
    if(!checkPrimitives(matchable_value, value_case_pattern)) return false
    return checkComplex(matchable, tail_case_pattern)
}

function checkComplex(matchable, case_pattern_complex) {
    if(!sameComplexClass(matchable, case_pattern_complex)) return false
    return checkComplexRec(matchable, Object.entries(case_pattern_complex))
}
```

Проверим решение:
```js
match({x:1, y:2})(
    ca$e([1, TAIL], (a => a.length < 5))(a => `lenght is less than 5`),
    ca$e("1")(_ => `It's number one but as string`),
    ca$e(new Number(1))(num_one => `It's number one`),
    ca$e({x:1, y:2, z: 3})(obj => "xyz"),
    ca$e({x:1, y:2})(obj => "It's complex object"),
    ca$e(ANY)(any => `It something different`)
)
// It's complex object

match({x:1, y:2})(
    ca$e([1, TAIL], (a => a.length < 5))(a => `lenght is less than 5`),
    ca$e("1")(_ => `It's number one but as string`),
    ca$e(new Number(1))(num_one => `It's number one`),
    ca$e({x:1, y:2, z: 3})(obj => "xyz"),
    ca$e({x:ANY, y:2})(obj => "It's complex object and x is ANY"),
    ca$e(ANY)(any => `It something different`)
)
// "It's complex object and x is ANY"

class A {
  constructor(a) {
    this.a = a
  }
}

class B {
  constructor(a) {
    this.a = a
  }
}

match(new A(42))(
  ca$e([1, TAIL], (a => a.length < 5))(a => `lenght is less than 5`),
    ca$e("1")(_ => `It's number one but as string`),
    ca$e(new Number(1))(num_one => `It's number one`),
    ca$e({x:1, y:2, z: 3})(obj => "xyz"),
    ca$e({x:1, y:2})(obj => "It's complex object"),
    ca$e(new B(42))(cls => "Mehhh..."),
    ca$e(new A(42))(cls => "wow such custom class wow 🐶"),
    ca$e(ANY)(any => `It something different`)
)

// "wow such custom class wow 🐶"

```

👍

### Но есть нюанс...

.... который заключается в том что в массиве, в ассоциативном массиве или в пользовательском классе в качестве значений тоже могут быть не только примитивы, но и массивы, ассоциативные массивы или экземпляры пользовательских классов. В этом случае проверка  
```js
if(!checkPrimitives(matchable_value, value_case_pattern)) return false
```
перестанет работать.
Давайте это исправим и напишем функцию, которая решает что за экземпляр пере нами и что с ним делать:
```js
function chooseCheckFunction(matchable, case_pattern) {
    if(areEveryArray(matchable, case_pattern)) return checkArraysRec
    if(areEveryComplexStruct(matchable, case_pattern)) return checkComplex
    if(areEveryPrimitive(matchable, case_pattern)) return checkPrimitives
    return null;
}
```

И применим её.
Для массивов:
```js
function checkArraysRec(matchable_array , case_pattern_array) {
    if([matchable_array, case_pattern_array].every(a => a.length == 0)) return true
    let [head_m, ...tail_m ] = matchable_array
    let [head_cp, ...tail_cp] = case_pattern_array
    if(head_cp === TAIL) return true
    //  if(!checkPrimitives(head_m, head_cp)) return false
    //  return checkArraysRec(tail_m, tail_cp)
    if(head_cp === ANY) return checkArraysRec(tail_m, tail_cp)
    let check_func = chooseCheckFunction(head_m, head_cp)
    return check_func && check_func(head_m, head_cp) &&
         checkArraysRec(tail_m, tail_cp)
}
```

Для "комплексных" сущностей:
```js
function checkComplexRec(matchable, [kv_case_pattern, ...tail_case_pattern]) {
    if(kv_case_pattern == undefined) return true
    let [key_case_pattern, value_case_pattern] = kv_case_pattern
    let matchable_value = matchable[key_case_pattern]
    //if(!checkPrimitives(matchable_value, value_case_pattern)) return false
    //return checkComplex(matchable, tail_case_pattern)
    if(value_case_pattern === ANY) return checkComplexRec(matchable, tail_case_pattern)
    let check_func = chooseCheckFunction(matchable_value, value_case_pattern)
    return check_func && check_func(matchable_value, value_case_pattern) &&
         checkComplexRec(matchable, tail_case_pattern)
}
```

 Проверим, что сопоставление работает со вложенными сущностями:
```js
match({x:1,y: {z: 2}})(
    ca$e({x:1, y:2, z: 3})(obj => "xyz"),
    ca$e({x:1, y: {z: 2}})(obj => "It's very complex object"),
    ca$e(ANY)(any => `It something different`)
)

// It's very complex object
```

В работоспособности предыдущих примеров, а также случаев композиции массивов, ассоциативных массивов и пользовательских классов вы можете убедиться самостоятельно.

### el$e

Функция `el$e` альтернативного действия, срабатывающая, когда ни одна из `ca$e` не сработала достаточно проста - всё, что от неё надо это соблюдать последовательность возвращаемых функций по аналогии с `ca$e`:

```js
function el$e(f) {
  return () => (matchable) => f(matchable)
}
```

## More better

Последнее изменение, которое автор хотел бы сделать это заменить часть логики внутри `ca$e` используя новый метод выбора функции сравнения:
```js
function ca$e(case_pattern, ...guards) {
    return (case_func) => 
                (matchable) => {
                    if(areTheyStrictEqual(matchable, case_pattern) && 
                            checkGuards(guards, matchable)){
                        return case_func
                    }
                    // if(areEveryArray(matchable, case_pattern) &&
                    //     checkArraysRec(matchable, case_pattern) &&
                    //     checkGuards(guards, matchable)) {
                    //     return case_func
                    // }
                    // if(areEveryPrimitive(matchable, case_pattern) &&
                    //     checkPrimitives(matchable, case_pattern) &&
                    //     checkGuards(guards, matchable)) {
                    //     return case_func
                    // }
                    // if(areEveryComplexStruct(matchable, case_pattern) &&
                    //     checkComplex(matchable, case_pattern) &&
                    //     checkGuards(guards, matchable)) {
                    //     return case_func
                    // }
                    let check_func = chooseCheckFunction(matchable, case_pattern)
                        
                    return check_func && 
                            check_func(matchable, case_pattern) &&
                            checkGuards(guards, matchable) ? case_func : null 
                }
}
```

# That all folks!

Автор надеется, что вы не скучали и узнали для себя что-то новое о JS.

# Ссылочки

- [Код из статьи](https://github.com/vial0ft/maccha)
- [match-toy](https://match-toy.github.io/#/), [z](https://z-pattern-matching.github.io/), [TS-Pattern](https://github.com/gvergnaud/ts-pattern) - другие варианты реализации сопоставления с образцом в JS и TS. 
- [предложение](https://github.com/tc39/proposal-pattern-matching) о добавлении механизма сопоставления с образцом на уровне языка
- [is](https://is.js.org/) - набор пред подготовленных предикатов для многих случаев. Просто забавная либа не связанная напрямую с топиком. Возможно пригодится кому-то в тестировании.

- > [Нашли ошибку?](https://github.com/vial0ft/marginal-notes/issues/new)
