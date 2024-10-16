## Выполнение предметно-направленного образца разработки

*Образец "Состояние"* — это предметно-направленный образец разработки. Суть образца заключается в том, что мы определяем набор состояний, которые может иметь внутреннее значение. Состояния представлены набором *предметов состояния*, а поведение переменной изменяется в зависимости от его состояния. Мы рассмотрим пример устройства записи в дневнике, в которой есть поле для хранения состояния, которое будет предметом состояния из набора «черновик», «обзор» или «обнародовано».

Предметы состояния имеют общую возможность: конечно в Ржавчине мы используем виды данных и сущности, а не предметы и наследование. Каждый предмет состояния отвечает за своё поведение и сам определяет, когда он должен перейти в другое состояние. Переменная, которая содержит предмет состояния, ничего не знает о различиях в поведении состояний или о том, когда одно состояние должно перейти в другое.

Преимуществом образца "Состояние" является то, что при изменение требований заказчика программы не требуется изменять рукопись переменной, содержащей состояние, или рукопись, использующую такую переменную. Нам нужно только обновить рукопись внутри одного из предметов состояния, чтобы изменить его порядок действий, либо, возможно, добавить больше предметов состояния.

Для начала выполняем образец "Состояние" более привычным предметно-направленным способом, а затем воспользуемся подходом, более естественным для Ржавчины. Давайте шаг за шагом выполняем поток действий для записи в дневнике, использующий образец "Состояние".

Окончательный возможности будет выглядеть так:

1. Запись в дневнике создаётся как пустой черновик.
2. Когда черновик готов, запрашивается его проверка.
3. После проверки происходит обнародование записи.
4. Только обнародованные записи дневника возвращают содержимое записи на вывод, поэтому сообщения, не прошедшие проверку, не могут быть обнародованы случайно.

Любые другие изменения, сделанные в записи, не должны иметь никакого последствий. Например, если мы попытаемся подтвердить черновик записи в дневнике до того, как запросим проверку, запись должна остаться необнародованным черновиком.

В приложении 17-11 показан этот поток действий в виде рукописи: это пример использования API, который мы собираемся использовать в библиотеке (ящике) с именем `blog`. Он пока не собирается, потому что ящик `blog` ещё не создан.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:all}}
```

<span class="caption">Приложение 17-11: Рукопись, отображающая желаемое поведение, которое мы хотим получить в ящике <code>blog</code></span>

Мы хотим, чтобы пользователь мог создать новый черновик записи в дневнике с помощью `Post::new`. Затем мы хотим разрешить добавление писания в запись дневника. Если мы попытаемся получить содержимое записи сразу, до её проверки, мы не должны получить никакого писания на выходе, потому что запись все ещё является черновиком. Мы добавили утверждение (`assert_eq!`) в рукописи для опытных целей. Утверждение (assertion), что черновик записи дневника должен возвращать пустую строку из способа `content` было бы отличным состоящим из разделов проверкой, но мы не собираемся писать проверки для этого примера.

Далее мы хотим разрешить сделать запрос на проверку записи и хотим, чтобы `content` возвращал пустую строку, пока проверки не завершена. Когда запись пройдёт проверку, она должна быть обнародована, то есть при вызове `content` будет возвращён писание записи.

Обратите внимание, что единственный вид из ящика, с которым мы взаимодействуем - это вид данных `Post`. Этот вид данных будет использовать образец "Состояние" и будет содержать значение, которое будет являться одним из трёх предметов состояний, представляющих различные состояния, в которых может находиться запись: "черновик", "ожидание проверки" или "обнародовано". Управление переходом из одного состояния в другое будет осуществляться внутренней ходом мыслей вида данных `Post`. Состояния будут переключаться в итоге реакции на вызов способов образца `Post` пользователями нашей библиотеки, но пользователи не должны управлять изменениями состояния напрямую. Кроме того, пользователи не должны иметь возможность ошибиться с состояниями, например, обнародовать сообщение до его проверки.

### Определение `Post` и создание нового образца в состоянии черновика

Приступим к выполнения библиотеки! Мы знаем, что нам нужна общедоступный стопка `Post`, хранящая некоторое содержимое, поэтому мы начнём с определения стопки и связанной с ней открытой функцией `new` для создания образца `Post`, как показано в приложении 17-12. Мы также сделаем закрытую сущность `State`, которая будет определять поведение, которое должны будут иметь все предметы состояний стопки `Post`.

Затем `Post` будет содержать сущность-предмет `Box<dyn State>` внутри `Option<T>` в закрытом поле `state` для хранения предмета состояния. Чуть позже вы поймёте, зачем нужно использовать `Option<T>` .

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-12/src/lib.rs}}
```

<span class="caption">Приложение 17-12: Определение стопки <code>Post</code> и функции <code>new</code>, которая создаёт новый образец данных <code>Post</code>, сущности <code>State</code> и стопки <code>Draft</code></span>

Сущность `State` определяет поведение, совместно используемое различными состояниями поста. Все предметы состояний (`Draft` - "черновик", `PendingReview`  - "ожидание проверки" и `Published` - "обнародовано") будут использовать сущность `State`. Пока у этого сущности нет никаких способов, и мы начнём с определения состояния `Draft`, просто потому, что это первое состояние, с которого, как мы хотим, обнародование будет начинать свой путь.

Когда мы создаём новый образец данных `Post`, мы устанавливаем его поле `state` в значение `Some`, содержащее `Box`. Этот `Box` указывает на новый образец данных стопки `Draft`. Это заверяет, что всякий раз, когда мы создаём новый образец данных `Post`, он появляется как черновик. Поскольку поле `state` в стопке `Post` является закрытым, нет никакого способа создать `Post` в каком-либо другом состоянии! В функции `Post::new` мы объявим поле `content` новой пустой строкой вида данных `String`.

### Хранение писания содержимого записи

В приложении 17-11 показано, что мы хотим иметь возможность вызывать способ `add_text` и передать ему `&str`, которое добавляется к записанному содержимому записи дневника. Мы выполняем эту возможность как способ, а не делаем поле `content` открыто доступным, используя `pub`. Это означает, что позже мы сможем написать способ, который будет управлять, как именно читаются данные из поля `content`. Способ `add_text` довольно прост, поэтому давайте добавим его выполнение в раздел`impl Post`приложения 17-13:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-13/src/lib.rs:here}}
```

<span class="caption">Приложение 17-13: Выполнение <code>add_text</code> для добавления писания к <code>content</code> (содержимому записи)</span>

Способ `add_text` принимает изменяемую ссылку на `self`, потому что мы меняем образец данных `Post`, для которого вызываем `add_text`. Затем мы вызываем `push_str` для `String` у поля `content` и передаём `text` переменной для добавления к сохранённому `content`. Это поведение не зависит от состояния, в котором находится запись, таким образом оно не является частью образца "Состояние". Способ `add_text` вообще не взаимодействует с полем `state`, но это часть поведения, которое мы хотим поддерживать.

### Убедимся, что содержание черновика будет пустым

Даже после того, как мы вызвали `add_text` и добавили некоторый содержание в нашу запись, мы хотим, чтобы способ `content` возвращал пустой отрывок строки, так как запись всё ещё находится в черновом состоянии, как это показано в строке 7 приложения 17-11. А пока давайте выполняем способ `content` наиболее простым способом, который будет удовлетворять этому требованию: будем всегда возвращать пустой отрывок строки. Мы изменим рукопись позже, как только выполняем возможность изменить состояние записи, чтобы она могла бы быть обнародована. Пока что записи могут находиться только в черновом состоянии, поэтому содержимое записи всегда должно быть пустым. Приложение 17-14 показывает такую выполнение-заглушку:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-14/src/lib.rs:here}}
```

<span class="caption">Приложение 17-14: Добавление выполнения-заглушки для способа <code>content</code> в <code>Post</code>, которая всегда возвращает пустой отрывок строки.</span>

С добавленным таким образом способом `content` всё в приложении 17-11 работает, как задумано, вплоть до строки 7.

### Запрос на проверку записи меняет её состояние

Далее нам нужно добавить возможность для запроса проверки записи, которое должно изменить её состояние с `Draft` на `PendingReview`. Приложение 17-15 показывает такую рукопись:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-15/src/lib.rs:here}}
```

<span class="caption">Приложение 17-15: Использование способов <code>request_review</code> в стопке <code>Post</code> и сущности <code>State</code></span>

Мы добавляем в `Post` открытый способ с именем `request_review` ("запросить проверку"), который будет принимать изменяемую ссылку на `self`. Затем мы вызываем внутренний способ `request_review` для текущего состояния `Post`, и этот второй способ `request_review` поглощает текущее состояние и возвращает новое состояние.

Мы добавляем способ `request_review` в сущность `State`; все виды данных, использующие эту сущность, теперь должны будут использовать способ `request_review`. Обратите внимание, что вместо `self`, `&self` или `&mut self` в качестве первого свойства способа у нас указан `self: Box<Self>`. Эти правила написания означают, что способ действителен только при его вызове с обёрткой `Box`, содержащей наш вид данных. Эти правила написания становятся владельцем `Box<Self>`, делая старое состояние недействительным, поэтому значение состояния `Post` может быть преобразовано в новое состояние.

Чтобы поглотить старое состояние, способ `request_review` должен стать владельцем значения состояния. Это место, где приходит на помощь вид данных `Option` поля `state` записи `Post`: мы вызываем способ `take`, чтобы забрать значение `Some` из поля `state` и оставить вместо него значение `None`, потому что Ржавчина не позволяет иметь необъявленные поля в стопках. Это позволяет перемещать значение `state` из `Post`, а не заимствовать его. Затем мы установим новое значение `state` как итог этого действия.

Нам нужно временно установить `state` в `None`, вместо того, чтобы установить его напрямую с помощью приказа вроде `self.state = self.state.request_review();`. Нам нужно завладеть значением поля `state`. Это даст нам заверение, что `Post` не сможет использовать старое значение `state` после того, как мы преобразовали его в новое состояние.

Способ `request_review` в `Draft` должен вернуть новый образец данных новой стопки `PendingReview`, обёрнутый в Box. Эта стопка будет представлять состояние, в котором запись ожидает проверки. Стопка `PendingReview` также использует способ `request_review`, но не выполняет никаких преобразований. Она возвращает сама себя, потому что, когда мы запрашиваем проверку записи, уже находящейся в состоянии `PendingReview`, она всё так же должна продолжать оставаться в состоянии `PendingReview`.

Теперь мы начинаем видеть преимущества образца "Состояние": способ `request_review` для `Post` одинаков, он не зависит от значения `state`. Каждое состояние само несёт ответственность за свои действия.

Оставим способ `content` у `Post` таким как есть, возвращающим пустой отрывок строки. Теперь мы можем иметь `Post` как в состоянии `PendingReview`, так и в состоянии `Draft`, но мы хотим получить такое же поведение в состоянии `PendingReview`. Приложение 17-11 теперь работает до строки 10!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

### Добавление `approve` для изменения поведения `content`

Способ `approve` ("одобрить") будет подобен способу `request_review`: он будет устанавливать у `state` значение, которое должна иметь запись при её одобрении, как показано в приложении 17-16:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-16/src/lib.rs:here}}
```

<span class="caption">Приложение 17-16: Использование способа <code>approve</code> для  данных <code>Post</code> и сущности <code>State</code></span>

Мы добавляем способ `approve` в сущность `State`, добавляем новую стопку, которая использует эту сущность `State` и стопку для состояния `Published`.

Подобно тому, как работает `request_review` для `PendingReview`, если мы вызовем способ `approve` для `Draft`, он не будет иметь никакого последствий, потому что `approve` вернёт `self`. Когда мы вызываем для `PendingReview` способ `approve`, то он возвращает новый упакованный образец данных стопки `Published`. Стопка `Published` использует сущность `State`, и как для способа `request_review`, так и для способа `approve` она возвращает себя, потому что в этих случаях запись должна оставаться в состоянии `Published`.

Теперь нам нужно обновить способ `content` для `Post`. Мы хотим, чтобы значение, возвращаемое из `content`, зависело от текущего состояния `Post`, поэтому мы собираемся перенести часть возможности `Post` в способ `content`, заданный для `state`, как показано в приложении 17.17:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-17/src/lib.rs:here}}
```

<span class="caption">Приложение 17-17: Обновление способа <code>content</code> в стопке <code>Post</code> для делегирования части возможности способу <code>content</code> стопка <code>State</code></span>

Поскольку наша цель состоит в том, чтобы сохранить все эти действия внутри стопок, использующих сущность `State`, мы вызываем способ `content` у значения в поле `state` и передаём образец обнародования (то есть `self` ) в качестве переменной. Затем мы возвращаем значение, которое нам выдаёт вызов способа `content` поля `state`.

Мы вызываем способ `as_ref` у `Option`, потому что нам нужна ссылка на значение внутри `Option`, а не владение значением. Поскольку `state` является видом данных `Option<Box<dyn State>>`, то при вызове способа `as_ref` возвращается `Option<&Box<dyn State>>`. Если бы мы не вызывали `as_ref`, мы бы получили ошибку, потому что мы не можем переместить `state` из заимствованного свойства `&self` функции.

Затем мы вызываем способ `unwrap`. Мы знаем, что этот способ здесь никогда не приведёт к завершению программы со сбоем, так все способы `Post` устроены таким образом, что после их выполнения, в поле `state` всегда содержится значение `Some`. Это один из случаев, про которых мы говорили в разделе ["Случаи, когда у вас больше сведений, чем у сборщика"]<!--  --> Главы 9 - случай, когда мы знаем, что значение `None` никогда не встретится, даже если сборщик не может этого понять.

Теперь, когда мы вызываем `content` у вида данных `&Box<dyn State>`, в действие вступает принудительное приведение (deref coercion) для `&` и `Box`, поэтому в конечном итоге способ `content` будет вызван для вида данных, который использует сущность `State`. Это означает, что нам нужно добавить способ `content` в определение сущности `State`, и именно там мы поместим ход мыслей для определения того, какое содержимое возвращать, в зависимости от текущего состояния, как показано в приложении 17-18:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-18/src/lib.rs:here}}
```

<span class="caption">Приложение 17-18: Добавление способа <code>content</code> в сущность <code>State</code></span>

Мы добавляем выполнение по умолчанию способа `content`, который возвращает пустой отрывок строки. Это означает, что нам не придётся использовать `content` в стопках `Draft` и `PendingReview`.  Стопка `Published` будет переопределять способ `content` и вернёт значение из `post.content`.

Обратите внимание, что для этого способа нам нужны изложении времени жизни, как мы обсуждали в главе 10. Мы берём ссылку на `post` в качестве переменной и возвращаем ссылку на часть этого `post`, поэтому время жизни возвращённой ссылки связано с временем жизни переменной `post`.

И вот, мы закончили - теперь всё из приложения 17-11 работает! Мы выполнили образец "Состояние", определяющий правила этапа работы с записью в дневнике. Ход мыслей, связанная с этими правилами, находится в предмета. состояний, а не разбросана по всей стопке `Post`.

> #### Почему не перечисление?
>
> Возможно, вам было важно, почему мы не использовали `enum` с различными возможными состояниями записи в качестве исходов. Это, безусловно, одно из возможных решений. Попробуйте его использовать и сравните конечные итоги, чтобы выбрать, какой из исходов вам больше нравится! Одним из недостатков использования перечисления является то, что в каждом месте, где проверяется значение перечисления, потребуется выражение `match` или что-то подобное для обработки всех возможных исходов. Возможно в этом случае нам придётся повторять больше рукописи, чем это было в решении с сущность-предметом.

### Установленные ограничения образца "Состояние"

Мы показали, что Ржавчина способна использовать предметно-направленный образец "Состояние" для инкапсуляции различных видов поведения, которые должна иметь запись в каждом состоянии. Способы в `Post` ничего не знают о различных видах поведения. При таком согласовании рукописи, нам достаточно взглянуть только на один его участок, чтобы узнать отличия в общедоступном поведении: в использование сущности `State` у стопки `Published`.

Если бы мы захотели создать иную выполнение, не использующую образец состояния, мы могли бы вместо этого использовать выражения `match` в способах `Post` или даже в `main`, которые бы проверяли состояние записи и изменяли поведение в этих местах. Это приведёт к тому, что нам придётся в нескольких местах исследовать все следствия того, что пост перешёл в состояние "обнародовано"! И эта нагрузка будет только увеличиваться по мере добавления новых состояний: для каждого из этих выражений `match` потребуются дополнительные ответвления.

С помощью образца "Состояние" способы `Post` и участки, где мы используем `Post`, не требуют использования выражений `match`, а для добавления нового состояния нужно только добавить новую стопку и использовать способы сущности у одной этой стопки.

Выполнение с использованием образца "Состояние" легко расширить для добавления новой возможности. Чтобы увидеть, как легко поддерживать рукопись, использующую данный образец, попробуйте выполнить некоторые из предложений ниже:

- Добавьте способ `reject`, который изменяет состояние обнародования с `PendingReview` обратно на `Draft`.
- Потребуйте два вызова способа `approve`, прежде чем переводить состояние в `Published`.
- Разрешите пользователям добавлять письменное содержимое только тогда, когда обнародование находится в состоянии `Draft`. Подсказка: пусть предмет состояния решает, можно ли менять содержимое, но не отвечает за изменение `Post`.

Одним из недостатков образца "Состояние" является то, что поскольку состояния сами выполняют переходы между собой, некоторые из состояний получаются связанными друг с другом. Если мы добавим другое состояние между `PendingReview` и `Published`,  например `Scheduled` ("рассчитано наперед"), то придётся изменить рукопись в `PendingReview`, чтобы оно теперь переходило в `Scheduled`. Если бы не нужно было менять `PendingReview` при добавлении нового состояния, было бы меньше работы, но это означало бы, что мы переходим на другой образец разработки.

Другим недостатком является то, что мы сделали повторение некоторую ход мыслей. Чтобы устранить некоторое повторение, мы могли бы попытаться сделать выполнения по умолчанию для способов `request_review` и `approve` сущности `State`, которые возвращают `self`; однако это нарушило бы безопасность предмета. потому что сущность не знает, каким определенно будет `self`. Мы хотим иметь возможность использовать `State` в качестве сущность-предмета. поэтому нам нужно, чтобы его способы были предметно-безопасными.

Другое повторение включает в себя схожие выполнения способов `request_review` и `approve` у  `Post`. Оба способа делегируют выполнения одного и того же способа значению поля `state` вида данных `Option` и устанавливают итогом новое значение поля `state`. Если бы у `Post` было много способов, которые следовали этому образцу, мы могли бы рассмотреть определение макроса для устранения повторения (смотри раздел ["Макросы"]<!--  --> в главе 19).

Выполняя образец "Состояние" точно так, как он определён для предметно-направленных языков, мы не настолько полно используем преимущества Ржавчина, как могли бы. Давайте посмотрим на некоторые изменения, которые мы можем внести в ящик `blog`, чтобы недопустимые состояния и переходы превратить в ошибки времени сборки.

#### Кодирование состояний и поведения в виде данных

Мы покажем вам, как переосмыслить образец "Состояние", чтобы получить другой набор соглашений. Вместо того, чтобы полностью инкапсулировать состояния и переходы, так, чтобы внешняя рукопись не знала о них, мы будем кодировать состояния с помощью разных видов данных. Следовательно, устройство проверки видов данных Ржавчины предотвратит попытки использовать черновики сообщений, там где разрешены только обнародованные сообщения, вызывая ошибки сборки.

Давайте рассмотрим первую часть `main` в приложении 17-11:

<span class="filename">Файл: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:here}}
```

Мы по-прежнему поддерживаем создание новых сообщений в состоянии "черновика" с помощью способа `Post::new` и возможность добавлять писание к содержимому обнародования. Но вместо способа `content` у чернового сообщения, возвращающего пустую строку, мы сделаем так, что у черновых сообщений вообще не будет способа `content`. Таким образом, если мы попытаемся получить содержимое черновика, мы получим ошибку сборщика, сообщающую, что способ не существует. В итоге мы не сможем случайно отобразить черновик содержимого записи в работающей программе, потому что эта рукопись даже не собирается. В приложении 17-19 показано определение стопок `Post` и `DraftPost`, а также способы для каждой из них:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-19/src/lib.rs}}
```

<span class="caption">Приложение 17-19: Стопка <code>Post</code> со способом <code>content</code> и стопка <code>DraftPost</code> без способа <code>content</code></span>

Обе стопки, `Post` и `DraftPost`, имеют закрытое поле `content`, в котором хранится писание сообщения дневника. Стопки больше не содержат поле `state`, потому что мы перемещаем кодирование состояния в виды данных. Стопка `Post` будет представлять обнародованное размещение, и у неё есть способ `content`, который возвращает `content`.

У нас все ещё есть функция `Post::new`, но вместо возврата образца `Post` она возвращает образец данных `DraftPost`. Поскольку поле `content` является закрытым и нет никаких функций, которые возвращают `Post`, просто так создать образец данных `Post` уже невозможно.

Стопка `DraftPost` имеет способ `add_text`, поэтому мы можем добавлять писание к `content` как и раньше, но учтите, что в `DraftPost` не определён способ `content`! Так что теперь программа заверяет, что все записи начинаются как черновики, а черновики размещений не имеют своего содержания для отображения. Любая попытка обойти эти ограничения приведёт к ошибке сборщика.

#### Выполнение переходов в виде преобразований в другие виды данных

Так как же получить обнародованный пост? Мы хотим обеспечить соблюдение правила, согласно которому черновик записи должен быть рассмотрен и утверждён до того, как он будет обнародован. Запись, находящаяся в состоянии проверки, по-прежнему не должна отображать содержимое. Давайте выполняем эти ограничения, добавив ещё одну стопку, `PendingReviewPost`, определив способ `request_review` у `DraftPost`, возвращающий `PendingReviewPost`, и определив способ `approve` у `PendingReviewPost`, возвращающий `Post`, как показано в приложении 17-20:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-20/src/lib.rs:here}}
```

<span class="caption">Приложение 17-20: Вид данных <code>PendingReviewPost</code>, который создаётся путём вызова <code>request_review</code> образца <code>DraftPost</code> и способ <code>approve</code>, который превращает <code>PendingReviewPost</code> в обнародованный <code>Post</code>.</span>

Способы `request_review` и `approve` забирают во владение `self`, таким образом поглощая образцы `DraftPost` и `PendingReviewPost`, которые потом преобразуются в `PendingReviewPost` и обнародованную `Post`, соответственно. Таким образом, у нас не будет никаких долгоживущих образцов `DraftPost`, после того, как мы вызвали у них `request_review` и так далее. В стопке `PendingReviewPost` не определён способ `content`, поэтому попытка прочитать его содержимое приводит к ошибке сборщика, также как и в случае с `DraftPost`. Так как единственным способом получить обнародованный образец данных `Post`, у которого действительно есть объявленный способ `content`, является вызов способа `approve` у образца `PendingReviewPost`, а единственный способ получить `PendingReviewPost` - это вызвать способ `request_review` у образца `DraftPost`, теперь мы закодировали этап смены состояний записи дневника с помощью системы видов данных.

Кроме этого, нужно внести небольшие изменения в `main`. Так как способы `request_review` и `approve` теперь возвращают предметы, а не преобразуют стопку от которой были вызваны, нам нужно добавить больше затеняющих присваиваний `let post =`, чтобы сохранять возвращаемые предметы. Также, теперь мы не можем использовать утверждения (assertions) для проверки того является ли содержимое черновиков и записей, находящихся на рассмотрении, пустыми строками, да они нам и не нужны - теперь стало невозможным собрать рукопись, которая бы пыталась использовать содержимое записей, находящихся в этих состояниях. Обновлённая рукопись в `main` показана в приложении 17-21:

<span class="filename">Файл: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-21/src/main.rs}}
```

<span class="caption">Приложение 17-21: Изменения в <code>main</code>, использующие новую выполнение этапа подготовки записи дневника</span>

Изменения, которые нам нужно было внести в `main`, чтобы переназначить `post` означают, что эта выполнение теперь не совсем соответствует предметно-направленному образцу "Состояние": преобразования между состояниями больше не инкапсулированы внутри выполнения `Post` полностью. Тем не менее, мы получили большую выгоду в том, что недопустимые состояния теперь невозможны из-за системы видов данных и проверки видов данных, которая происходит во время сборки! У нас есть заверенияия, что некоторые ошибки, такие как отображение содержимого необнародованной обнародования, будут обнаружены до того, как они дойдут до пользователей.

Попробуйте выполнить задачи, предложенные в начале этого раздела, в исполнении ящика `blog`, каким он стал после приложения 17-20, чтобы создать своё мнение о внешнем виде этой исполнения рукописи. Обратите внимание, что некоторые задачи в этом исходе могут быть уже выполнены.

Мы увидели, что хотя Ржавчина и способен использовать предметно-направленные образцы разработки, в нём также доступны и другие образцы, такие как кодирование состояния с помощью системы видов данных. Эти подходы имеют различные соглашения. Хотя вы, возможно, очень хорошо знакомы с предметно-направленными образцами данных, переосмысление неполадок для использования преимуществ и возможностей Ржавчина может дать такие выгоды, как предотвращение некоторых ошибок во время сборки. Предметно-направленные образцы не всегда будут лучшим решением в Ржавчине из-за наличия определённых возможностей, таких как владение, которого нет у предметно-направленных языков.

## Итоги

Независимо от того, что вы думаете о принадлежности Ржавчина к предметно-направленным языкам после прочтения этой Главы, теперь вы знаете, что можете использовать сущность-предметы, чтобы использовать некоторые предметно-направленные свойства в Ржавчине Изменяемая управление может дать вашей рукописи некоторую гибкость в обмен на небольшое ухудшение производительности во время выполнения. Вы можете использовать эту гибкость для выполнения предметно-направленных образцов, которые могут улучшить сопровождаемость вашей рукописи. В Ржавчине также есть другие особенности, такие как владение, которых нет у предметно-направленных языков. Предметно-направленный образец не всегда будет лучшим способом использовать преимущества Ржавчина, но является доступной возможностью.

Далее мы рассмотрим образцы, которые являются ещё одной особенностью Ржавчина, обеспечивающей высокую гибкость. Мы бегло рассказывали о них на протяжении всей книги, но ещё не видели всех их возможностей. Вперёд!


["Случаи, когда у вас больше сведений, чем у сборщика"]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
["Макросы"]: ch19-06-macros.html#macros