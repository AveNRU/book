## Как медленные запросы влияют на пропускную способность

Сейчас наш отдельный вычислитель обрабатывает каждый запрос по очереди. Это работает для систем
с небольшой загрузкой (которая получает не очень много запросов), но как только
приложения становятся более сложными, такая выполнение уже не будет разумной.

Поскольку наша текущая программа последовательно обрабатывает соединения, она не будет
обработать второе соединение, пока оно не завершит обработку первого. Если мы
получить один запрос, который требует много времени для обработки, запросам,
поступающие во время обработки придется подождать, пока длинный запрос не будет
завершен, даже если новый запрос может быть обработан быстро. Давайте посмотрим
на это в действии.

### Подражание медленного запроса в выполнении текущего отдельного вычислителя

Давайте посмотрим на итог от запроса, который требует много времени для обработки.
В рукописи 20-10 показано, пример симуляции медленной обработки запроса. Рукопись при ответе
на запрос `/sleep`, отдельный вычислитель "заснёт" на пять секунд.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;
# use std::io::prelude::*;
# use std::net::TcpStream;
# use std::fs::File;
// ...snip...

fn handle_connection(mut stream: TcpStream) {
#     let mut buffer = [0; 512];
#     stream.read(&mut buffer).unwrap();
    // ...snip...

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };

    // ...snip...
}
```

<span class="caption">рукопись 20-10: симуляция обработки медленного запроса</span>

Мы создали особый запрос `sleep`. При выполнении данного запроса будет 5-секундная
задержка, перед тем, как отобразиться содержимое файла "hello.html".

Вы можете увидеть в существующем времени, насколько прост наш отдельный вычислитель. В существующих дела
может происходить и более длинная задержка.

Запустите программу приказом `cargo run`, а затем в окне обозреватели запросите данные
по адресам `http://localhost:8080/` и `http://localhost:8080/sleep`. Если вы запросите
данные и строка запроса будет начинаться с `/` даже несколько раз - вы получите
быстрый ответ. Но если вы запросите `/sleep` и затем попробуете ещё раз получить
данные стартовой страницы - вы будете ожидать пока `sleep` рукопись функции не закончит
ожидания и не приступит к дальнейшей работе.

Существует несколько способов изменить работу нашего сетевого-отдельного вычислителя, чтобы избежать
повторного запроса всех запросов следовавших за медленным запросом. Тот, что мы собираемся сделать называется объединением потоков.

### Улучшение пропускной способности объединения потоков

*Объединение потоков* - объединение порожденных потоков, которые готовы обрабатывать некоторые
задача. Когда программа получает новую задачу, один из потоков в объединении будет
назначен выполнять эту задачу. Остальные потоки в объединении доступны для обработки
любых других задач, которые могут возникнуть во врем работы занятого потока.

Объединение потоков позволит нам одновременно обрабатывать соединения: мы можем начать
обработку нового соединения до завершения старого соединения. Это увеличит
пропускную способность нашего отдельного вычислителя.

Итак, вот что мы собираемся сделать: вместо ожидания каждого запроса перед тем, как начать с следующей, мы отправим обработку каждого соединение с другой поток. Потоки будут поступать из объединения , который мы будем создавать после запуска программы на выполнение. Причина, по которой мы ограничиваем число потоков на небольшое число (четыре) - потому, что если бы мы создавали бы
новый поток для каждого запроса, то мощности системы были бы быстро израсходованы при увеличении количества запросов.

Вместо того, чтобы создавать неограниченное количество потоков, у нас будет определенное
их количество в объединении. По мере поступления запросов мы будем отправлять запросы в
объединение для обработки. Объединение будет поддерживать очередь входящих запросов. Каждый из
потоков в объединении получает запрос из этой очереди, обрабатывает его, а затем запрашивает
следующий. С таким внешнем видом мы можем обрабатывает `N` запросы одновременно, где
`N` - количество потоков. Эта все равно означает, что длительные запросы `N` могут
привести к запасному воспроизведению запросов в очереди, но мы увеличили количество
длительных запросов, которые мы можем обрабатывать до этого времени от одного до `N`.

Такое решение является одним из способов повысить пропускную способность нашего
сетевого-отдельного вычислителя. Однако, это книга не о сетевых-отдельных вычислителях, поэтому мы не будем углубляться
в сбои исполнений. Скажем только, что способами увеличения пропускной способности
является прообраз fork/join и прообраз однопоточного не согласованного ввода-выводы. Если для вас важен этот вопрос, вы можете больше узнать о нем и попытаться сделать его
в Ржавчине. Ржавчина является языком низкого уровня и может исполнить все эти подходы.
