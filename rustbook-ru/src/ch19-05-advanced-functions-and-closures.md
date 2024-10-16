## Продвинутые функции и замыкания

В этом разделе рассматриваются некоторые продвинутые возможности, относящиеся к функциям и замыканиям, такие как указатели функций и возвращаемые замыкания.

### Указатели функций

Мы уже обсуждали, как передавать замыкания в функции; но также можно передавать обычные функции в функции! Эта техника полезна, когда вы хотите передать ранее созданную функцию, а не определять новое замыкание. Функции соответствуют виду данных `fn` (со строчной буквой f), не путать с сущностью замыкания `Fn`. Вид данных `fn` называется *указателем функции*. Передача функций с помощью указателей функций позволяет использовать функции в качестве переменных других функций.

Для указания того, что свойство является указателем на функцию, используются правила написания, такой же, как и для замыканий, что отображается в приложении 19-27, где мы определили функцию `add_one`, которая добавляет единицу к переданному ей свойству. Функция `do_twice` принимает два свойства: указатель на любую функцию, принимающую свойство `i32` и возвращающую `i32`, и число вида данных `i32`. Функция `do_twice` дважды вызывает функцию `f`, передавая ей значение `arg`, а затем складывает полученные итоги. Функция `main` вызывает функцию `do_twice` с переменными `add_one` и `5`.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-27/src/main.rs}}
```

<span class="caption">Приложение 19-27: Использование вида данных <code>fn</code> для получения указателя на функцию в качестве переменной</span>

Эта рукопись выводит `Ответ: 12`. Мы указали, что свойство `f` в `do_twice` является `fn`, которое принимает на вход единственное свойство вида данных `i32` и возвращает `i32`. Затем мы можем вызвать `f` в теле `do_twice`. В `main` мы можем передать имя функции `add_one` в качестве первой переменной в `do_twice`.

В отличие от замыканий, `fn` является видом данных, а не сущностью, поэтому мы указываем `fn` непосредственно в качестве вида данных свойства, а не объявляем свойство гибкого вида данных с одним из сущностей `Fn` в качестве связанного.

Указатели функций используют все три сущности замыканий (`Fn`, `FnMut` и `FnOnce`), то есть вы всегда можете передать указатель функции в качестве переменной функции, которая ожидает замыкание. Лучше всего для описания функции использовать гибкий вид данных и один из сущностей замыканий, чтобы ваши функции могли принимать как функции, так и замыкания.

Однако, одним из примеров, когда вы бы хотели принимать только `fn`, но не замыкания, является взаимодействие с внешним рукописью, который не имеет замыканий: функции языка C могут принимать функции в качестве переменных, однако замыканий в языке C нет.

В качестве примера того, где можно использовать либо замыкание, определяемое непосредственно в месте передачи, либо именованную функцию, рассмотрим использование способа `map`, предоставляемого сущностью `Iterator` во встроенной библиотеке. Чтобы использовать функцию `map` для преобразования вектора чисел в вектор строк, мы можем использовать замыкание, например, так:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-15-map-closure/src/main.rs:here}}
```

Или мы можем использовать функцию в качестве переменной `map` вместо замыкания, например, так:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-16-map-function/src/main.rs:here}}
```

Обратите внимание, что мы должны использовать полные правила написания, о котором мы говорили ранее в разделе ["Продвинутые сущности"](ch19-03-advanced-traits.html#advanced-traits)<!--  -->, потому что доступно несколько функций с именем `to_string`. Здесь мы используем функцию `to_string` определённую в сущности `ToString`, который выполнен во встроенной библиотеке для любого вида данных выполняющего сущность `Display`.

Вспомните из раздела ["Значения перечислений"] Главы 6, что имя каждого определённого нами исхода перечисления также становится функцией-объявителем. Мы можем использовать эти объявители в качестве указателей на функции, использующих сущности замыканий, что означает, что мы можем использовать объявители в качестве переменных для способов, принимающих замыкания, например, так:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-17-map-initializer/src/main.rs:here}}
```

Здесь мы создаём образцы `Status::Value`, используя каждое значение `u32` в ряде (0..20), с которым вызывается `map` с помощью функции объявителя `Status::Value`. Некоторые люди предпочитают это исполнение, а некоторые предпочитают использовать замыкания. Оба исхода собирается в одну и ту же рукопись, поэтому используйте любое исполнение, которое вам понятнее.

### Возврат замыканий

Замыкания представлены сущностями, что означает, что вы не можете возвращать замыкания из функций. В большинстве случаев, когда вам захочется вернуть сущность, вы можете использовать определенный вид данных, выполняющий эту сущность, в качестве возвращаемого значения функции. Однако вы не можете сделать подобного с замыканиями, поскольку у них не может быть определенного вида данных, который можно было бы вернуть; например, вы не можете использовать указатель на функцию `fn` в качестве возвращаемого вида данных.

Следующая рукопись пытается напрямую вернуть замыкание, но она не собирается:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-18-returns-closure/src/lib.rs}}
```

Ошибка сборщика выглядит следующим образом:

```console
{{#include ../listings/ch19-advanced-features/no-listing-18-returns-closure/output.txt}}
```

Ошибка снова ссылается на сущность `Sized` ! Ржавчина не знает, сколько памяти нужно будет выделить для замыкания. Мы видели решение этих сбоев ранее. Мы можем использовать сущность-предмет:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-19-returns-closure-trait-object/src/lib.rs}}
```

Эта рукопись просто отлично собирается. Для получения дополнительных сведений об сущности-предмета обратитесь к разделу ["Использование сущность-предметов, которые допускают значения разных видов данных"](ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types)<!--  --> Главы 17.

Далее давайте посмотрим на макросы!


["Значения перечислений"]: ch06-01-defining-an-enum.html#enum-values