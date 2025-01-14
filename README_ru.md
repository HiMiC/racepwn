# RacePWN (Race Condition framework)

## 1. Описание

RacePWN - это библиотека librace а также утилита racepwn, которые предназначены для тестирования web-приложения, а также других протоколов, использующих TCP соединие на сетевую атаку типа race condition.

### 1.1. librace

**librace** - это библиотека написанная на языке программирования Си и предназначенная для создания и отправки по сети множественного количества однотипных запросов к одному хосту. Библиотека поддерживает 2 типа рейсов:

1. **Параллельный** - в данном режиме для каждого из запросов создается отдельное соединение. Запросы отправляются одновременно (с помощью использования неблокирующей записи). Опционально отправка запроса может быть разбита на 2 части. Первая - отправка основной части запроса, вторая - отправка последней части запроса, через некоторое время. Время задержки, а также размер последней части можно задавать.
2. **Последовательный** - в данном режиме запросы склеиваются в один большой запрос, который отправляется через одно соединение.

### 1.2. racepwn

**racepwn** - утилита, написанная на golang и имплементирующая интерфейс работы с librace, с помощью задания параметров через конфиг, написанный на json.

#### Пример файла конфигурации

```json
[
    {
        "race": {
            // задание параметров работы
            "type": "paralell", // режим работы
            "delay_time_usec": 10000, // время задержки перед отправкой последней части данных
            "last_chunk_size": 10 // размер последней части данных
        },
        "raw": {
            "host": "tcp://localhost:8080", // имя и порт хоста к кому соединяемся
            "ssl": false, // флаг использования ssl
            "race_param": [
                {
                    // параметры тела запроса
                    "data": "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n", // данные запроса
                    "count": 100 // количество пакетов
                }
            ]
        }
    }
]
```

Поддерживаются 2 режима работы. Запуск как консольная утилита, а также как веб-сервис.

#### **Консольная утилита:**

В данном режиме работы на вход приложения подается файл конфигурации. Приложение выполняет запросы к серверу, а потом прекращает работу.
Пример запуска:

```
racepwn < config.json
```

#### **Веб-сервис**:

В данном режиме утилита начинает работу в качестве веб-сервиса. Для работы необходимо сделать `POST-запрос` по пути `/race` с содержимым файла конфигурации.
Имя хоста и порт должны быть заданы с помощью ключа `-hostname`
Пример запуска:

```
racepwn -hostname "localhost:8080"
```

Пример
```
curl -i -X POST localhost:8080/race --data @<(cat config.json)
```

## 2. Установка

### 2.1. Зависимости

-   clang или gcc
-   golang
-   libevent (libevent-dev)
-   openssl (libssl-dev)
-   cmake

### 2.2. Cборка

Для запуска процесса сборки необходимо выполнить `./build.sh`.

_Внимание_ : сборка тестировалась только для linux.

## 3. Что нужно еще сделать

-   Поддержка HTTP2.
-   Парсинг HTTP ответов.
-   Написание модуля для python.
