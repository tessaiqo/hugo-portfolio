+++
title = "Протокол HTTP"
date = "2026-04-07"
description = "Отчёт по заданию"

tags = ["hugo", "theme", "web-development", "open-source"]
categories = ["web"]
featured = true
+++


## Цель работы

Изучить протокол HTTP на практике: научиться вручную формировать и отправлять GET и POST запросы через терминал (telnet, netcat), автоматизировать их с помощью cURL, освоить графический инструмент Insomnia и получить реальные данные от API Банка России.

---

## 1. Теоретическая справка

### Что такое HTTP?

HTTP (HyperText Transfer Protocol) — текстовый протокол прикладного уровня. Клиент (браузер, программа) отправляет запрос серверу, сервер возвращает ответ.

**Структура HTTP-запроса:**
1. **Стартовая строка** — метод, URI, версия HTTP (например, `GET /index.html HTTP/1.1`)
2. **Заголовки** — метаданные (Host, Content-Type, Content-Length и др.)
3. **Пустая строка** — разделитель (обязательные символы `\r\n\r\n`)
4. **Тело** (необязательно) — данные для POST, PUT и т.д.

**Структура HTTP-ответа:**
1. **Стартовая строка** — версия HTTP, код состояния, пояснение (`HTTP/1.1 200 OK`)
2. **Заголовки**
3. **Пустая строка**
4. **Тело** (HTML, JSON, XML, и т.д.)

### Основные методы

| Метод | Назначение | Тело | Идемпотентность |
|-------|------------|------|------------------|
| GET   | Получение данных | нет | да |
| POST  | Отправка данных на сервер | да | нет |

### Важные заголовки

- **`Host`** — **обязательный** в HTTP/1.1. Указывает, какой сайт на сервере запрашивается (виртуальный хостинг).
- **`Content-Length`** — длина тела запроса в байтах. При POST нужно точное значение, иначе ошибка `400 Bad Request`.
- **`Content-Type`** — тип данных тела (например, `application/json`, `text/xml`).
- **`User-Agent`** — идентификация клиента. Строка `Mozilla/5.0` используется для совместимости со старыми серверами.

### Коды состояния HTTP (основные)

- `200 OK` — успех
- `400 Bad Request` — ошибка в синтаксисе запроса (неверные заголовки, длина)
- `404 Not Found` — ресурс не найден
- `500 Internal Server Error` — ошибка на сервере

---

## 2. Выполнение задания 1: GET и POST через Telnet / Netcat

**Цель:** вручную написать HTTP-запрос в терминале, чтобы увидеть его текстовую структуру.

### 2.1 GET-запрос через Telnet

**Подключение к серверу:**
```bash
telnet httpbin.org 80
```
- `httpbin.org` — публичный тестовый сервер (отвечает эхом на любые запросы)
- `80` — стандартный порт HTTP


**Вручную вводим запрос** (после подключения, затем дважды нажимаем Enter):

```http
GET /get?message=hello_from_mac HTTP/1.1
Host: httpbin.org
User-Agent: Mozilla/5.0
Connection: close
```
- `GET /get?message=hello_from_mac` — метод, путь и параметр message
- `Host: httpbin.org ` — обязательный заголовок
- `User-Agent: Mozilla/5.0 ` — имитируем браузер
- `Connection: close  ` — просим сервер закрыть соединение после ответа
-  Пустая строка — конец заголовков

**Полученный ответ (успешный, 200 OK):**
```http
HTTP/1.1 200 OK
Date: Sat, 04 Apr 2026 18:31:43 GMT
Content-Type: application/json
Content-Length: 289
Connection: close

{
    "args": {
        "message": "hello_from_mac"
    },
    "headers": {
        "Host": "httpbin.org",
        "User-Agent": "Mozilla/5.0"
    },
    "origin": "5.18.170.247",
    "url": "http://httpbin.org/get?message=hello_from_mac"
}
```
**Что важно:** в поле `args` сервер вернул переданный параметр, а в `headers` — какие заголовки он получил. Это доказывает, что запрос сформирован правильно.

![get1](/hugo-portfolio/images/get1.jpg)

### 2.2 POST-запрос через Telnet

**Подключаемся:**
```bash
telnet httpbin.org 80
```

**Вводим запрос:**

```http
POST /post HTTP/1.1
Host: httpbin.org
Content-Type: application/json
Content-Length: 24

{"name":"Taya","age":20}
```

- `Content-Length: 24` — мы посчитали длину строки {"name":"Taya","age":20} (24 символа). Ошибка в длине вызовет 400 Bad Request.
- Пустая строка обязательна.
-  Тело запроса — JSON.

**Ответ сервера:**

```http
HTTP/1.1 200 OK
...
{
    "json": { "name": "Taya", "age": 20 },
    "data": "{\"name\":\"Taya\",\"age\":20}",
    "method": "POST"
}
```
- Поле `json`показывает, что сервер распарсил JSON.
- Поле `data`— сырая строка, которую мы отправили.

![post1](/hugo-portfolio/images/post1.jpg)

### 2.3 Альтернатива: Netcat (одной строкой)
Netcat позволяет отправить запрос без интерактивного режима.

**GET через netcat:**
```bash
printf "GET /get HTTP/1.1\r\nHost: httpbin.org\r\n\r\n" | nc httpbin.org 80
```
**POST через netcat:**
```bash
printf "POST /post HTTP/1.1\r\nHost: httpbin.org\r\nContent-Type: application/json\r\nContent-Length: 24\r\n\r\n{\"name\":\"Taya\",\"age\":20}\r\n" | nc httpbin.org 80
```
- `printf` корректно обрабатывает `\r\n` в macOS (в отличие от `echo`).
- Если ответ не показывается, добавьте задержку: `(printf "..."; sleep 1) | nc ...`

---

## 3. Выполнение задания 2: cURL
**Цель:** автоматизировать отправку запросов с помощью утилиты cURL (встроена в macOS).

### 3.1 GET-запрос через cURL
```bash
curl -X GET "https://httpbin.org/get?message=hello_from_mac" -H "User-Agent: Mozilla/5.0"
```
- `X GET `— метод (можно опустить, так как GET по умолчанию)
- `-H` — добавление заголовка

**Ответ:**
```json
{
    "args": { "message": "hello_from_mac" },
    "headers": { "User-Agent": "Mozilla/5.0" },
    "origin": "5.18.170.247",
    "url": "https://httpbin.org/get?message=hello_from_mac"
}
```

![get-url](/hugo-portfolio/images/geturl.jpg)

### 3.2 POST-запрос через cURL
```bash
curl -X POST "https://httpbin.org/post" -H "Content-Type: application/json" -d '{"name":"Taya","age":20}'
```
- `-d` — передаёт тело запроса. cURL автоматически вычисляет `Content-Length` и добавляет его.
- Ответ аналогичен telnet, но без служебных строк.

**Ответ:**
```json
{
    "json": { "name": "Taya", "age": 20 },
    "data": "{\"name\":\"Taya\",\"age\":20}",
    "method": "POST"
}
```
![post-url](/hugo-portfolio/images/posturl.jpg)

## 4.Выполнение задания 3: установка GUI-инструмента
**Выбранный инструмент:** Insomnia (бесплатный, удобный интерфейс).
**Установка на macOS:**
```bash
brew install --cask insomnia
```
или скачать с официального сайта [insomnia.rest](https://insomnia.rest/download "insomnia.rest").

После установки запускаем Insomnia. Создаём новую коллекцию и первый запрос.

![insomnia](/hugo-portfolio/images/insomnia.jpg)
![insomnia2](/hugo-portfolio/images/insomnia2.jpg)

## 5. Выполнение задания 4: GET-запрос к API Банка России (курс валюты за период)
**Цель:** получить курс одной выбранной валюты за выбранный период (задать любые значения), используя API Банка России.**Мой выбор:** курс доллара США с 1 по 4 апреля 2026 года.

### 5.1 Выбор API

На странице <https://www.cbr.ru/development/sxml/> описан метод XML_dynamic.asp, который позволяет получить динамику курса за период через GET-запрос с параметрами:
- `date_req1`— дата начала (ДД/ММ/ГГГГ)
- `date_req2`— дата конца
- `VAL_NM_RQ`—  код валюты (для доллара США — `R01235`)

### 5.2 Запрос в Insomnia
**Метод:**  GET
**URL:** 
```text
https://www.cbr.ru/scripts/XML_dynamic.asp?date_req1=01/04/2026&date_req2=04/04/2026&VAL_NM_RQ=R01235
```
**Тело:**  отсутствует (No Body)

**Ответ сервера (XML):** 
```xml
<ValCurs ID="R01235" DateRange1="01.04.2026" DateRange2="04.04.2026" name="Foreign Currency Market Dynamic">
    <Record Date="01.04.2026" Id="R01235">
        <Nominal>1</Nominal>
        <Value>81,2504</Value>
    </Record>
    <Record Date="02.04.2026" Id="R01235">
        <Value>80,6234</Value>
    </Record>
    <Record Date="03.04.2026" Id="R01235">
        <Value>79,7293</Value>
    </Record>
    <Record Date="04.04.2026" Id="R01235">
        <Value>79,7293</Value>
    </Record>
</ValCurs>
```
**Выбранная валюта:** USD (доллар США)
**Курс на 4 апреля 2026 года:** 79,7293 руб.

![usd](/hugo-portfolio/images/usd.jpg)

### 5.3 Примечание
GET-метод `XML_dynamic.asp` официально считается устаревшим (рекомендуется SOAP через POST), но он полностью рабочий и подходит для учебных целей, так как задание требует именно GET-запрос.

## 6. Возможные ошибки и их решения
| Ошибка                                        | Причина                                                           | Решение                                                       |
|-----------------------------------------------|-------------------------------------------------------------------|---------------------------------------------------------------|
| Нет ответа, сразу возврат в shell             | Сервер закрыл соединение                                          | Использовать `(printf "...."; sleep 1) \| nc ...`             |
| 400 Bad Request                               | Неверный Content-Length или отсутствие пустой строки после заголовков | Пересчитать длину тела, проверить два `\r\n`                 |
| Connection refused                            | Неверный порт или хост                                            | Проверить, что порт 80, имя хоста правильное                  |
| Insomnia не устанавливается через brew        | Проблемы с сетью                                                  | Скачать установщик с официального сайта                       |

## 7. Заключение и выводы
**В ходе выполнения работы я:**
- ✅ Научилась вручную формировать HTTP-запросы через telnet/netcat, понимая назначение заголовков Host, Content-Length, Content-Type.

- ✅ Освоила автоматизацию запросов с помощью cURL, который упрощает отправку и не требует ручного подсчёта байтов.

- ✅ Установила и использовала графический инструмент Insomnia, который удобен для тестирования API.

- ✅ Получила реальные данные от API Банка России — курс доллара США за выбранный период.

**Теперь я понимаю:**
- HTTP — текстовый протокол, запросы можно «писать руками».

- GET — для получения данных, POST — для отправки.

- Ошибка 400 Bad Request чаще всего связана с неверным `Content-Length` или пропущенной пустой строкой.

- GUI-инструменты экономят время при работе с API.

- API ЦБ РФ возвращает данные в формате XML; для периода можно использовать `XML_dynamic.asp` с параметрами `date_req1`, `date_req2`, `VAL_NM_RQ`.
