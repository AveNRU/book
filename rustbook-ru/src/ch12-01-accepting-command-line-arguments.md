## Принятие переменных приказной строки

Создадим новое дело с окном вывода приложения как обычно с помощью приказов `cargo new`. Мы назовём дело `minigrep`, чтобы различать наше приложение от `grep`, которое возможно уже есть в вашей системе.

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Первая задача - заставить `minigrep` принимать две переменной приказной строки: путь к файлу и строку для поиска. То есть мы хотим иметь возможность запускать нашу программу через `cargo run`, с использованием двойного дефиса, чтобы указать, что следующие переменные предназначены для нашей программы, а не для `cargo`, строки для поиска и пути к файлу в котором нужно искать, как описано ниже:

```console
$ cargo run -- searchstring example-filename.txt
```

В данный мгновение программа созданная `cargo new` не может обрабатывать переменные, которые мы ей передаём. Некоторые существующие библиотеки на [crates.io](https://crates.io/) могут помочь с написанием программы, которая принимает переменные приказной строки, но так как вы просто изучаете этот подход, давайте применим эту возможность сами.

### Чтение значений переменных

Чтобы `minigrep` мог воспринимать значения переменных приказной строки, которые мы ему передаём, нам понадобится функция `std::env::args`, входящая во встроенную библиотеку Ржавчины. Эта функция возвращает повторитель переменных приказной строки, переданных в `minigrep`. Мы подробно рассмотрим повторители в [главе 13]<!-- ignore -->. Пока вам достаточно знать две вещи об повторителях: повторители порождают последовательность значений, и мы можем вызвать способ `collect` у повторителя, чтобы создать из него собрание, например вектор, который будет содержать все переменные, произведённые повторителем.

Рукопись представленный в Приложении 12-1 позволяет вашей программе `minigrep` читать любые переданные ей переменные приказной строки, а затем собирать значения в вектор.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

<span class="caption">Приложение 12-1: Собираем переменные приказной строки в вектор и выводим их на вывод</span>

Сначала мы вводим раздел `std::env` в область видимости с помощью указания `use`, чтобы мы могли использовать его функцию `args`. Обратите внимание, что функция `std::env::args` вложена в два уровня разделов. Как мы обсуждали в [главе 7]<!-- ignore -->, в случаях, когда нужная функция оказывается вложенной в более чем один раздел, советуется выносить в область видимости родительский раздел, а не функцию. Таким образом, мы можем легко использовать другие функции из `std::env`. Это менее двусмысленно, чем добавление `use std::env::args` и последующий вызов функции только с `args`, потому что `args` может быть легко принят за функцию, определённую в текущем разделе.

> ### Функция `args` и недействительный Юнирукопись знак (Unicode)
>
> Обратите внимание, что `std::env::args` вызовет сбой, если какой-либо переменная содержит недопустимый знак Юнирукописи. Если вашей программе необходимо принимать переменные, содержащие недопустимые знаки Unicode, используйте вместо этого `std::env::args_os`. Эта функция возвращает повторитель , который выдаёт значения `OsString` вместо значений `String`. Мы решили использовать `std::env::args` здесь для простоты, потому что значения `OsString` отличаются для каждой площадки и с ними сложнее работать, чем со значениями `String`.

В первой строке рукописи функции `main` мы вызываем `env::args` и сразу используем способ `collect`, чтобы превратить повторитель в вектор содержащий все полученные значения. Мы можем использовать функцию `collect` для создания многих видов собраний, поэтому мы явно определяем вид данных `args` чтобы указать, что мы хотим вектор строк. Хотя нам очень редко нужно определять виды данных в Ржавчине, `collect` - это одна из функций, с которой вам часто нужно изложение вида данных, потому что Ржавчина не может сама вывести именно то собрание, какое вы хотите.

И в заключение мы выводим вектор с помощью отладочного макроса. Попробуем запустить рукопись сначала без переменных, а затем с двумя переменными:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Обратите внимание, что первое значение в векторе `"target/debug/minigrep"` является названием нашего двоичного файла. Это соответствует поведению списка переменных в Си, позволяя программам использовать название с которым они были вызваны при выполнении. Часто бывает удобно иметь доступ к имени программы, если вы хотите вывести его в сообщениях или изменить поведение программы в зависимости от того, какой псевдоним приказной строки был использован для вызова программы. Но для целей этой Главы, мы пренебрегаем им и сохраним только две переменной, которые нам нужны.

### Сохранения значений переменных в переменные

На текущий мгновение программа может получить доступ к значениям, указанным в качестве переменных приказной строки. Теперь нам требуется сохранять значения этих двух переменных в переменных, чтобы мы могли использовать их в остальных частях программы. Мы сделаем это в приложении 12-2.

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

<span class="caption">Приложение 12-2: Создание переменных для хранения значений переменных искомой подстроки и пути к файлу</span>

Как видно из вывода вектора, имя программы занимает первое значение в векторе по адресу `args[0]`, значит, переменные начинаются с порядкового указателя `1`. Первая переменная `minigrep` - это строка, которую мы ищем, поэтому мы помещаем ссылку на первую переменную в переменную `query`. Второй переменной является путь к файлу, поэтому мы помещаем ссылку на вторую переменную в переменную `file_path`.

Для проверки соблюдения правил работы нашей программы, значения переменных выводятся в окно вывода. Далее, запустим нашу программу со следующими переменными: `test` и `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Отлично, программа работает! Нам нужно чтобы значения переменных были сохранены в правильных переменных. Позже мы добавим обработку ошибок с некоторыми вероятными ошибочными случайми, например, когда пользователь не предоставляет переменные; сейчас мы пренебрегаем эту случай и поработаем над добавлением возможности чтения файла.


[главе 13]: ch13-00-functional-features.html
[главе 7]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths