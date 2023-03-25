# "M", "F", "A" - words 

## В предыдущих сериях ...

- В части I : были реализованы многие полезные функции для работы со списками как `map`, `filter`, `reduce`, `flatMap`, `concat`, `head`, `tail` и т.д.
- В части II : материал был посвящён замыканиям, каррированию и частичному применению с примерами, которые, как надеется автор, добавили понимание читателю: что это такое, как это работает, и что при помощи этих техник можно делать. 

Была рассмотрена ленивость исполнения и ленивые коллекции, которые вместо захламления памяти генерируются в процессе выполнения программы и могут даже не иметь конца.

Так же был рассмотрен механизм "цепочки вызовов" (`chaining`), и читателю предлагалось реализовать данный механизм, используя только функции и замыкания. Автор надеется, что те читатели, которые решились взяться за эту задачу успешно справились с ней, и предлагает на оценку читателя свой вариант решения:

```js
function chaining (value) {
	return (f) => f(value)
}

function and_then(value) {
	return (f) => chaining(f(value))
}

function getValue(value) {
	return value
}

chaining(5)
(and_then)(x => x + 1)
(and_then)(x => x * 10)
(and_then)(x => x / 2)
(getValue)                // 30
```

Как вы можете догадаться логика в функциях `chaining`, `and_then` и `getValue` может быть произвольной в зависимости от того что нам нужно. Например, если мы работаем со списком - можно применять переданную функцию `f` не ко всему списку, а к каждому элементу, а `getValue` можно заменить на функцию подсчитывающую количество элементов или сумму элементов, если они числовые, или сгруппировать данные по какому-либо признаку.


## Композиция функций

Функции сами по себе обладают огромными возможностями. Но эти возможности можно расширить при помощи композиции функций.

Композировать можно любые две функции при условии: **количество входных аргументов и их порядок одной функции должны соответствовать выходным аргументам другой**. В ЯП со статической типизацией добавляется ещё одно условие: **входные и выходные данные должны совпадать по типам.**

Сообразительный читатель заметит: но ведь у функции может быть только **одно**
результирующее значение?

Обычно да, но есть языки, в которых заявлена возможность вернуть несколько результирующих значений ([python](https://pythonbasics.org/multiple-return/), [go](https://gobyexample.com/multiple-return-values)). В этом случае можно считать, что результатом функции является одно значение, и этим значением является **кортеж значений** (`tuple`)

С этим разобрались. Но что делать, если входных аргументов у функции больше одного? 

Как мы убедились в предыдущей части - с помощью каррирования **любую функцию можно свести к функции с одним аргументом**:

```js

function sum(a,b) {
 return a + b
}
// |
// V

function curring_sum(a) {
	return (b) => a + b
}

```

К тому же: точно так же как из функции можно **вернуть кортеж** из нескольких значений, можно **передать кортеж** из нескольких значений в качестве одного единственного аргумента. Необязательно в вашем любимом языке программирования будет определена специальная структура кортежа - кортеж можно заменить например на список.

В некоторых ЯП - например в JavaScript - есть оператор `...`(`spread`) или его аналоги, который позволяет представить аргументы функции в виде массива и таким образом сделать "функцию с переменным количеством аргументов"

```js
finction prirn_args(... args) {
 console.log(args)
}

print_args(1,2,3,4,5) // [1,2,3,4,5]
print_args(1,2,3) // [1,2,3]

```

Его использование приводит к усложнению механизма композиций функций, когда на стыке между функциями оказывается массив. Иногда проще будет сводить и количество аргументов и количество результирующих значений к одному, используя кортеж или ассоциативный массив.

Но давайте всё-таки реализуем функционал позволяющий делать композицию функций с использованием `spread`:

```js
  function compose_two(a,b) {
    return (...arg) => a(b(...arg))
  }
```

Проверим на примере:

```js
function sum_and_avg (... numbers) {
	let sum = reduce(numbers, (a,b) => a + b)
	return [sum, sum / numbers.length]
}
const sum_all = (numbers) => reduce(numbers, (a, b) => a + b)
let comp_f = compose_two(sum_all, sum_and_avg)

comp_f(1,2,3,4,5) // 18
sum_all(sum_and_avg(1,2,3,4,5)) // 18
```


Как вы можете заметить из реализации `compose` порядок выполнения функций от правой к левой - СНАЧАЛА выполняется `sum_and_avg`, а ЗАТЕМ результат её работы передаётся в `sum_all` - это прямо следует из [определения композиции](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BC%D0%BF%D0%BE%D0%B7%D0%B8%D1%86%D0%B8%D1%8F_%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D0%B9) и считается **традиционным**. Конечно ничто не мешает сделать порядок обратным

``` js  
function compose_two(a,b) {
    return (...arg) => b(a(...arg))
}
```

В этом случае для достижения того же эффекта сделать 

```js
let comp_f = compose_two(sum_and_avg, sum_all) // поменять местами функции

comp_f(1,2,3,4,5) // 18
sum_all(sum_and_avg(1,2,3,4,5)) // 18
```


Но что если у нас не 2, а 3 функции, которые мы хотим скомпозировать?

С этим нет никаких проблем - ведь результат композиции функции **тоже функция**, а значит она может быть скомпозирована с чем-то ещё.

```js
function sum_and_avg (... numbers) {
	let sum = reduce(numbers, (a,b) => a + b)
	return [sum, sum / numbers.length]
}
const sum_all = (numbers) => reduce(numbers, (a, b) => a + b)
const double_it = (x) => x * 2

let comp_f = compose_two(double_it, compose(sum_all, sum_and_avg)) // здесь исрользуется ТРАДИЦИОННАЯ реализация композиции

comp_f(1,2,3,4,5) // 36
```

А если будет больше функций? Неужели придётся писать так? 
`compose(f, (compose(g, ...... (compose a, b)`

Нет - конечно нет, потому что мы вполне можем создать решение, которое не будет зависеть от количества функций участвующих в композиции:

```js

function composeTwo(a, b) {
	return (...args) => a(b(...args))
}

function compose(first, ...rest) {
  if (rest.length == 0){
    return (...args) => first(...args)  
  }
  return (...args) => reduce(rest, (acc, f) => composeTwo(acc,f), first)(...args) 
}
```

Проверим:

```js
const square = (x) => x * x
const timesTwo = (x) => 2 * x
const sum = (x, y) => x + y
const discr = (a, b, c) => b ** 2 - 4 * a * c

compose(square, timesTwo)(2) === square(timesTwo(2)) // true
compose(square, timesTwo, sum)(3, 4) === square(timesTwo(sum(3,4))) // true
compose(discr)(4, 5, 6) === discr(4,5,6) // true
```

👍🏻

Через композицию так же можно сделать цепочку вызовов (чем по сути композиция функций и является).

> Автор уверен, что вы можете это сделать самостоятельно используя функцию `compose` =)


## Типы-"обёртки", о которых говорил всю дорогу автор, или слово на букву "М"

`List`, `Array`, `Option`, `Either`, `Promise` - всё это "обёртки" над данными. "Обёртки", потому что они оборачивают интересующие нас данные и предоставляют удобное api для работы с этими. Если для читателя не очень звучно слово "обёртка" - он может заменить его на "абстракция" 

В предыдущей мы уже частично затронули реализацию типа `List` (`тип != класс`). В этой разберём ещё несколько обёрток.

### `Option` (maybe yes, maybe not - I don't care)

Представим ситуацию нам нужно получить информацию от сервиса, отправляющего запрос на спутники, и возвращающий координаты спутника, чтобы наладить между ними связь.
Вполне может быть, что спутник не ответит по каким-то причинам, и тогда сервис вернёт `null` - в этом случае наладить связь невозможно, и мы вернём `null` вызывающему коду.

```js
	let first_coords = satelliteService.find("satellite1")
	if(first_coords == null) {return null}

	let second_coord = satelliteService.find("satellite1")
	if(second_coord == null) {return null}

	return {
		first_coords,
		second_coords
	}
```

>Существуют специальные механизмы для действий в таких ситуациях в других языках ([js1](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing), [js2](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining), [kotlin](https://kotlinlang.org/docs/null-safety.html#safe-calls)), но это механизмы встроенные в сам язык на уровне синтаксиса и операторов, мы же реализуем это на уровне типов.

Посмотрите на код: каждый раз мы делаем одно и тоже проверяем на `null`, и если всё в порядке продолжаем работу. Автор уверен, что вы помимо кода выше много раз видели или делали точно такие же проверки.

Почему бы не унифицировать эту логику (проверки на `null`) и использовать её там, где она может быть применена.

Для этого был создан тип `Option` (или `Maybe`), который представлен в виде двух подтипов: `Some` (или `Maybe`) и `None` (или `Nothing`).

> Реализовать данный механизм можно используя функции и ассоциативные массивы, но автор посчитал, что в качестве ДЕМОНСТРАЦИИ  ИДЕИ достаточно будет и классов.

Сделаем минимальную реализацию c ex с учётом специфики js:

```js
// file Option.js
class Option {
	// ограничиваем возможность создавать экземпляр класса Option а-ля абстрактный класс
    constructor() {
        if (this.constructor == Option) {
            throw new Error("You can't use constuctor of class Option")
        }
    }

	// делаем статический метод, через который будет создаваться экземпляр подтипа Option 
    static of(value) {
        return new Some(value)
    }

	static ofNullable(maybeNullValue) {
		return maybeNullValue == null ? None : new Some(maybeNullableValue)
	} 

    isEmpty() {
        return this instanceof Nothing
    }

    isDefined() {
        return !this.isEmpty()
    }

    map(f) {
        return this.isEmpty() ? None : new Some(f(this.getValue()))
    }

    flatMap(f) {
        return this.isEmpty() ? None : f(this.getValue())    
    }

	getOrDefault(defaultValue) {
		return this.isEmpty() ? defaultValue : this.getValue() 
	}

	getOrNull() {
		return this.getOrDefault(null)
	}
}

class Some extends Option {

    constructor(value) {
        super()
        this._wrappedValue = value
    }

    getValue() {
        return this._wrappedValue;
    }
}

let None;

class Nothing extends Option {

	// автор нашёл только такой способ в js сделать синглетон
    constructor() {
		if(None != null) { return None}
		super()
}

    getValue() {
        throw new Error("None.getValue()")
    }
}

// None - один единственный - имеет смысл сделать его синглетоном
None = new Nothing()

// здесь должен располагаться код, в котором происходит экспорт:
// классов Option и Some, а так же константы None - инстанса класса Nothing 
```


Это очень условная реализация, не претендующая на единственно верную и лишённую изъянов - главная задача: проиллюстрировать идею.

Теперь давайте попробуем вернуться например к нашей задаче со спутниками и применим наш `Option`.

```js
// возврат вызывающему коду
return Option.ofNullableValue(satelliteService.find("satellite1"))
		.flatMap(coord1 => 
		 Option.ofNullable(satelliteService.find("satellite2"))
		 .map(coord2 => {coord1, coord2}))
		.getOrNull()
```

Да и это всё. Мы поместили логику проверки на `null` внутрь типа `Option` и теперь можем избежать множественных `if(value == null)`. Более того: в виду специфики реализации `map` и `flatMap` - если в какой-то момент мы получили `None` (т.е. где-то попался `null`) - последующая цепочка вызовов игнорируется.

> Можете попробовать дополнить класс `Option` методом `orElse`, который принимает объект `Some`, и если текущий `Option` является `None` - возвращает переданный `Some`. Получается этакое компенсаторное значение или значение по умолчанию.

Теперь давайте представим, что  `satelliteService` был передан нам на поддержку, и мы можем его менять - кроме нас его никто не использует. И давайте представим, что мы изменили его метод `find`, чтобы он возвращал нам `Option`, а не `null`, тогда код выше можно заменить на следующий:

```js
return satelliteService.find("satellite1")
		.flatMap(coord1 => 
				satelliteService.find("satellite2")
				.map(coord2 => {coord1, coord2}))
		.getOrNull()
```

И если фантазировать дальше: представим - мы договорились с вызывающей наш код стороной, что будем возвращать `Option`, а не `null`:
```js
return satelliteService.find("satellite1")
		.flatMap(coord1 => 
				satelliteService.find("satellite2")
				.map(coord2 => {coord1, coord2}))
```

Сравните все три варианта, использующие `Option` с тем который был изначально, и перейдём к рассмотрению ещё одной очень интересной "обёртке".

### `Either`

> ...
> И днём и ночью кот учёный  
> Всё ходит по цепи кругом;  
> Идёт направо — песнь заводит,  
> Налево — сказку говорит.
> ...

`Either` подобно `Option`, только вместо `Some` и `None`, `Either` оперирует подтипами `Left` и `Right`. Где `Left` - условно нежелательная (неуспешная) ветка сценария, а `Right` - желаемая (успешная). Обе ветки содержат в себе дополнительную информацию для своего сценария. И действительно - иногда недостаточно информации о том: успешно или неуспешно прошёл сценарий, как это происходит в случаем с `Option`.

Вернёмся к нашему примеру со спутниками: было неплохо знать от какого спутника мы не смогли получить координаты и по какой причине.

Но сначала реализуем необходимый нам функционал типа `Either`

```js
class Either {
    constructor() {
        if (this.constructor == Either) {
            throw new Error("You can't use constuctor of class Either")
        }
    }

    map(f) {
        return this.isRight() ? new Right(f(this.getValue())) : this // Left не меняем
    }


    flatMap(f) {
        return this.isRight() ? f(this.getValue()) : this // Left не меняем
    }

    swap() {
        return this.isRight() ? new Left(this.getValue()) : new Right(this.getValue())
    }

    getOrDefault(defaultValue) {
		return this.isRight() ? this.getValue() : defaultValue 
	}
}


class Left extends Either {
    constructor(value) {
        super()
        this.value = value
    }

    getValue() {
        return this.value
    }

    isRight() {
        return false
    }

    isLeft() {
        return true
    }
}

class Right extends Either {
    constructor(value) {
        super()
        this.value = value
    }

    getValue() {
        return this.value
    }

    isRight() {
        return true
    }

    isLeft() {
        return false
    }
}
```

Кажется этого нам будет достаточно. А теперь вернёмся к задаче.

Покопавшись в коде `satelliteService`, вы выяснили, что `null` возвращался по нескольким причинам:
1. Ответ от спутника не поступил в течение 10 сек.
2. Спутник ответил в течение 10 сек, но не может передать координаты по какой-то причине (возможно спутник даже укажет причину в детализации ответа).
3. Спутник не понимает вашего запроса и просит повторить запрос в корректном формате.

Видите сколько информации мы теряли просто получая `null` или `Optional` ?
Давайте это исправим. Сделаем допущение, что в случае ошибки у нас будет объект `Error` с полями:
- `code` - код ошибки;
- `details` - текстовое сообщение об ошибке;

В `satelliteService` мы в свою очередь помещаем объект `Error` в `Left` и возвращаем, а если всё прошло успешно, и мы получили координаты - помещаем объект с координатами в `Right` и возвращаем.

Как только мы получили эту информацию - мы сразу передали её команде, вызывающей наш код, которой возвращался `Optional`. Они согласились с форматом и теперь нам нужно поменять код, в котором соединялись в один ответ ответы от двух спутников. Вот он:

```js
return satelliteService.find("satellite1")
		.flatMap(coord1 => 
				satelliteService.find("satellite2")
				.map(coord2 => {coord1, coord2}))

```

И каково же ваше удивление, что вам НЕ НУЖНО МЕНЯТЬ НИ ОДНОЙ СТРОЧКИ КОДА!
> Можете объяснить почему?

Можете налить себе чашечку горячего кофе и немного отдохнуть.

☕

В мире ФП не очень принято бросать ошибки используя `throw`, т.к. это вносит некоторую непредсказуемость в логику программ, и зачастую в языке отсутствуют механизмы описания сигнатуры для качественной защиты функционала от проброса ошибки через несколько уровней вызова. В связи с этим были созданы определённые типы, которые сигнализируют о том, что что-то пошло не так.

Помимо `Either` существуют и другие абстракции для работы с ошибками как то:
- `Try`, который внутри себя пытается отловить ошибку. Если ошибка возникает, то `Try` переключается в `Failure` (содержащим внутри себя ошибку) и дальнейшие вычисления пропускаются, а если ошибки не было, то остаётся `Success` (со значением операции внутри) и цепочка вычислений продолжается (реализации, которого можно встретить например в [js](https://www.npmjs.com/package/try-monad) и в [scala](https://www.scala-lang.org/api/2.12.4/scala/util/Try.html))
- `Result`, который менее универсален и более специфичен, чем `Either`, в виду того, что может являться либо `Ok` с результатом операции внутри, либо содержать ошибку можно встретить в [kotlin](https://adambennett.dev/2020/05/the-result-monad/), [c#](https://csharp-functional.readthedocs.io/en/latest/result-monad.html) или [rust](https://doc.rust-lang.org/std/result/) и конечно же в [haskell](https://hackage.haskell.org/package/either-result-0.3.1.0/docs/Control-Monad-Result.html)

Давайте немного подытожим и сгруппируем наиболее часто встречающиеся типы-обёртки:
- `Option` или `Maybe` - реализуют внутри себя проверку на `null`
- `Either`, `Try`, `Result` - реализуют в той или иной мере работу с ошибками
- `List`, `Array` (`Seq` - от sequence - в общем смысле) - реализуют работу с последовательностями
- `Promise`, `Future`, `Task` - реализуют внутри себя работу с асинхронными операциями
- `IO` - реализует внутри себя работу с "хождением во внешний мир" - ввод/вывод.

Типы-обёртки хорошо себя показывают в яп со строгой статической типизацией (`Haskell`, `Scala`, `F#`), где строгость типов даёт дополнительные страховки от ошибок при написании программ, однако это не значит, что они не могут быть поменяны к языкам с динамической типизацией, в чём мы убедились ранее.   

### Слова на буквы "М", "Ф", "А"

Если вы, прочитав [определение](https://ru.wikipedia.org/wiki/%D0%9C%D0%BE%D0%BD%D0%B0%D0%B4%D0%B0_(%D1%82%D0%B5%D0%BE%D1%80%D0%B8%D1%8F_%D0%BA%D0%B0%D1%82%D0%B5%D0%B3%D0%BE%D1%80%D0%B8%D0%B9)) "М"-слова или [даже статью касающуюся программирования](https://ru.wikipedia.org/wiki/%D0%9C%D0%BE%D0%BD%D0%B0%D0%B4%D0%B0_(%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)), как и автор, подумали, что это "что-то на умном", то не отчаивайтесь. 

Автор, постарается как можно проще изложить суть, вынося за скобки некоторые нюансы - важные с академической части, но не очень простые для понимания неподготовленному читателю.

#### "Ф" - функтор
`Функтором` является "обёртка" над данными, для которой определена функция или метод `map`, принимающая в качестве аргумента другую функцию, применяемую к данным. Обратите внимание - любая функция, преобразующая что-то одно в что-то другое сводится к `map`, т.е. `map` - это обобщённая функция преобразователь. Вспомните метод `map` у класса  `Array` js, и у тех классов, которые мы написали в рамках серии статей: `List`, `Option`, `Either`

```js
new Array(1,2,3).map(x => x + 1).map(x=> x * 10) // [20,30,40]
new List([1,2,3]).map(x => x + 1).map(x=> x * 10) // List(20,30,40)
Option.of(2).map(x => x + x).map(x=> x * x) // Some(16)
new Right(15).map(x => x / 5).map(x => x + 2) // Right(15)
```

#### "M" - монада

`Монадой` является "обёртка" над данными, для которой определена функция или метод `flatMap` (или ещё она может называться `bind`). `flatMap` принимает в качестве аргумента другую функцию, применяемую к данным, а результатом это функции должен быть другой экземпляр `монады`. 

```js
new Array(1,2,3).flatMap(x => new Array(x, x + 1)) // [1, 2, 2, 3, 3, 4]
new List([1,2,3]).flatMap(x => [x, x + 1]) // List(1, 2, 2, 3, 3, 4)
Option.of(2).flatMap(x => new Some(x * x)) // Some(4)
new Right(15).flatMap(x => new Left(42)).map(x => x + 2) // Left(42)
```

#### Другая "М" - моноид

[`Моноидом`](https://ru.wikipedia.org/wiki/%D0%9C%D0%BE%D0%BD%D0%BE%D0%B8%D0%B4) является всё, что умеет комбинироваться с себе подобным ассоциативны и для чего определено **некое "нулевое" или "нейтральное" значение**.
Например: для нашего класса `List` "нулевым" или "пустым" значением является пустой список `[]`, и если бы наш класс `List` мог комбинироваться с другим `List` используя некоторую функцию `combine` или `merge`, то это сделало бы его `моноидом`:

```js
class List {

	//... прочие методы
	
	empty = []

	combine(another_list) {
		return new List([...this, ...another_list])
	}
}
```


Для `Option`

```js
class Option {

	//... прочие методы
	
	empty = None

	combine(another_option) {
		if (this.isEmpty()) {return another_option}
		if (another_option.isEmpty()) {return this}

		return Some(this.getValue() + another_option.getValue()) // !!! 
	}
}
```

> `!!!` операция `+` здесь является частностью. Если углубляться в детали, то у нашего `Option` ДОЛЖНА быть определена операция, что мы делаем в двумя значениями внутри двух "обёрток" - хотим ли мы сложить или сделать из двух значений список из двух элементов. Когда такая операция (назовём её `append`) будет определена - наш тип станет [`Semigroup`](https://en.wikipedia.org/wiki/Semigroup). 

#### "A" - аппликатив

[`Аппликатив`](https://en.wikipedia.org/wiki/Applicative_functor) чуть более сложная для понимания штука, мы не выделяли её явно, поэтому стоит остановиться на ней чуть поподробнее. По сути своей `Аппликатив` это [функтор](ф-функтор) , к которому можно применить другой такой же функтор. Т.е. внутри обёртки содержится не значение, а функция которая может работать со значением другой такой же обёртки. 

Вернёмся к нашей задаче со спутниками, где мы использовали `Either`:

```js
return satelliteService.find("satellite1")
		.flatMap(coord1 => 
				satelliteService.find("satellite2")
				.map(coord2 => {coord1, coord2}))
```

Напомним условие - в случае ошибки метод мы решили, что `find` будет возвращать `Left`, в котором у нас будет объект `Error` с полями:
- `code` - код ошибки;
- `details` - текстовое сообщение об ошибке;

При успешном сценарии мы дожидаемся прихода первого ответа и только после этого запускаем второй. Представим на минуточку, что мы смогли сделать `Either` асинхронным. И теперь мы можем не дожидаться ответа, а сразу сделать оба запроса одновременно, а потом совместить их результаты при помощи функции. Как мы можем это сделать?

Для начала напишем функцию совмещения результата:
```js
function join_coords(coord1, coord2) {
	return {coord1, coord2}
}
```

Хорошо, но мы не сможем приметить эту функцию к результату методу `find`, потому что в каждом из результатов содержится только одна часть аргументов.

Здесь нам поможет каррирование:
```js
const join_coords = (coord1) => (coord2) => {coord1, coord2}
```

Попробуем собрать всё вместе:

```js
return satelliteService.find("satellite1")        // Right(coord1)
		.map(join_coords)                         // Right(join_coord(coord1))
		.map(satelliteService.find("satellite2")) // ERROR!!!
```

Произошла ошибка, потому что внутри `Right` находится функция, а в последний `map` приходит значение, т.е. всё наоборот:

Вместо того чтобы положить в "обёртку" значение и протаскивать его через череду функций - мы положили внутрь обёртки функцию (или группу функций) и вкидываем в качестве аргумента значения, которые за счёт каррирования применяются по одному.

Давайте добавим в `Either` метод, который будет применять функцию внутри к значению пришедшему в качестве аргумента. Метод будет называться  `ap`: 

```js

class Either {

	// ... конструктор и остальные методы
	
	ap(other) {
		if (this.isLeft()) {return this}
		if (other.isLeft()) {return other} 
		return other.map(this.getValue())
	}
}
```

проверим:

```js
new Right(x => x + 1).ap(new Right(5)).getValue() // 6
new Right(x => y => x + y).ap(new Right(5)).ap(new Right(6)).getValue() // 11
```

А теперь применим к нашей задаче:

```js
return new Right(coord1 => coord2 => {coord1, coord2})
		.ap(satelliteService.find("satellite1"))
		.ap(satelliteService.find("satellite2"))
```

И это всё: теперь два метода выполняются независимо.

Подытожим: `Аппликативом` является `функтор`, к которому можно применить другой `функтор`. Напомним, что `функтором` будет всё у чего есть метод `map` - который будет использоваться для применения - `other.map(this.getValue())`

> Автор не припоминает, чтобы когда-то он явно встречал или использовал аппликативы за время своей работы.  




## That' s all folks! 

Есть ещё много вещей из мира ФП, не освящённых в рамках этих статей: мемоизация, [ссылочная прозрачность](https://ru.wikipedia.org/wiki/%D0%A1%D1%81%D1%8B%D0%BB%D0%BE%D1%87%D0%BD%D0%B0%D1%8F_%D0%BF%D1%80%D0%BE%D0%B7%D1%80%D0%B0%D1%87%D0%BD%D0%BE%D1%81%D1%82%D1%8C) и почему иммутабельность [не такая уж и "дорогая"](https://blog.klipse.tech/javascript/2021/02/26/structural-sharing-in-javascript.html), при клонировании или модификации структур.

## Полезные ссылочки

#### Книжечки
[Mostly adequate guide to FP (in javascript)](https://mostly-adequate.gitbook.io/mostly-adequate-guide/) 

[Structure and Interpretation of Computer Programs: JavaScript Edition (MIT Electrical Engineering and Computer Science)](https://www.amazon.com/Structure-Interpretation-Computer-Programs-Engineering-ebook-dp-B094X8316F/dp/B094X8316F) - книжечка, которая, как говорят у нас деревнях - маст хэв.

#### Библиотеки
[Ramda](https://ramdajs.com/) [Lodash](https://lodash.com/) - js библиотеки содержащие много полезных для работы функций (безусловно лучше тех, которые написал автор)

#### Видео
[Functional Design Patterns - Scott Wlaschin](https://youtu.be/srQt1NAHYC0)
[The Functional Programmer's Toolkit - Scott Wlaschin](https://youtu.be/Nrp_LZ-XGsY)
[Thirteen ways of looking at a Turtle - Scott Wlaschin](https://youtu.be/g06igkxbF78)


### Запоздалый дисклеймер

Серией этих статей, автор:
- не ставил целью блеснуть своими знаниями
- не пытался переманить читателя "в свою секту"
- не пытался разводить споры на тему "FP vs OOP", "Any-programming-language-name VS Your-favorite-programming-language-name OR Any-other-programming-language-name"
- не принуждал использовать решения и примеры рассмотренные в рамках цикла статей, непосредственно на работе для решения рабочих задач
- не утверждает, что его решения и подходы абсолютно верные, точные и правильные
- не утверждает, что знает всё обо всём

И ещё автор:
- Хотел поблагодарить, читателя, что он проявил интерес к данному циклу статей
- Надеется, что читатель, найдёт здесь, что-то новое, полезное, или интересное
- Будет рад вопросам, замечаниям, предложениям и словам благодарности
- Надеется, что читатель моет с мылом руки перед приёмом пищи, чистит зубы утром и вечером, питается преимущественно здоровой и полезной пищей, а также проявляет физическую активность хотя бы иногда, а лучше регулярно.

[◀◀](./fp-ii.html)