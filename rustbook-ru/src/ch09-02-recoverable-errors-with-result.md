## Исправимые ошибки с `Result`

Многие ошибки являются не настолько существенными, чтобы останавливать выполнение программы. Иногда, когда в функции происходит сбой, необходимо просто правильное преобразование и обработка ошибки. К примеру, при попытке открыть файл может произойти ошибка из-за отсутствия файла. Вы, возможно, захотите исправить ошибку и создать новый файл вместо остановки программы.

Вспомните раздел ["Обработка возможного сбоя с помощью `Result`"][Обработка ошибок]<!-- ignore --> Главы 2: мы использовали там перечисление `Result`, имеющее два исхода. `Ok` и `Err` для обработки сбоев. Само перечисление определено следующим образом:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Виды `T` и `E` являются свойствами обобщённого вида данных: мы обсудим обобщённые виды данных более подробно в Главе 10. Все что вам нужно знать прямо сейчас - это то, что `T` представляет вид значения, которое будет возвращено в случае успеха внутри исхода `Ok`, а `E` представляет вид ошибки, которая будет возвращена при сбое внутри исхода `Err`. Так как вид данных `Result` имеет эти обобщённые свойства (generic type parameters), мы можем использовать вид данных `Result` и функции, которые определены для него, в разных случаях, когда вид успешного значение и значения ошибки, которые мы хотим вернуть, отличаются.

Давайте вызовем функцию, которая возвращает значение `Result`, потому что может потерпеть неудачу. В приложении 9-3 мы пытаемся открыть файл.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

<span class="caption">Приложение 9-3: Открытие файла</span>

`File::open` возвращает значения вида данных `Result<T, E>`. Гибкий вид данных `T` в выполнении `File::open` соответствует виду успешно полученного значения, `std::fs::File`, а именно указателю файла. Вид данных `E`, используемый для значения в случае возникновения ошибки, - `std::io::Error`. Такой возвращаемый вид данных означает, что вызов `File::open` может быть успешным и вернуть указатель файла, из которого мы можем читать или в который можем вносить данные. Также вызов функции может завершиться неудачей: например, файл может не существовать, или у нас может не быть разрешения на доступ к файлу. Функция `File::open` должна иметь способ сообщить нам об успехе или неудаче и в то же время дать нам либо указатель файла, либо сведения об ошибке. Эту возможность как раз и предоставляет перечисление `Result`.

В случае успеха `File::open` значением переменной `greeting_file_result` будет образец данных `Ok`, содержащий указатель файла. В случае неудачи значение в переменной `greeting_file_result` будет образцом `Err`, содержащим дополнительные сведения о том, какая именно ошибка произошла.

Необходимо дописать в рукопись приложения 9-3 выполнение разных действий в зависимости от значения, которое вернёт вызов `File::open`. Приложение 9-4 показывает один из способов обработки `Result` - пользуясь основным средством языка, таким как выражение `match`, рассмотренным в Главе 6.

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

<span class="caption">Приложение 9-4: Использование выражения <code>match</code> для обработки возвращаемых исходов вида <code>Result</code></span>

Обратите внимание, что также как перечисление `Option`, перечисление `Result` и его исходы, входят в область видимости благодаря само-подключению (prelude), поэтому не нужно указывать `Result::` перед использованием исходов `Ok` и `Err` в ветках выражения `match`.

Если итогом будет `Ok`, эта рукопись вернёт значение `file` из исхода `Ok`, а мы затем присвоим это значение файлового указателя переменной `greeting_file`. После `match` мы можем использовать указатель файла для чтения или записи.

Другая ветвь `match` обрабатывает случай, где мы получаем значение `Err` после вызова `File::open`. В этом примере мы решили вызвать макрос `panic!`. Если в нашей текущей папки нет файла с именем *hello.txt* и мы выполним эту рукопись, то мы увидим следующее сообщение от макроса `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Как обычно, данное сообщение точно говорит, что пошло не так.

### Обработка различных ошибок с помощью match

Рукопись в приложении 9-4 будет вызывать `panic!` независимо от того, почему вызов `File::open` не удался. Однако мы хотим предпринять различные действия для разных причин сбоя. Если открытие `File::open` не удалось из-за отсутствия файла, мы хотим создать файл и вернуть его указатель. Если вызов `File::open` не удался по любой другой причине - например, потому что у нас не было прав на открытие файла, то все равно мы хотим вызвать `panic!` как у нас сделано в приложении 9-4. Для этого мы добавляем выражение внутреннего `match`, показанное в приложении 9-5.

<span class="filename">Файл: src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

<span class="caption">Приложение 9-5: Обработка различных ошибок разными способами</span>

Видом значения возвращаемого функцией `File::open` внутри `Err` исхода является `io::Error`, видом данных из встроенной библиотеки. Данный вид данных имеет способ `kind`, который можно вызвать для получения значения `io::ErrorKind`. Перечисление `io::ErrorKind` из встроенной библиотеки имеет исходы, представляющие различные виды ошибок, которые могут появиться при выполнении действий в `io`. Исход, который мы хотим использовать, это `ErrorKind::NotFound`, который предоставляет сведения, о том, что файл, который мы пытаемся открыть, ещё не существует. Итак, во второй строке мы вызываем сопоставление образца данных с переменной `greeting_file_result` и попадаем в ветку с обработкой ошибки, но также у нас есть внутренняя проверка для сопоставления `error.kind()` ошибки.

Условие, которое мы хотим проверить во внутреннем `match`, заключается в том, является ли значение, возвращаемое `error.kind()`, исходом `NotFound` перечисления `ErrorKind`. Если это так, мы пытаемся создать файл с помощью функции `File::create`. Однако, поскольку вызов `File::create` тоже может завершиться ошибкой, нам нужна обработка ещё одной ошибки, теперь уже во внутреннем выражении `match`. Заметьте: если файл не может быть создан, выводится другое, особое сообщение об ошибке. Вторая же ветка внешнего `match` (который обрабатывает вызов `error.kind()`), остаётся той же самой - в итоге программа вызывает сбой при любой ошибке, кроме ошибки отсутствия файла.

> ### Иные использованию `match` с `Result<T, E>`
>
> Как много `match`! Выражение `match` является очень полезным, но в то же время довольно простым. В главе 13 вы узнаете о замыканиях (closures), которые используются во многих способах вида данных `Result<T, E>`. Эти способы помогают быть более кратким, чем использование `match` при работе со значениями `Result<T, E>` в вашей рукописи.
>
> Например, вот другой способ написать тот же ход мыслей, что показан в Приложении 9-5, но с использованием замыканий и способа `unwrap_or_else`:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {:?}", error);
>             })
>         } else {
>             panic!("Problem opening the file: {:?}", error);
>         }
>     });
> }
> ```
>
> Несмотря на то, что данная рукопись имеет такое же поведение как в приложении 9-5, она не содержит ни одного выражения `match` и проще для чтения. Советуем вам вернуться к примеру этого раздела после того как вы прочитаете Главу 13 и изучите способ `unwrap_or_else` по пособию встроенной библиотеки. Многие из способов, о которых вы узнаете в пособии и Главе 13 могут очистить рукопись от больших, вложенных выражений `match` при обработке ошибок.

### Краткие способы обработки ошибок - `unwrap` и `expect`

Использование средства `match` работает достаточно хорошо, но может быть довольно многословным и не всегда хорошо передаёт смысл. Вид данных `Result<T, E>` имеет множество вспомогательных способов для выполнения различных, более отличительных задач. Способ `unwrap` - это средство быстрого доступа к значениям, выполненный так же, как и выражение `match`, которое мы написали в Приложении 9-4. Если значение `Result` является исходом `Ok`, `unwrap` возвращает значение внутри `Ok`. Если `Result` - исход `Err`, то `unwrap` вызовет для нас макрос `panic!`. Вот пример `unwrap` в действии:

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

Если мы запустим эту рукопись при отсутствии файла *hello.txt*, то увидим сообщение об ошибке из вызова `panic!` способа `unwrap`:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:4:49
```

Другой способ, похожий на `unwrap`, это `expect`, позволяющий указать сообщение об ошибке для макроса `panic!`. Использование `expect` вместо `unwrap` с предоставлением хорошего сообщения об ошибке выражает ваше намерение и делает более простым отслеживание источника сбоя. Правила написания способов `expect` выглядят так:

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

Способ `expect` используется так же как и `unwrap`: для возврата указателя файла либо вызова макроса `panic!`. Сообщение об ошибке в `expect` будет передано в `panic!` и заменит обычное используемое сообщение.Вот как это выглядит:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at 'hello.txt should be included in this project: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:5:10
```

В рабочей рукописи большинство людей выбирают способу `expect` в угоду `unwrap`, также добавляют описание, почему действие должно закончиться успешно. Если возникнет сбой, то у вас будет больше сведений для отладки вашей рукописи.

### Проброс ошибок

Когда вы пишете функцию, выполнение которой вызывает что-то, что может завершиться ошибкой, вместо обработки ошибки в этой функции, вы можете вернуть ошибку в вызывающую рукопись, чтобы она могла решить, что с ней делать. Такой приём известен как *распространение ошибки* (*propagating the error*). Благодаря нему мы даём больше управления вызывающей рукописи, где может быть больше сведений или хода мыслей, которые диктуют, как ошибка должна обрабатываться, чем то, что доступно вам в среде вашей рукописи.

Например, рукопись программы 9-6 считывает имя пользователя из файла. Если файл не существует или не может быть прочтён, то функция возвращает ошибку в рукопись, которая вызвала данную функцию.

<span class="filename">Файл: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

<span class="caption">Приложение 9-6: Функция, которая возвращает ошибки в вызывающую рукопись, используя приказчик <code>match</code></span>

Эта функция может быть написана гораздо более коротким способом, но мы начнём с того, что многое сделаем вручную, чтобы изучить обработку ошибок; а в конце покажем более короткий способ. Давайте сначала рассмотрим вид данных возвращаемого значения: `Result<String, io::Error>`. Здесь есть возвращаемое значение функции вида данных `Result<T, E>`, где образец свойства `T` был заполнен определенным видом данных `String` и образец свойства `E` был заполнен определенным видом `io::Error`.

Если эта функция выполнится без неполадок, то рукопись, вызывающая эту функцию, получит значение `Ok`, содержащее `String` - имя пользователя, которое эта функция прочитала из файла. Если функция столкнётся с какими-либо неполадками, вызывающая рукопись получит значение `Err`, содержащее образец данных `io::Error`, который включает дополнительные сведения о том, какие возникли ошибки. Мы выбрали `io::Error` в качестве возвращаемого вида данных этой функции, потому что это вид значения ошибки, возвращаемого из обоих действий, которые мы вызываем в теле этой функции и которые могут завершиться неудачей: функция `File::open` и способ `read_to_string`.

Тело функции начинается с вызова `File::open`. Затем мы обрабатываем значение `Result` с помощью `match`, подобно `match` из приложения 9-4. Если `File::open` завершается успешно, то указатель файла в переменной образца данных `file` становится значением в изменяемой переменной `username_file` и функция продолжит свою работу. В случае `Err`, вместо вызова `panic!`, мы используем ключевое слово `return` для досрочного возврата из функции и передаём значение ошибки из `File::open`, которое теперь находится в переменной образца данных `e`, обратно в вызывающую рукопись как значение ошибки этой функции.

Таким образом, если у нас есть файловый указатель в `username_file`, функция создаёт новую `String` в переменной `username` и вызывает способ `read_to_string` для файлового указателя в `username_file`, чтобы прочитать содержимое файла в `username`. Способ `read_to_string` также возвращает `Result`, потому что он может потерпеть неудачу, даже если `File::open` завершился успешно. Поэтому нам нужен ещё один `match` для обработки этого `Result`: если `read_to_string` завершится успешно, то наша функция сработала, и мы возвращаем имя пользователя из файла, которое теперь находится в `username`, обёрнутое в `Ok`. Если `read_to_string` потерпит неудачу, мы возвращаем значение ошибки таким же образом, как мы возвращали значение ошибки в `match`, который обрабатывал возвращаемое значение `File::open`. Однако нам не нужно явно указывать `return`, потому что это последнее выражение в функции.

Затем рукопись, вызывающая эту, будет обрабатывать полученное значение `Ok`, содержащего имя пользователя, либо значения `Err`, содержащего `io::Error`. Вызывающая рукопись должна решить, что делать с этими значениями. Если вызывающая рукопись получает значение `Err`, она может вызвать `panic!` и завершить работу программы, использовать имя пользователя по умолчанию или найти имя пользователя, например, не в файле. У нас недостаточно сведений о том, что на самом деле пытается сделать вызывающая рукопись, поэтому мы распространяем все сведения об успехах или ошибках вверх, чтобы она могла обрабатываться соответствующим образом.

Эта схема передачи ошибок настолько распространена в языке Ржавчина, что язык предоставляет приказчик вопросительного знака `?`, чтобы облегчить эту задачу.

#### Сокращение для проброса ошибок: приказчик `?`

В приложении 9-7 показано использование собственного способа `read_username_from_file`, который имеет ту же возможность, что и в приложении 9-6, но в этои исполнении используется приказчик `?`.

<span class="filename">Файл: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

<span class="caption">Приложение 9-7: Функция, возвращающая ошибки в вызывающую рукопись с помощью приказчика <code>?</code></span>

Выражение `?`, расположенное после `Result`, работает почти так же, как и те выражения `match`, которые мы использовали для обработки значений `Result` в приложении 9-6. Если в качестве значения `Result` будет `Ok`, то значение внутри `Ok` будет возвращено из этого выражения, и программа продолжит работу. Если же значение представляет собой `Err`, то `Err` будет возвращено из всей функции, как если бы мы использовали ключевое слово `return`, так что значение ошибки будет передано в вызывающую рукопись.

Существует разница между тем, что делает выражение `match` из приложения 9-6 и тем, что делает приказчик `?`: значения ошибок, для которых вызван приказчик `?`, проходят через функцию `from`, определённую в сущности `From` встроенной библиотеки, которая используется для преобразования значений из одного вида данных в другой. Когда приказчик `?` вызывает функцию `from`, полученный вид ошибки преобразуется в вид ошибки, определённый в возвращаемом виде данных текущей функции. Это полезно, когда функция возвращает только один вид ошибки, для описания всех возможных исходов сбоев, даже если её отдельные составляющие могут выходить из строя по разным причинам.

Например, мы могли бы изменить функцию `read_username_from_file` в приложении 9-7, чтобы возвращать пользовательский вид ошибки с именем `OurError`, который мы определим. Если мы также определим `impl From<io::Error> for OurError` для создания образца данных `OurError` из `io::Error`, то приказчик `?`, вызываемый в теле `read_username_from_file`, вызовет `from` и преобразует виды ошибок без необходимости добавления дополнительного рукописи в функцию.

В случае приложения 9-7 приказчик `?` в конце вызова `File::open` вернёт значение внутри `Ok` в переменную `username_file`. Если произойдёт ошибка, приказчик `?` выполнит ранний возврат значения `Err` вызывающей рукописи. То же самое относится к приказчику `?` в конце вызова `read_to_string`.

Приказчик `?` позволяет избавиться от большого количества образцов и упростить выполнение этой функции. Мы могли бы даже ещё больше сократить эту рукопись, если бы использовали цепочку вызовов способов сразу после `?`, как показано в приложении 9-8.

<span class="filename">Файл: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

<span class="caption">Приложение 9-8: Цепочка вызовов способов после приказчика <code>?</code></span>

Мы перенесли создание новой `String` в `username` в начало функции; эта часть не изменилась. Вместо создания переменной `username_file` мы соединили вызов `read_to_string` непосредственно с итогом `File::open("hello.txt")?`. У нас по-прежнему есть `?` в конце вызова `read_to_string`, и мы по-прежнему возвращаем значение `Ok`, содержащее `username`, когда и `File::open` и `read_to_string` завершаются успешно, а не возвращают ошибки. Возможность снова такая же, как в Приложении 9-6 и Приложении 9-7; это просто другой, более удобный способ её написания.

Продолжая рассматривать разные способы записи данной функции, приложение 9-9 отображает способ сделать её ещё короче с помощью `fs::read_to_string`.

<span class="filename">Файл: src/main.rs</span>

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

<span class="caption">Приложение 9-9: Использование <code>fs::read_to_string</code> вместо открытия и последующего чтения файла</span>

Чтение файла в строку довольно распространённое действие, так что обычная библиотека предоставляет удобную функцию `fs::read_to_string`, которая открывает файл, создаёт новую `String`, считывает содержимое файла, размещает его в `String` и возвращает её. Конечно, использование функции `fs::read_to_string` не даёт возможности объяснить обработку всех ошибок, поэтому мы сначала изучили длинный способ.

#### Где можно использовать приказчик `?`

Приказчик `?` может использоваться только в функциях, вид данных возвращаемого значения которых совместимы со значением, для которого используется `?`. Это потому, что приказчик `?` определён для выполнения раннего возврата значения из функции таким же образом, как и выражение `match`, которое мы определили в приложении 9-6. В приложении 9-6 `match` использовало значение `Result`, а ответвление с ранним возвратом вернуло значение `Err(e)`. Вид данных возвращаемого значения функции должен быть `Result`, чтобы он был совместим с этим `return`.

В приложении 9-10 давайте посмотрим на ошибку, которую мы получим, если воспользуемся приказчиком `?` в функции `main` с видом данных возвращаемого значения, несовместимым с видом данных значения, для которого мы используем `?`:

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

<span class="caption">Приложение 9-10: Попытка использовать <code>?</code> в <code>main</code> функции, которая возвращает <code>()</code> , не будет собираться</span>

Эта рукопись открывает файл, что может привести к сбою. `?` приказчик следует за значением `Result` , возвращаемым `File::open` , но эта `main` функция имеет возвращаемый вид данных `()` , а не `Result` . Когда мы собираем эту рукопись, мы получаем следующее сообщение об ошибке:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Эта ошибка указывает на то, что приказчик `?` разрешено использовать только в функции, которая возвращает `Result`, `Option` или другой вид данных, использующий `FromResidual`.

Для исправления ошибки есть два исхода. Первый - изменить возвращаемый вид данных вашей функции так, чтобы он был совместим со значением, для которого вы используете приказчик `?`, если у вас нет ограничений, препятствующих этому. Другой способ - использовать `match` или один из способов `Result<T, E>` для обработки `Result<T, E>` любым подходящим способом.

В сообщении об ошибке также упоминалось, что `?` можно использовать и со значениями `Option<T>`. Как и при использовании `?` для `Result`, вы можете использовать `?` только для `Option` в функции, которая возвращает `Option`. Поведение приказчика `?` при вызове `Option<T>` похоже на его поведение при вызове `Result<T, E>`: если значение равно `None`, то `None` будет возвращено раньше из функции в это мгновение. Если значение `Some`, значение внутри `Some` является итоговым значением выражения, и функция продолжает исполняться. В приложении 9-11 приведён пример функции, которая находит последний знак первой строки заданного писания:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

<span class="caption">Приложение 9-11: Использование приказчика <code>?</code> для значения <code>Option&lt;T&gt;</code></span>

Эта функция возвращает `Option<char>`, потому что возможно, что там есть знак, но также возможно, что его нет. Эта рукопись принимает переменная среза `text` строки и вызывает для него способ `lines`, который возвращает повторитель для строк в строке. Поскольку эта функция хочет проверить первую строку, она вызывает `next` у повторителя, чтобы получить первое значение от повторителя. Если `text` является пустой строкой, этот вызов `next` вернёт `None`, и в этом случае мы используем `?` чтобы остановить и вернуть `None` из `last_char_of_first_line`. Если `text` не является пустой строкой, `next` вернёт значение `Some`, содержащее отрывок строки первой строки в `text`.

Знак `?` извлекает отрывок строки, и мы можем вызвать `chars` для этого отрывка строки. чтобы получить повторитель знаков. Нас важно последний знак в первой строке, поэтому мы вызываем `last`, чтобы вернуть последнюю переменную в повторителе. Вернётся `Option`, потому что возможно, что первая строка пустая - например, если `text` начинается с пустой строки, но имеет знаки в других строках, как в `"\nhi"`. Однако, если в первой строке есть последний знак, он будет возвращён в исходе `Some`. Приказчик `?` в середине даёт нам краткий способ выразить этот ход мыслей, позволяя выполнить функцию в одной строке. Если бы мы не могли использовать приказчик `?` в `Option`, нам пришлось бы выполнить этот ход мыслей, используя больше вызовов способов или выражение `match`.

Обратите внимание, что вы можете использовать приказчик `?` `Result` в функции, которая возвращает `Result` , и вы можете использовать приказчик `?` для `Option` в функции, которая возвращает `Option` , но вы не можете смешивать и сопоставлять. Приказчик `?` не будет самостоятельно преобразовывать `Result` в `Option` или наоборот; в этих случаях вы можете использовать такие способы, как способ `ok` для `Result` или способ `ok_or` для `Option`, чтобы выполнить преобразование явно.

До сих пор все функции `main`, которые мы использовали, возвращали `()`. Функция `main` - особенная, потому что это точка входа и выхода исполняемых программ, и существуют ограничения на вид данных возвращаемого значения, чтобы программы вели себя так, как ожидается.

К счастью, `main` также может возвращать `Result<(), E>` . В приложении 9-12 используется рукопись из приложения 9-10, но мы изменили возвращаемый вид данных `main` на `Result<(), Box<dyn Error>>` и добавили возвращаемое значение `Ok(())` в конец. Теперь эта рукопись будет собрана:

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

<span class="caption">Приложение 9-12: Замена <code>main</code> на return <code>Result&lt;(), E&gt;</code> позволяет использовать приказчик <code>?</code> приказчик над значениями <code>Result</code></span>

Вид данных `Box<dyn Error>` является *сущность-предметом*, о котором мы поговорим в разделе ["Использование сущность-предметов, допускающих значения разных видов данных"][Сущность-предмет]<!-- ignore --> в главе 17. Пока что вы можете считать, что `Box<dyn Error>` означает "любой вид ошибки". Использование `?` для значения `Result` в функции `main` с видом ошибки `Box<dyn Error>` разрешено, так как позволяет вернуть любое значение `Err` раньше времени. Даже если тело этой функции `main` будет возвращать только ошибки вида данных `std::io::Error`, указав `Box<dyn Error>`, эта ярлык останется правильным, даже если в тело `main` будет добавлена рукопись, возвращающая другие ошибки.

Когда `main` функция возвращает `Result<(), E>`, исполняемый файл завершится со значением `0`, если `main` вернёт `Ok(())`, и выйдет с ненулевым значением, если `main` вернёт значение `Err`. Исполняемые файлы, написанные на C, при выходе возвращают целые числа: успешно завершённые программы возвращают целое число `0`, а программы с ошибкой возвращают целое число, отличное от `0`. Ржавчина также возвращает целые числа из исполняемых файлов, чтобы быть совместимым с этим соглашением.

Функция `main` может возвращать любые виды данных, использующие [сущность `std::process::Termination`][Подавление]<!-- ignore -->, в которых имеется функция `report`, возвращающая `ExitCode`. Обратитесь к пособию встроенной библиотеки за дополнительными сведениями о порядке использования сущности `Termination` для ваших собственных видов данных.

Теперь, когда мы обсудили подробности вызова `panic!` или возврата `Result`, давайте вернёмся к тому, как решить, какой из случаев подходит для нашего примера.


[Сущность-предмет]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[Подавление]: ../std/process/trait.Termination.html
[Обработка ошибок]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result