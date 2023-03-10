# Лекция 4

## Классификация ошибок параллельного программирования

### Дедлоки

- захват нерекурсивного примитива синхронизации **два раза подряд в одном потоке**

*Вообще за этим тоже нужно отдельно следить, поскольку в разных ЯП блокировки по умолчанию разные: в С++, например, нерекурсивная, а в Java рекурсивный `ReentrantLock`.*

- **перекрестный дедлок**: в двух потоках захватывается по примитиву, а потом нужно захватить эти же примитивы в другом порядке

*Обычно решается сменой порядка захвата примитивов синхронизации.*

Для поиска ошибка параллельного программирования можно запускать приложение под управлением valgrind с помощью `valgrind --tool=helgrind` (неэффективное с точки зрения решение --- транслирует машинный код во внутреннее представление, исполняемое на виртуальном процессоре).

Обычно алгоритм поиска дедлоков просто запоминает, где берутся примитивы, и помечает, где потенциально может возникнуть дедлок. Проблема с таким подходом в том, что алгоритм может **не знать о самописных примитивах синхронизации**, в частности скрытых спин-локах.

- **дедлок при форке процесса с захваченным примитивом синхронизации**: пусть есть процесс с двумя потоками, первый поток захватывает ресурс, а второй поток делает `fork()`. В процессе-потомке будет один поток, который запустится с места, где был вызван `fork()` (при этом здесь он вернет ноль, а в родителе --- идентификатор процесса-потомка)  . Если он попытается взять тот же самый примитив, будет дедлок, поскольку примитив был скопирован в процесс-потомок в захваченном состоянии с идентификатором потока, который был в процессе-родителе.

Автоматически это никак не обнаружить, **но решить можно**. В POSIX это делается с помощью `pthread_at_fork()`; она будет выполняться при каждом `fork()` и ее сигнатура включает в себя:
- указатель на функцию, которая будет выполнена в рамках процеса-родителя перед `fork()`
- указатель на функцию, которая будет выполнена в рамках процеса-родителя после`fork()`
- указатель на функцию, которая будет выполнена в рамках процеса-потомка после`fork()`

Однако, **атомарность `pthread_at_fork()` не гарантируется**, поэтому нужно продумать возможность захвата освобожденных на время `fork()` примитивов другим потоком. Как ее можно использовать:
- в первой функции захватываем все примитивы синхронизации в потоке, который делает `fork()`, как будто структуры, защищаемые ими, нам нужны
- во второй функции освобождаем все примитивы синхронизации в этом потоке
- в третьей функции мы переинициализируем все примитивы синхронизации в единственном потоке в новом процессе

Естественно, потоку, который делает `fork()`, необходим доступ ко всем примитивам синхронизации во всем процессе. Более того, некоторые ОС запрещают делать `fork()` в многопоточном приложении.

*Вообще при `fork()` все адресное пространство не копируется. В ОС применяется **оптимизация copy-on-write**, по которой изначально виртуальное адресное пространство процесса-потомка всеми страницами ссылается на реальную физическую память первого процесса и только те страницы, в которых произошли изменения, реально аллоцируются физически в пространстве процесса-потомка.*

### Гонки данных

Ошибки типа **гонки данных** (data races) порождают неконсистентные данные. Простой пример: запись 64-битного числа на 32-битной платформе. Нам нужно 2 инструкции, чтобы считать число, и между ними другим потоком может быть исполнена инструкция, меняющая считываемое число. Еще одним примером является параллельный инкремент одного и того же числа, а именно если инкремент выполняется в три инструкции `mov`, `inc`, `mov`, то инкременты могут быть отработаны "вместе" и оба раза будет записано число, большее исходного на 1, а не на 2.

Более высокоуровневый пример: пусть есть некоторое хранилище со значениями `{"Тополь", 3}` и `{"Сатана", 12}`. Выполняя код: `x = A; x = B` параллельно с `y = x` в `y` может быть записано что-то вроде `{"Сатана", 3}` из-за неатомарного присваивания. Подобные ошибки тяжело отлавливать из-за того, что порой неясно, как понять, что полученный элемент не приналдежит предметной области.

Задача поиска мест с потенциальной гонкой данных --- **неразрешимая задача** (разве что можно использовать ЯП без общей памяти, например, функциональные). В имеющихся инструментах используется проход по коду и отлавливание определенных паттернов (типа чтение и запись из разных потоков в одну переменную без блокировки), большая часть срабатываний которых являются ложными. Пример:

```c++
void print_int(int v) { std::cout << v; }

class X {
	static int a;
public:
	X() { a = 10; }
	// ...
	void start() { std::thread(print_int, a) }
}

int main() {
	X x;
	x.start();
}
```

Такой код выдаст предупреждение как раз по паттерну, упомянутому выше.

*Разработчики библиотек для параллельного программирования зачастую выпускают **supperssion lists** для своих библиотек, с помощью которых инструмент для пиоска ошибок понимает, какие ошибки ему стоит игнорировать.*

### Инверсия приоритетов

![lec_04_1](https://i.imgur.com/bQu3qMt.png)
Выше приведен план состояний потоков A, B, C, созданных на одноядерном процессоре (расположены в порядке приоритета), и состояния блокировки общего мьютекса M. Проблема такова: на последнем сегменте вместо потока A работает поток C **с более низким приоритетом** из-за того, что поток A попытался взять примитив, который был ранее взят еще менее приоритетным потоком B (и этот временной сегмент может длиться неопределенно долго).

Решение, которое применяется в современных системах: поднимем приоритет потока B до уровня A, чтобы он добработал и отпустил примитив синхронизации.

*Как правило, это реализуется на уровне самого примитива синхронизации, и разработчику об этом думать не нужно.*

Вообще, можно придумать более озощренный пример, в котором не получится избавиться от инверсии приоритетов даже применив имеющееся решение. И вообще, даже в ОС инверсии приоритетов есть, но в очень редких случаях, и обособленное решение таких редких случаев на данный момент не является необходимым с точки зрения производительности.
