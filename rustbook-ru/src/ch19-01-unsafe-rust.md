## Unsafe Rust

Во всех предыдущих главах этой книги мы обсуждали рукопись на Ржавчине безопасное управлению памятью в котором обеспечивается во время сборки. Однако внутри Ржавчина скрывается другой язык - небезопасная Ржавчина, который не обеспечивает безопасной работы с памятью. Этот язык называется *unsafe Ржавчина* и работает также как и первый, но предоставляет вам дополнительные возможности.

Небезопасная Ржавчина существует потому что по своей природе постоянная оценка довольно устоявшееся. Когда сборщик пытается определить, соответствует ли рукопись заверениям, то он скорее отвергнет несколько допустимых программ, чем пропустит несколько недопустимых. Не смотря на то, что рукопись *может* быть в порядке, если сборщик Ржавчины не будет располагать достаточными сведениями, чтобы убедиться в этом, он отвергнет рукопись. В таких случаях вы можете использовать небезопасную рукопись, чтобы сказать сборщику: "Поверь мне, я знаю, что делаю". Однако имейте в виду, что вы используете небезопасную Ржавчину на свой страх и риск: если вы неправильно используете небезопасную рукопись, могут возникнуть сбои, связанные с нарушением безопасности памяти, например, разыменование нулевого указателя.

Другая причина, по которой у Ржавчины есть небезопасное альтер эго, заключается в том, что по существу аппаратное обеспечение компьютера небезопасно. Если Ржавчина не позволяла бы вам выполнять небезопасные действия, вы не могли бы выполнять определённые задачи. Ржавчина должна позволить вам использовать системное, низкоуровневое программирование, такое как прямое взаимодействие с операционной системой, или даже написание вашей собственной операционной системы. Возможность написания низкоуровневого, системного рукописи является одной из целей языка. Давайте рассмотрим, что и как можно делать с небезопасным Ржавчиной.

### Небезопасные сверхспособности

Чтобы переключиться на небезопасная Ржавчина используйте ключевое слово `unsafe`, а затем начните новый раздел, содержащий небезопасный рукопись. В небезопасном Ржавчина можно выполнять пять действий, которые недоступны в безопасном Ржавчина, которые мы называем *небезопасными супер силами*. Эти супер силы включают в себя следующее:

- Разыменование сырого указателя
- Вызов небезопасной функции или небезопасного способа
- Доступ или изменение изменяемой постоянной переменной
- Выполнение небезопасной сущности
- Доступ к полям в `union`

Важно понимать, что `unsafe` не отключает проверку заимствования или любые другие проверки безопасности языка Ржавчина: если вы используете ссылку в небезопасной рукописи, она всё равно будет проверена. Единственное, что делает ключевое слово `unsafe` - даёт вам доступ к этим пяти возможностям, безопасность работы с памятью, в которых не проверяет сборщик. Вы по-прежнему получаете некоторую степень безопасности внутри небезопасного раздела.

Кроме того, `unsafe` не означает, что рукопись внутри этого раздела является неизбежно опасным или он точно будет иметь сбои с безопасностью памяти: цель состоит в том, что вы, как программист, заверяете, что рукопись внутри раздела `unsafe` будет обращаться к действительной памяти правильным образом.

Люди подвержены ошибкам и ошибки будут происходить, но требуя размещение этих четырёх небезопасных действия внутри разделов, помеченных как `unsafe`, вы будете знать, что любые ошибки, связанные с безопасностью памяти, будут находиться внутри `unsafe` разделов. Делайте `unsafe` разделы маленькими; вы будете благодарны себе за это позже, при исследовании ошибок с памятью.

Чтобы отделить небезопасную рукопись, советуется заключить небезопасную рукопись в безопасную абстракцию и предоставить безопасный API, который мы обсудим позже, когда будем обсуждать небезопасные функции и способы. Части встроенной библиотеки выполнены как проверенные, безопасные абстракции над небезопасным рукописью. Оборачивание небезопасной рукописи в безопасную абстракцию предотвращает возможную утечку использования `unsafe` рукописи во всех местах, где вы или ваши пользователи могли бы захотеть напрямую использовать возможность, выполненную `unsafe` рукописью, потому что использование безопасной абстракции само безопасно.

Давайте поговорим о каждой из четырёх небезопасных сверх способностей, и по ходу дела рассмотрим некоторые абстракции, которые обеспечивают безопасную внешнюю оболочкудля небезопасной рукописи.

### Разыменование сырых указателей

В главе 4 раздела ["Недействительные ссылки"](ch04-02-references-and-borrowing.html#dangling-references)<!--  --> мы упоминали, что сборщик заверяет, что ссылки всегда действительны. Небезопасная Ржавчина имеет два новых вида данных, называемых *сырыми указателями* (raw pointers), которые похожи на ссылки. Как и в случае ссылок, сырые указатели могут быть неизменяемыми или изменяемыми и записываться как `*const T` и `*mut T` соответственно. Звёздочка не является приказчиком разыменования; это часть имени вида данных. В среде сырых указателей *неизменяемый* (immutable) означает, что указателю нельзя напрямую присвоить что-то после того как он разыменован.

В отличие от ссылок и умных указателей, сырые указатели:

- могут пренебрегать правила заимствования и иметь неизменяемые и изменяемые указатели, или множество изменяемых указателей на одну и ту же область памяти
- не заверяют что ссылаются на действительную память
- могут быть пустые
- не выполняют самостоятельную очистку памяти

Отказавшись от этих заверений, вы можете обменять безопасность  на большую производительность или возможность взаимодействия с другим языком или оборудованием, где заверения Ржавчины не применяются.

В приложении 19-1 показано, как создать неизменяемый и изменяемый сырой указатель из ссылок.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-01/src/main.rs:here}}
```

<span class="caption">Приложение 19-1: Создание необработанных указателей из ссылок</span>

Обратите внимание, что мы не используем ключевое слово `unsafe` в этой рукописи. Можно создавать сырые указатели в безопасном рукописи; мы просто не можем разыменовывать сырые указатели за пределами небезопасного раздела, как вы увидите чуть позже.

Мы создали сырые указатели, используя `as` для приведения неизменяемой и изменяемой ссылки к соответствующим им видам сырых указателей. Поскольку мы создали их непосредственно из ссылок, которые обязательно являются действительными, мы знаем, что эти определенные сырые указатели являются действительными, но мы не можем делать такое же предположение о любом сыром указателе.

Чтобы отобразить это, создадим сырой указатель, в достоверности которого мы не можем быть так уверены. В приложении 19-2 показано, как создать необработанный указатель на произвольное место в памяти. Попытка использовать произвольную память является непредсказуемой: по этому адресу могут быть данные, а могут и не быть, сборщик может перерабатывать рукопись так, что доступа к памяти не будет, или программа может завершиться с ошибкой сегментации. Обычно нет веских причин писать такую рукопись, но это возможно.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-02/src/main.rs:here}}
```

<span class="caption">Приложение 19-2: Создание сырого указателя на произвольный адрес памяти</span>

Напомним, что можно создавать сырые указатели в безопасном рукописи, но нельзя *разыменовывать* сырые указатели и читать данные, на которые они указывают. В приложении 19-3 мы используем приказчик разыменования `*` для сырого указателя, который требует `unsafe` раздела.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-03/src/main.rs:here}}
```

<span class="caption">Приложение 19-3: Разыменование сырых указателей в разделе <code>unsafe</code></span>

Создание указателей безопасно. Только при попытке доступа к предмету по адресу в указателе мы можем получить недопустимое значение.

Также обратите внимание, что в примерах рукописи 19-1 и 19-3 мы создали `*const i32` и `*mut i32`, которые ссылаются на одну и ту же область памяти, где хранится `num`. Если мы попытаемся создать неизменяемую и изменяемую ссылку на `num` вместо сырых указателей, такая рукопись не собирается, т.к. будут нарушены правила заимствования, запрещающие наличие изменяемой ссылки одновременно с неизменяемыми ссылками. С помощью сырых указателей мы можем создать изменяемый указатель и неизменяемый указатель на одну и ту же область памяти и изменять данные с помощью изменяемого указателя, возможно создавая итог гонки данных. Будьте осторожны!

С учётом всех этих опасностей, зачем тогда использовать сырые указатели? Одним из основных применений является взаимодействие с рукописью C, как вы увидите в следующем разделе ["Вызов небезопасной функции или способа"](#calling-an-unsafe-function-or-method)<!--  -->. Другой случай это создание безопасных абстракций, которые не понимает оценщик заимствований. Мы введём понятие небезопасных функций и затем рассмотрим пример безопасной абстракции, которая использует небезопасный рукопись.

### Вызов небезопасной функции или способа

Второй вид действий, которые можно выполнять в небезопасном разделе - это вызов небезопасных функций. Небезопасные функции и способы выглядят точно так же, как обычные функции и способы, но перед остальным определением у них есть дополнительное `unsafe`. Ключевое слово `unsafe` в данном среде указывает на то, что к функции предъявляются требования, которые мы должны соблюдать при вызове этой функции, поскольку Ржавчина не может обеспечить, что мы их выполняем. Вызывая небезопасную функцию внутри раздела `unsafe`, мы говорим, что прочитали пособие к этой функции и берём на себя ответственность за соблюдение её условий.

Вот небезопасная функция с именем `dangerous` которая ничего не делает в своём теле:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Мы должны вызвать функцию `dangerous` в отдельном `unsafe` разделе. Если мы попробуем вызвать `dangerous` без `unsafe` раздела, мы получим ошибку:

```console
{{#include ../listings/ch19-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

С помощью раздела `unsafe` мы сообщаем Ржавчине, что прочитали пособие к функции, поняли, как правильно её использовать, и убедились, что выполняем договор функции.

Тела небезопасных функций являются в действительности `unsafe` разделами, поэтому для выполнения других небезопасных действий внутри небезопасной функции не нужно добавлять ещё один `unsafe` раздел.

#### Создание безопасных абстракций вокруг небезопасной рукописи

То, что функция содержит небезопасную рукопись, не означает, что мы должны пометить всю функцию как небезопасную. На самом деле, обёртывание небезопасной рукописи в безопасную функцию - это обычная абстракция. В качестве примера рассмотрим функцию `split_at_mut` из встроенной библиотеки, которая требует некоторой небезопасной рукописи. Рассмотрим, как мы могли бы её использовать. Этот безопасный способ определён для изменяемых срезов: он берет один срез и превращает его в два, разделяя срез по порядковому указателю, указанному в качестве переменной. В приложении 19-4 показано, как использовать `split_at_mut`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-04/src/main.rs:here}}
```

<span class="caption">Приложение 19-4: Использование безопасной функции <code>split_at_mut</code></span>

Эту функцию нельзя выполнить, используя только безопасную Ржавчина. Попытка выполнения могла бы выглядеть примерно как в приложении 19-5, которое не собирается. Для простоты мы используем `split_at_mut` как функцию, а не как способ, и только для значений вида данных `i32`, а не обобщённого вида данных `T`.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-05/src/main.rs:here}}
```

<span class="caption">Приложение 19-5: Попытка выполнения <code>split_at_mut</code> с использованием только безопасного Ржавчина</span>

Эта функция сначала получает общую длину среза. Затем она проверяет (assert), что порядковый указатель, переданный в качестве свойства, находится в границах среза, сравнивая его с длиной. Assert означает, что если мы передадим порядковый указатель, который больше, чем длина среза, функция завызывает сбой ещё до попытки использования этого порядкового указателя.

Затем мы возвращаем два изменяемых отрывка в упорядоченном ряде: один от начала исходного отрывка до `mid` порядкового указателя (не включая сам mid), а другой - от `mid` (включая сам mid) до конца отрывка.

При попытке собрать рукопись в приложении 19-5, мы получим ошибку.

```console
{{#include ../listings/ch19-advanced-features/listing-19-05/output.txt}}
```

Оценщик заимствований Ржавчина не может понять, что мы заимствуем различные части среза, он понимает лишь, что мы хотим осуществить заимствование частей одного среза дважды. Заимствование различных частей среза в принципе в порядке вещей, потому что они не перекрываются, но Ржавчина недостаточно умна, чтобы это понять. Когда мы знаем, что рукопись верна, но Ржавчина этого не понимает, значит пришло время прибегнуть к небезопасной рукописи.

Приложение 19-6 отображает, как можно использовать `unsafe` раздел, сырой указатель и вызовы небезопасных функций чтобы `split_at_mut` заработала:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-06/src/main.rs:here}}
```

<span class="caption">Приложение 19-6. Использование небезопасной рукописи в выполнении функции <code>split_at_mut</code></span>

Напомним, из раздела ["Вид данных срез"]<!-- ignore --> Главы 4, что срезы состоят из указателя на некоторые данные и длины. Мы используем способ `len` для получения длины среза и способ `as_mut_ptr` для доступа к сырому указателю среза. Поскольку у нас есть изменяемый срез на значения вида данных `i32`, функция `as_mut_ptr` возвращает сырой указатель вида данных `*mut i32`, который мы сохранили в переменной `ptr`.

Далее проверяем, что порядковый указатель`mid` находится в границах среза. Затем мы обращаемся к небезопасной рукописи: функция `slice::from_raw_parts_mut` принимает сырой указатель, длину и создаёт срез. Мы используем эту функцию для создания среза, начинающегося с `ptr` и имеющего длину в `mid` переменных. Затем мы вызываем способ `add` у `ptr` с `mid` в качестве переменной, чтобы получить сырой указатель, который начинается с `mid`, и создаём срез, используя этот указатель и оставшееся количество переменных после `mid` в качестве длины.

Функция `slice::from_raw_parts_mut` является небезопасной, потому что она принимает необработанный указатель и должна полагаться на то, что этот указатель действителен. Способ `add` для необработанных указателей также небезопасен, поскольку он должен считать, что местоположение смещения также является действительным указателем. Поэтому мы были вынуждены разместить `unsafe` раздел вокруг наших вызовов `slice::from_raw_parts_mut` и `add`, чтобы иметь возможность вызвать их. Посмотрев на рукопись и добавив утверждение, что `mid` должен быть меньше или равен `len`, мы можем сказать, что все необработанные указатели, используемые в разделе `unsafe`, будут правильными указателями на данные внутри среза. Это приемлемое и уместное использование `unsafe`.

Обратите внимание, что нам не нужно помечать итоговую функцию `split_at_mut` как `unsafe`, и мы можем вызвать эту функцию из безопасного Ржавчины. Мы создали безопасную абстракцию для небезопасной рукописи с помощью выполнения функции, которая использует рукопись `unsafe` раздела безопасным образом, поскольку она создаёт только допустимые указатели из данных, к которым эта функция имеет доступ.

Напротив, использование `slice::from_raw_parts_mut` в приложении 19-7 приведёт к вероятному сбою при использовании среза. Эта рукопись использует произвольный адрес памяти и создаёт срез из 10000 переменных.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-07/src/main.rs:here}}
```

<span class="caption">Приложение 19-7: Создание среза из произвольного адреса памяти</span>

Мы не владеем памятью в этом произвольном месте, и нет никакой заверения, что созданный этим рукописью отрывок содержит допустимые значения `i32`. Попытка использовать `values` так, как будто это допустимый срез, приводит к неопределённому поведению.

#### Использование `extern` функций для вызова внешней рукописи

Иногда вашей рукописи на языке Ржавчина может потребоваться взаимодействие с рукописью, написанной на другом языке. Для этого в Ржавчине есть ключевое слово `extern`, которое облегчает создание и использование *внешней оболочки внешних функций (Foreign Function Interface - FFI)*. FFI - это способ для языка программирования определить функции и позволить другому (внешнему) языку программирования вызывать эти функции.

Приложение 19-8 отображает, как настроить встраивание с функцией `abs` из встроенной библиотеки C. Функции, объявленные внутри разделов `extern`, всегда небезопасны для вызова из рукописи Ржавчины. Причина в том, что другие языки не обеспечивают соблюдение правил и заверений языка Ржавчины, Ржавчина также не может проверить заверения, поэтому ответственность за безопасность ложится на программиста.

<span class="filename">Имя файла: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-08/src/main.rs}}
```

<span class="caption">Приложение 19-8: Объявление и вызов <code>extern</code> функции, написанной на другом языке программирования</span>

Внутри раздела `extern "C"` мы перечисляем имена и ярлыки внешних функций из другого языка, которые мы хотим вызвать. Часть `"C"` определяет какой *application binary interface* (ABI - двоичная внешняя оболочка приложений) использует внешняя функция. Внешнюю оболочку ABI определяет как вызвать функцию на уровне ассемблера. Использование ABI `"C"` является наиболее часто используемым и следует правилам ABI внешней оболочки языка Си.

> #### Вызов функций Ржавчина из других языков
>
> Также можно использовать `extern` для создания внешней оболочки, позволяющего другим языкам вызывать функции Ржавчины. Вместо того чтобы создавать целый раздел`extern`, мы добавляем ключевое слово `extern` и указываем ABI для использования непосредственно перед ключевым словом `fn` для необходимой функции. Нам также нужно добавить изложение `#[no_mangle]`, чтобы сказать сборщику Ржавчины не искажать имя этой функции. *Искажение* - это когда сборщик меняет имя, которое мы дали функции, на другое имя, которое содержит больше сведений для других частей этапа сборки, но менее удобно для чтения для человека. Сборщик каждого языка программирования искажает имена по-разному, поэтому, чтобы функция Ржавчины могла быть использована другими языками, мы должны отключить искажение имён в сборщике Ржавчины.
>
> В следующем примере мы делаем функцию `call_from_c` доступной из рукописи на C, после того как она будет собрана в разделяемую библиотеку и прилинкована с C:
>
> ```rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Ржавчина function from C!");
> }
> ```
>
> Такое использование `extern` не требует `unsafe`.

### Получение доступа и внесение изменений в изменяемую постоянную переменную

В этой книге мы ещё не говорили о *вездесущих переменных*, которые Ржавчина поддерживает, но с которыми могут возникнуть сбои из-за действующих в Ржавчине правил владения. Если два потока обращаются к одной и той же изменяемой вездесущей переменной, это может привести к гонке данных.

Вездесущие переменные в Ржавчине называют *постоянными* (static). Приложение 19-9 отображает пример объявления и использования в качестве значения постоянной переменной, имеющей вид данных строкового среза:

<span class="filename">Имя файла: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-09/src/main.rs}}
```

<span class="caption">Приложение 19-9: Определение и использование неизменяемой постоянной переменной</span>

Постоянные переменные похожи на постоянные переменные, которые мы обсуждали в разделе [“Различия между переменными и постоянными переменными”](ch03-01-variables-and-mutability.html#constants)<!-- ignore --> Главы 3. Имена постоянных переменных по общему соглашению пишутся в наставлении `SCREAMING_SNAKE_CASE`, и мы <em>должны</em> указывать вид данных переменной, которым в данном случае является <code>&amp;'static str</code>. Постоянные переменные могут хранить только ссылки со временем жизни <code>'static</code>, это означает что сборщик Ржавчины может вывести время жизни и нам не нужно прописывать его явно. Доступ к неизменяемой постоянной переменной является безопасным.

Тонкое различие между постоянными переменными и неизменяемыми постоянными переменными заключается в том, что значения в постоянной переменной имеют определенный адрес в памяти. При использовании значения всегда будут доступны одни и те же данные. Постоянные переменные, с другой стороны, могут повторять свои данные при каждом использовании. Ещё одно отличие заключается в том, что постоянные переменные могут быть изменяемыми. Обращение к изменяемым постоянном переменным и их изменение является *небезопасным*. В приложении 19-10 показано, как объявить, получить доступ и изменять изменяемую постоянную переменную с именем `COUNTER`.

<span class="filename">Имя файла: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-10/src/main.rs}}
```

<span class="caption">Приложение 19-10: Чтение из изменяемой постоянной переменной или запись в неё небезопасны</span>

Как и с обычными переменными, мы определяем изменяемость с помощью ключевого слова `mut`. Любая рукопись, которая считывает из или пишет в переменную `COUNTER` должен находиться в `unsafe` разделе. Эта рукопись собирается и выводит `COUNTER: 3`, как и следовало ожидать, потому что выполняется в одном потоке. Наличие нескольких потоков с доступом к `COUNTER` приведёт к случаю гонки данных.

Наличие изменяемых данных, которые доступны вездесуще, делает трудным выполнение заверения отсутствия гонок данных, поэтому Ржавчина считает изменяемые постоянные переменные небезопасными. Там, где это возможно, предпочтительно использовать техники многопоточности и умные указатели, направленные на многопоточное исполнение, которые мы обсуждали в главе 16. Таким образом, сборщик сможет проверить, что обращение к данным, доступным из разных потоков, выполняется безопасно.

### Выполнение небезопасных сущностей

Мы можем использовать `unsafe` для выполнения небезопасной сущности. Сущность является небезопасной, если хотя бы один из его способов имеет некоторый неизменная величина, который сборщик не может проверить. Мы объявляем сущности `unsafe`, добавляя ключевое слово `unsafe` перед `trait` и помечая использование сущности как `unsafe`, как показано в приложении 19-11.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-11/src/main.rs}}
```

<span class="caption">Приложение 19-11: Определение и выполнение небезопасной сущности</span>

Используя `unsafe impl`, мы даём обещание поддерживать неизменные величины, которые сборщик не может проверить.

Для примера вспомним маркерные сущности `Sync` и `Send`, которые мы обсуждали в разделе <a data-md-type="raw_html" href="ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits">"Расширяемый одновременность с помощью сущностей `Sync` и `Send`"</a><!-- ignore --> Главы 16: сборщик использует эти сущности самостоятельно , если наши виды полностью состоят из видов данных `Send` и `Sync`. Если мы создадим вид, который содержит вид, не являющийся `Send` или `Sync`, такой, как сырой указатель, и мы хотим пометить этот вид как `Send` или `Sync`, мы должны использовать `unsafe` раздел. Ржавчина не может проверить, что наш вид поддерживает заверения того, что он может быть безопасно передан между потоками или доступен из нескольких потоков; поэтому нам нужно добавить эти проверки вручную и указать это с помощью `unsafe`.

### Доступ к полям объединений (union)

Последнее действие, которое работает только с `unsafe` - это доступ к полям *union*. `union` похож на `struct`, но в каждом определенном образце одновременно может использоваться только одно объявленное поле. Объединения в основном используются для взаимодействия с объединениями в рукописи на языке Си. Доступ к полям объединений небезопасен, поскольку Ржавчина не может обязательно определить вид данных, которые в данный мгновение хранятся в образце объединения. Подробнее об объединениях вы можете узнать в [the Ржавчина Reference].

### Когда использовать небезопасную рукопись

Использование `unsafe` для выполнения одного из пяти действий (супер способностей), которые только что обсуждались, не является ошибочным или не одобренным. Но получить правильный `unsafe` рукопись сложнее, потому что сборщик не может помочь в обеспечении безопасности памяти. Если у вас есть причина использовать `unsafe` рукопись, вы можете делать это, а наличие явной `unsafe` изложении облегчает отслеживание источника неполадок. если они возникают.


["Вид срез"]: ch04-03-slices.html#the-slice-type
[the Ржавчина Reference]: ../reference/items/unions.html