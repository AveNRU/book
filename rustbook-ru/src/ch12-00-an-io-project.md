# Дело с вводом/выводом (I/O): создание с окном вывода приложения

В этой главе вы примените многие знания, полученные ранее, а также познакомитесь с ещё неизученными API встроенной библиотеки. Мы создадим  приложение с окном вывода, которое будет взаимодействовать с файлом и вводом с окном ввода / выводом, чтобы применить в некоторых подходах Ржавчина, с которыми вы уже знакомы.

Скорость, безопасность, сборка в один исполняемый файл и кроссплатформенность делают Ржавчина наилучшим языком для создания средств с окнами ввода/вывода, так что в нашем деле мы создадим свою собственную исполнение обычной утилиты поиска `grep`, что расшифровывается, как "вездесущеее средство поиска и выводе" (**g**lobally search a **r**egular **e**xpression and **p**rint). В простейшем случае `grep` используется для поиска в выбранном файле указанного писания. Для этого утилита `grep` получает имя файла и писание в качестве переменных. Далее она считывает файл, находит и выводит строки, содержащие искомый писание.

Попутно мы покажем, как сделать так, чтобы наше приложение с окном вывода использовало возможности окна вызова, которые используются многими другими средствами с окнами ввода/вывода. Мы будем читать значение переменной окружения, чтобы позволить пользователю настроить поведение нашего средства. Мы также будем выводить сообщения об ошибках в обычный поток ошибок ( `stderr` ) вместо принятого вывода ( `stdout` ), чтобы, к примеру, пользователь мог перенаправить успешный вывод в файл, в то время, как сообщения об ошибках останутся на экране.

Один из участников Ржавчины-сообщества, Andrew Gallant, уже выполнил полновозможный, очень быстрый подобие программы `grep` и назвал его `ripgrep`. По сравнению с ним, наша исполнение будет довольно простой, но эта глава даст вам знания, которые нужны для понимания существующих дел, таких как <code>ripgrep</code>.

Наше дело  `grep` будет использовать ранее изученные подходы:

- Создание рукописи (используя то, что вы узнали о разделах в [ главе 7]<!--  -->)
- Использование векторов и строк (собрания, [глава 8]<!--  -->)
- Обработка ошибок ([Глава 9]<!--  -->)
- Использование сущностей и времени жизни там, где это необходимо ([глава 10]<!--  -->)
- Написание проверок ( [Глава 11]<!--  -->)

Мы также кратко представим замыкания, повторители и предметы сущности, которые будут объяснены подробно в главах [13]<!--  --> и [17]<!--  -->.


[ главе 7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[глава 8]: ch08-00-common-collections.html
[Глава 9]: ch09-00-error-handling.html
[глава 10]: ch10-00-generics.html
[Глава 11]: ch11-00-testing.html
[13]: ch13-00-functional-features.html
[17]: ch17-00-oop.html