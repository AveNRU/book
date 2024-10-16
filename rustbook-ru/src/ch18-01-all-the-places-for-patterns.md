## Все случаи, где могут быть использованы образцы

В этапе использования языка Ржавчина вы часто используете образцы, даже не осознавая этого! В этом разделе обсуждаются все случаи, где использование образцов является правильным.

### Ветки `match`

Как обсуждалось в главе 6, мы используем образцы в ветках выражений `match`. Условное выражения `match` определяется как ключевое слово `match`, значение используемое для сопоставления, одна или несколько веток, которые состоят из образца данных и выражения для выполнения, если значение соответствует образцу данных этой ветки, как здесь:

```text
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

Например, вот выражение `match` из приложения 6-5, которое соответствует значению `Option<i32>` в переменной `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Образцами данных в этом выражении `match` являются `None` и `Some(i)` слева от каждой стрелки.

Одно из требований к выражениям `match` состоит в том, что они должны быть *исчерпывающими* (exhaustive) в том смысле, что они должны учитывать все возможности для значения в выражении `match`. Один из способов убедиться, что вы рассмотрели каждую возможность - это иметь образец перехвата всех исходов в последней ветке выражения: например, имя переменной, совпадающее с любым значением, никогда не может потерпеть неудачу и таким образом, охватывает каждый оставшийся случай.

Особый образец данных `_` будет соответствовать чему угодно, но он никогда не привязывается к переменной, поэтому он часто используется в последней ветке. Образец данных `_` может быть полезен, если вы, например, хотите пренебрегать любое не указанное значение. Мы рассмотрим образец данных `_` более подробно в разделе ["Пренебрежение значений в образце](ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern)<!--  --> позже в этой главе.

### Условные выражения `if let`

В главе 6 мы обсуждали, как использовать выражения `if let` как правило в качестве более короткого способа записи равнозначного `match`, которое обрабатывает только один случай. Дополнительно `if let` может иметь соответствующий `else`, содержащий рукопись для выполнения, если образец выражения `if let` не совпадает.

В приложении 18-1 показано, что можно также смешивать и сопоставлять выражения `if let`, `else if` и `else if let`. Это даёт больше гибкости, чем `match` выражение, в котором можно выразить только одно значение для сравнения с образцами данных. Кроме того, условия в последовательности `if let`, `else if`, `else if let` не обязаны соотноситься друг с другом.

Рукопись в приложении 18-1 показывает последовательность проверок нескольких условий, определяющих каким должен быть цвет фона. В данном примере мы создали переменные с предопределёнными значениями, которые в существующей программе могли бы быть получены из пользовательского ввода.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-01/src/main.rs}}
```

<span class="caption">Приложение 18-1: Использование условных стопок <code>if let</code>, <code>else if</code>, <code>else if let</code>, и <code>else</code></span>

Если пользователь указывает любимый цвет, то этот цвет используется в качестве цвета фона. Если любимый цвет не указан, и сегодня вторник, то цвет фона - зелёный. Иначе, если пользователь указывает свой возраст в виде строки, и мы можем успешно просмотреть её и представить в виде числа, то цвет будет либо фиолетовым, либо оранжевым, в зависимости от значения числа. Если ни одно из этих условий не выполняется, то цвет фона будет синим.

Это условное устройство позволяет поддерживать сложные требования. С жёстко заданными значениями, которые у нас здесь есть, этот пример выведет `Using purple as the background color`.

Можно увидеть, что `if let` может также вводить затенённые переменные, как это можно сделать в `match` ветках: строка `if let Ok(age) = age` вводит новую затенённую переменную `age`, которая содержит значение внутри исхода `Ok`. Это означает, что нам нужно поместить условие `if age > 30` внутри этого раздел: мы не можем объединить эти два условия в `if let Ok(age) = age && age > 30`. Затенённый `age`, который мы хотим сравнить с 30, не является действительным, пока не начнётся новая область видимости с узорчатой скобки.

Недостатком использования `if let` выражений является то, что сборщик не проверяет полноту (exhaustiveness) всех исходов, в то время как с помощью выражения `match` это происходит. Если мы не напишем последний раздел`else` и, благодаря этому, пропустим обработку некоторых случаев, сборщик не предупредит нас о возможной разумной ошибке.

### Условные круговороты `while let`

Подобно устройству `if let`, устройство условного круговорота `while let` позволяет повторять круговорот `while` до тех пор, пока образец данных продолжает совпадать. Пример в приложении 18-2 отображает круговорот `while let`, который использует вектор в качестве обоймы и выводит значения вектора в порядке, обратном тому, в котором они были помещены.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-02/src/main.rs:here}}
```

<span class="caption">Приложение 18-2: Использование круговорота <code>while let</code> для выводе значений до тех пор, пока <code>stack.pop()</code> возвращает <code>Some</code></span>

В этом примере выводится 3, 2, а затем 1. Способ `pop` извлекает последнюю переменную из вектора и возвращает `Some(value)`. Если вектор пуст, то `pop` возвращает `None`. Круговорот `while` продолжает выполнение рукописи в своём разделе, пока `pop` возвращает `Some`. Когда `pop` возвращает `None`, круговорот останавливается. Мы можем использовать `while let` для удаления каждой переменной из обоймы.

### Круговорот `for`

В круговороте `for` значение, которое следует непосредственно за ключевым словом `for` , является образцом. Например, в `for x in y`  выражение `x` является образцом. В приложении 18-3 показано, как использовать образец данных в круговороте `for` , чтобы разъединять или разбить упорядоченный ряд как часть круговорота `for` .

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-03/src/main.rs:here}}
```

<span class="caption">Приложение 18-3: Использование образца данных в круговороте <code>for</code> для разъединения упорядоченного ряда</span>

Рукопись в приложении 18-3 выведет следующее:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-03/output.txt}}
```

Мы приспособим повторитель с помощью способа `enumerate`, чтобы он порождал упорядоченный ряд, состоящий из значения и порядкового указателя этого значения. Первым созданным значением будет упорядоченный ряд `(0, 'a')`. Когда это значение сопоставляется с образцом `(index, value)`, `index` будет равен `0`, а `value` будет равно `'a'` и будет выведена первая строка выходных данных.

### Указание `let`

До этой Главы мы подробно обсуждали только использование образцов с `match` и `if let`, но на самом деле, мы использовали образцы и в других местах, в том числе в указаниях `let`. Например, рассмотрим следующее простое назначение переменной с помощью `let`:

```rust
let x = 5;
```

Каждый раз, когда вы использовали подобным образом указанию `let`, вы использовали образцы, хотя могли и не осознавать этого! Более условно указание `let` выглядит так:

```text
let PATTERN = EXPRESSION;
```

В указаниях вида данных `let x = 5;` с именем переменной в слоте `PATTERN`, имя переменной является просто отдельной, простой способом образца. Ржавчина сравнивает выражение с образцом и присваивает любые имена, которые он находит. Так что в примере `let x = 5;`, `x` - это образец данных, который означает "привязать то, что соответствует здесь, переменной `x`". Поскольку имя `x` является полностью образцом, этот образец данных в действительности означает "привязать все к переменной `x` независимо от значения".

Чтобы более чётко увидеть особенность сопоставления с образцом `let`, рассмотрим приложение 18-4, в котором используется образец данных с `let` для разъединения упорядоченного ряда.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-04/src/main.rs:here}}
```

<span class="caption">Приложение 18-4. Использование образца данных для разъединения упорядоченного ряда и создания трёх переменных одновременно</span>

Здесь мы сопоставляем упорядоченный ряд с образцом. Ржавчина сравнивает значение `(1, 2, 3)` с образцом `(x, y, z)` и видит, что значение соответствует образцу данных, поэтому Ржавчина связывает `1` с `x`, `2` с `y` и `3` с `z`. Вы можете думать об этом образце упорядоченного ряда как о вложении в него трёх отдельных образцов переменных.

Если количество переменных в образце не совпадает с количеством переменных в упорядоченном ряде, то весь вид данных не будет совпадать и мы получим ошибку сборщика. Например, в приложении 18-5 показана попытка разъединять упорядоченный ряд с тремя переменными в две переменные, что не будет работать.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-05/src/main.rs:here}}
```

<span class="caption">Приложение 18-5: Неправильное построение образца, переменные не соответствуют количеству переменных в упорядоченном ряде</span>

Попытка собрать эту рукопись приводит к ошибке:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-05/output.txt}}
```

Чтобы исправить ошибку, мы могли бы пренебрегать одно или несколько значений в упорядоченном ряде, используя `_` или `..`, как вы увидите в разделе [“Пренебрежение значений в Образце”](ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern) <!-- ignore -->. Если образец данных содержит слишком много переменных в образце, можно решить неполадку, сделав виды данных совпадающими, удалив некоторые переменные таким образом, чтобы число переменных равнялось числу переменных в упорядоченном ряде.

### Свойства функции

Свойства функции также могут быть образцами данных. Рукопись в приложении 18-6 объявляет функцию с именем `foo`, которая принимает одно свойство с именем `x` вида данных `i32`, к настоящему времени это должно выглядеть знакомым.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-06/src/main.rs:here}}
```

<span class="caption">Приложение 18-6: Ярлык функции использует образцы в свойствах</span>

`x` это часть образца! Как и в случае с `let`, мы можем сопоставить упорядоченный ряд в переменных функции с образцом. Приложение 18-7 разделяет значения в упорядоченном ряде при его передачи в функцию.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-07/src/main.rs}}
```

<span class="caption">Приложение 18-7: Функция с свойствами, которая разрушает упорядоченный ряд</span>

Эта рукопись выводит `Current location: (3, 5)`. Значения `&(3, 5)` соответствуют образцу данных `&(x, y)`, поэтому `x` - это значение `3`, а `y` - это значение `5`.

Добавляя к вышесказанному, мы можем использовать образцы в списках свойств замыкания таким же образом, как и в списках свойств функции, потому что, как обсуждалось в главе 13, замыкания похожи на функции.

На данное мгновение вы видели несколько способов использования образцов, но образцы работают не одинаково во всех местах, где их можно использовать. В некоторых местах образцы должны быть неопровержимыми; в других обстоятельствах они могут быть опровергнуты. Мы обсудим эти два подхода далее.
