## Управление хода выполнения проверок

Подобно тому, как `cargo run` выполняет сборку вашей рукописи, а затем запускает полученный двоичный файл, `cargo test` собирает вашу рукопись в режиме проверки и запускает полученный двоичный файл с проверками. Двоичный файл, создаваемый `cargo test`, по умолчанию запускает все проверки одновременно и перехватывает вывод, порождаемый во время выполнения проверок, предотвращая их вывод на экран для облегчения чтения вывода, относящегося к итогам проверки. Однако вы можете указать свойства приказной строки, чтобы изменить это поведение по умолчанию.

Часть свойств приказной строки передаётся в `cargo test`, а часть - в итоговый двоичный файл с проверками. Чтобы разделить эти два вида данных переменных, нужно сначала указать переменные, которые идут в `cargo test`, затем использовать разделитель `--`, а потом те, которые попадут в двоичный файл проверки. Использование `cargo test --help` выводит возможности, которые вы можете использовать с `cargo test`, а использование `cargo test -- --help` выводит возможности, которые вы можете использовать за разделителем.

### Выполнение проверок одновременно или последовательно

Когда вы запускаете несколько проверок, по умолчанию они выполняются одновременно с использованием потоков, что означает, что они завершатся быстрее, и вы быстрее получите итоги. Поскольку проверки выполняются одновременно, вы должны убедиться, что ваши проверки не зависят друг от друга или от какого-либо общего состояния, включая общее окружение, например, текущая рабочая папка или переменные окружения.

Например, допустим, каждая из ваших проверок запускает рукопись, которая создаёт файл на диске с именем *test-output.txt* и записывает некоторые данные в этот файл. Затем каждая проверка считывает данные из этого файла и утверждает, что файл содержит определённое значение, которое в каждой проверке разное. Поскольку все проверки выполняются одновременно, одна из проверок может перезаписать файл в промежутке между записью и чтением файла другой проверкой. Тогда вторая проверка потерпит неудачу, но не потому, что рукопись неверна, а потому, что эти проверки мешали друг другу при одновременном выполнении. Одно из решений - убедиться, что каждая проверка пишет в свой отдельный файл; другое решение - запускать проверки по одной.

Если вы не хотите запускать проверки одновременно или хотите более подробный управление над количеством используемых потоков, можно установить клеймо `--test-threads` и то количество потоков, которое вы хотите использовать для проверки. Взгляните на следующий пример:

```console
$ cargo test -- --test-threads=1
```

Мы устанавливаем количество проверочных потоков равным `1` , указывая программе не использовать одновременность. Выполнение проверок с использованием одного потока займёт больше времени, чем их одновременное выполнение, но проверки не будут мешать друг другу, если они совместно используют состояние.

### Отображение итогов работы функции

По умолчанию, если проверка пройдена, система управления запуска проверок запрещает вывод на вывод, т.е. если вы вызовете макрос `println!` внутри рукописи проверки и проверка будет пройден, вы не увидите вывода на окно вывода итогов вызова `println!`. Если же проверка не был пройден, все несущие сведения сообщения, а также описание ошибки будут выведены на окно вывода.

Например, в рукописи (11-10) функция выводит значение свойства с поясняющим выводом сообщения, а также возвращает целочисленное  значение в виде постоянной переменной <code>10</code>. Далее следует проверка, которая имеет правильное входное свойство и проверка, которая имеет ошибочное входное свойство:

<span class="filename">Файл: src/lib.rs</span>

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

<span class="caption">Приложение 11-10: Проверка функции, которая использует макрос <code>println!</code></span>

Итог вывода на окно вывода приказы `cargo test`:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Обратите внимание, что нигде в этом выводе мы не видим сообщения `I got the value 4` , которое выводится при выполнении пройденной проверки. Этот вывод был записан. Итог неудачного проверки, `I got the value 8` , появляется в разделе итоговых итогов проверки, который также показывает причину неудачного проверки.

Если мы хотим видеть выведенные итоги прохождения проверок, мы можем сказать Ржавчина, чтобы она также показывала итоги успешных проверок с помощью `--show-output`.

```console
$ cargo test -- --show-output
```

Когда мы снова запускаем проверки из Приложения 11-10 с клеймом `--show-output` , мы видим следующий итог:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Запуск подмножества проверок по имени

Бывают случаи, когда в запуске всех проверок нет необходимости и нужно запустить только несколько проверок. Если вы работаете над функцией и хотите запустить проверки, которые исследуют её работу - это было бы удобно. Вы можете это сделать, используя приказ `cargo test`, передав в качестве переменной имена проверок.

Для отображения, как запустить объединение проверок, мы создадим объединение проверок для функции `add_two` function, как показано в Приложении 11-11, и постараемся выбрать какие из них запускать.

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

<span class="caption">Приложение 11-11: Три проверки с различными именами</span>

Если вы выполните приказ `cargo test` без уточняющих переменных, все проверки выполнятся одновременно:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Запуск одного проверки

Мы можем запустить один проверка с помощью указания его имени в приказу `cargo test`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Был запущен только проверка с названием `one_hundred`; два других проверки не соответствовали этому названию. Итоги проверки с помощью вывода `2 filtered out` дают нам понять, что у нас было больше проверок, но они не были запущены.

Таким образом мы не можем указать имена нескольких проверок; будет использоваться только первое значение, указанное для `cargo test` . Но есть способ запустить несколько проверок.

#### Использование фильтров для запуска нескольких проверок

Мы можем указать часть имени проверки, и будет запущен любой проверка, имя которого соответствует этому значению. Например, поскольку имена двух наших проверок содержат `add`, мы можем запустить эти два, запустив `cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Этот приказ запускала все проверки с `add` в имени и отфильтровывала проверка с именем `one_hundred` . Также обратите внимание, что раздел, в котором появляется проверка, становится частью имени проверки, поэтому мы можем запускать все проверки в разделе, фильтруя имя раздела.

### Пренебрежение проверок

Бывает, что некоторые проверки требуют продолжительного времени для своего исполнения, и вы хотите исключить их из исполнения при запуске `cargo test`. Вместо перечисления в приказной строке всех проверок, которые вы хотите запускать, вы можете определять проверки, требующие много времени для прогона, свойством `ignore`, чтобы исключить их, как показано здесь:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs}}
```

После `#[test]` мы добавляем строку `#[ignore]` в проверка, которую хотим исключить. Теперь, когда мы запускаем наши проверки, `it_works` запускается, а `expensive_test` пренебрегается:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

Функция `expensive_test` помечена как `ignored`. Если вы хотите выполнить только проверки, которые были отклонены, вы можете воспользоваться приказом `cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Управляя тем, какие проверки запускать, вы можете быть уверены, что итоги вашего `cargo test` будут быстрыми. Когда вы дойдёте до этапа, где имеет смысл проверить итоги проверок `ignored`, и у вас есть время дождаться их итогов, вы можете запустить их с помощью `cargo test -- --ignored`. Если вы хотите запустить все проверки независимо от того, пренебрегаются они или нет, выполните `cargo test -- --include-ignored`.
