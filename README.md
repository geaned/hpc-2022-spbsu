# hpc-2022-spbsu

Курс читал [@eugenyk](https://github.com/eugenyk)

## Содержание курса

Лекция 1:

- Введение в курс параллельного программирования
- Каким бывает параллелизм
- Межпоточное и межпроцессорное взаимодействие
- Закон Амдала

Лекция 2:

- Способы создания потока, их сравнение
- Завершение работы потока
- Механизм завершения потока в libc и Java
- Работа с примитивами синхронизации
- Классификация примитивов синхронизации: начало

Лекция 3:

- Kernel space и user space
- Класификация примитивов синхронизации: продолжение

Лекция 4:

- Классификация ошибок параллельного программирования

Лекция 5:

- Виды синхронизации
- Thread-local storage

Лекция 6:

- Поиск ошибки в многопоточном приложении
- Приближенное устройство современного процессора
- Коротко о volatile

Лекция 7:

- Барьеры памяти и модели памяти
- Проблема ABA
- Линеаризуемость
- Очередь Майкла-Скотта

Лекция 8:

- Lock-free и wait-free алгоритм снятия снэпшота

Лекция 9:

- Kernel-space RCU
- User-space RCU
- Flat combining
- Архитектура thread pool в Java

Лекция 10:

- Профилировка приложений
- LD preload
- Профилировка приложений: продолжение

Лекция 11:

- OpenMP
- TBB, аллокаторы, parallel pipeline
- Консенсус

Лекция 12:

- Асинхронный ввод-вывод
- Асинхронные серверы

Лекция 13:

- Транзакционная память
- MPI

Лекция 14:

- Организация параллельных вычислений
- Шаблон double-check
- JIT-оптимизации

OpenCL --- лекция 1:

- Обзор технологии
- Архитектура видеокарты
- Простые алгоритмы и паттерны

OpenCL --- лекция 2:
- Операция scan
- Битоническая сортировка
- Оптимизации

## Дополнительная литература

- [Страница](http://wiki.osll.ru/doku.php/courses:high_performance_computing:start) курса
- [Статья](https://storage.yandexcloud.net/lms-vault/private/1/courses/2019-spring/spb-hp-course/materials/flat-combining.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=YCAJEG-LFlOUp7t_VtjANSWBT%2F20230117%2Fru-central1-a%2Fs3%2Faws4_request&X-Amz-Date=20230117T185556Z&X-Amz-Expires=10&X-Amz-SignedHeaders=host&X-Amz-Signature=7c7782d4caea5d32c157ba9fe22944ccccc515beb980a2ba78b07fe163b80f32) о Flat combining
- [Пост](https://habr.com/ru/company/raiffeisenbank/blog/466719/) об устройстве профилировщиков
- [Публикации](https://habr.com/ru/users/khizmax/posts/) на тему lock-free структур данных от разработчика [libcds](https://github.com/khizmax/libcds)
