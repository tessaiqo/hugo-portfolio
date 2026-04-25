+++
title = "Webpack + Этап 2"
date = "2026-04-22"
description = "Отчёт по заданию"
tags = ["hugo", "theme", "web-development", "open-source"]
categories = ["web"]
featured = true
+++
---
# Этап №1
---

![luxontime](/hugo-portfolio/images/luxontime.jpg)

## 1. Введение

**Цель работы** — освоить базовую сборку frontend-проекта с использованием сборщика модулей Webpack, подключить стороннюю библиотеку Luxon для работы с датой и временем, а также упаковать готовое приложение в Docker-контейнер для обеспечения переносимости и демонстрации навыков работы с современными инструментами разработки.

В рамках работы решались следующие задачи:
- Установка Node.js и необходимых npm-пакетов.

- Создание проекта, подключение Luxon и настройка Webpack.

- Разработка интерактивных 3D-часов с отображением текущей даты и времени в формате `дд.мм.гггг чч:мм:сс`.

- Сборка проекта в единый бандл.

- Создание Docker-образа на основе Alpine Linux и запуск приложения в контейнере.

- Оформление отчёта с описанием всех шагов и приложением скриншотов.
---

## 2. Установка Node.js и инструментов разработки

Работа выполнялась на компьютере под управлением macOS. Установка Node.js была произведена с использованием пакетного менеджера **Homebrew**. Для этого в терминале были выполнены команды:

```bash
brew install node
```

После завершения установки в терминале стали доступны команды `node`, `npm` и `npx`. Проверка версии подтвердила успешную установку:
```bash
node --version   # v25.9.0 (LTS)
npm --version    # 11.12.1
```
Для удобного переключения между версиями Node.js в дальнейшем может использоваться менеджер `n` или `fnm`, однако в рамках данной работы достаточно глобально установленной LTS-версии.

## 3. Создание проекта и настройка Webpack

### 3.1 Инициализация проекта
В терминале была создана директория `webpack-luxon-demo`и выполнена команда:

```bash
npm init -y
```
Затем установлены зависимости:
- **Luxon** (основная библиотека): `npm install luxon`
- **Webpack**, **webpack-cli** и **serve** (dev-зависимости): `npm install -D webpack webpack-cli serve`

### 3.2 Проблема с ES-модулями и её решение
При первой попытке собрать проект командой `npx webpack` возникла ошибка:

```text
Module parse failed: 'import' and 'export' may appear only with 'sourceType: module'
```
Причина заключалась в том, что в файле `package.json` было указано `"type": "commonjs"`, а в исходном коде использовался синтаксис ES-модулей (`import`). Webpack по умолчанию интерпретировал `.js`-файлы как CommonJS и не распознавал `import`.

**Что такое ES-модули?**

ES-модули (ECMAScript Modules) — это официальный стандарт организации JavaScript-кода, позволяющий разбивать программу на отдельные файлы и явно указывать, какие части одного файла можно использовать в другом. В отличие от старой системы CommonJS (`require`), которая работает синхронно и характерна для Node.js, ES-модули (`import`/`export`) являются асинхронными, поддерживаются браузерами без дополнительных инструментов и позволяют сборщикам выполнять «tree shaking» — удаление неиспользуемого кода для уменьшения размера бандла.

В файле `package.json` поле `"type"` определяет, как Node.js и инструменты вроде Webpack интерпретируют файлы `.js`:

`"type": "commonjs"` — все `.js`-файлы считаются модулями CommonJS (работает `require`).

`"type": "module"`— все .js-файлы считаются ES-модулями (работает `import`).

Поскольку в проекте использовался современный синтаксис `import`, требовалось явно указать, что проект работает в режиме ES-модулей.

**Решение:**

Было принято решение перевести проект полностью на ES-модули. Для этого:

- В `package.json` значение `"type"` изменено на `"module"`.

- Создан файл конфигурации Webpack с именем `webpack.config.js`.

- Для корректной работы с `__dirname` в ES-модулях использована конструкция с `import.meta.url` и `fileURLToPath`.

Файл webpack.config.js принял следующий вид:

```javascript
import path from 'path';
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

export default {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```
После этих изменений сборка стала выполняться успешно, формируя файл `dist/main.js`.


## 4. Разработка приложения: 3D-часы с использованием Luxon

### 4.1 Общая концепция

Вместо минималистичного вывода времени, предлагавшегося в исходном задании, было решено создать более сложный и визуально привлекательный интерфейс — 3D-часы в стиле «домино». Каждая цифра времени (часы, минуты, секунды) представлена двумя кубиками, грани которых поворачиваются в соответствии с цифрой. Пользователь может вращать сцену движением мыши и приближать/отдалять колёсиком.

### 4.3 HTML и CSS (краткое описание)

В файле `index.html` размещены 12 кубиков (по два на часы, минуты, секунды — верхний и нижний ряды) и два разделителя-двоеточия. Каждый кубик представляет собой контейнер с шестью гранями, на которых с помощью элементов `<i>`отрисованы точки, соответствующие цифрам от 1 до 6.

CSS использует свойство `transform-style: preserve-3d` и функции `rotateX`/`rotateY` для вращения кубиков. Анимация осуществляется изменением data-атрибута `data-nr`, к которому привязаны CSS-правила поворота.


### 4.4 Исходный код JavaScript: от базового задания к 3D-часам

#### 4.4.1 Базовое требование задания

Исходное задание предполагало минимальную реализацию с использованием Luxon:
```javascript
import { DateTime } from 'luxon';

setInterval(() => {
  hh.textContent = DateTime
    .local()
    .setLocale('ru')
    .toFormat('dd.LL.y HH:mm:ss');
}, 1000);
```

Этот код:
- Импортирует `DateTime` из Luxon.

- Каждую секунду обновляет текстовое содержимое элемента с идентификатором `hh`.

- Выводит дату и время в формате `день.месяц.год часы:минуты:секунды`.

Требование выполнено, но визуальная часть остаётся статичной.

#### 4.4.2 Развитие в полноценное приложение

Для создания интерактивных 3D-часов код был значительно расширен, однако **ключевые элементы задания полностью сохранены**:

- Импорт Luxon.
- Использование DateTime.local().setLocale('ru').
- Ежесекундное обновление.
- Форматирование строки с датой и временем.

Ниже представлен итоговый файл `src/index.js` с подробными комментариями, поясняющими каждое изменение относительно базового примера.

```javascript
// === Импорт Luxon (точно как в задании) ===
import { DateTime } from 'luxon';

// === Получаем контейнер для 3D-сцены (добавлено для управления камерой) ===
const clockContainer = document.querySelector('.clock-container');

// === Запуск анимации (вызов сразу и установка интервала — аналог setInterval из задания) ===
animateClock();
setInterval(animateClock, 1000);

/**
 * Основная функция обновления времени.
 * Вызывается каждую секунду.
 */
function animateClock() {
    // === Получение текущего времени через Luxon (ядро задания) ===
    const now = DateTime.local().setLocale('ru');
    const hours = now.hour;
    const minutes = now.minute;
    const seconds = now.second;

    // === Обновление 3D-кубиков (дополнительная логика) ===
    setDice('h', hours);
    setDice('m', minutes);
    setDice('s', seconds);

    // === Вывод даты и времени в текстовый элемент (прямое соответствие заданию) ===
    setTime(now);
}

// --- Интерактивное управление мышью (зум и вращение) ---
// Эти обработчики добавлены для улучшения пользовательского опыта
document.body.addEventListener('wheel', (e) => {
    const dir = Math.sign(e.wheelDeltaY);
    const currentValue = clockContainer.style.getPropertyValue('--clock-tz');
    const newValue = Number(currentValue) + 100 * dir;
    clockContainer.style.setProperty('--clock-tz', cap(newValue, 0, 1000));
});

document.body.addEventListener('mousemove', (e) => {
    const degY = map(e.clientX, 0, window.innerWidth, -45, 45);
    const degX = map(e.clientY, 0, window.innerHeight, 45, -45);
    clockContainer.style.setProperty('--clock-rx', degX);
    clockContainer.style.setProperty('--clock-ry', degY);
});

document.body.addEventListener('mouseleave', () => {
    clockContainer.classList.add('smooth-leave');
    clockContainer.style.setProperty('--clock-rx', 0);
    clockContainer.style.setProperty('--clock-ry', 0);
    clockContainer.style.setProperty('--clock-tz', 0);
    setTimeout(() => clockContainer.classList.remove('smooth-leave'), 250);
});

// --- Вспомогательные функции ---

/**
 * Устанавливает значения на кубиках для часов, минут или секунд.
 * @param {string} dice - префикс ('h', 'm' или 's')
 * @param {number} val  - числовое значение (0-59)
 */
function setDice(dice, val) {
    const digit1 = Math.floor(val / 10);
    const digit1top = Math.floor(digit1 / 2);
    const digit1bottom = digit1 - digit1top;

    const digit2 = val % 10;
    const digit2top = Math.floor(digit2 / 2);
    const digit2bottom = digit2 - digit2top;

    document.querySelector(`.${dice}1t`).dataset.nr = digit1top;
    document.querySelector(`.${dice}1b`).dataset.nr = digit1bottom;
    document.querySelector(`.${dice}2t`).dataset.nr = digit2top;
    document.querySelector(`.${dice}2b`).dataset.nr = digit2bottom;
}

/**
 * Отображает дату и время в текстовом элементе.
 * Это прямое выполнение требования задания.
 * @param {DateTime} now - объект Luxon с текущим моментом времени
 */
function setTime(now) {
    const timeEl = document.querySelector('.time');
    // Формат: день.месяц.год часы:минуты:секунды (как в задании)
    timeEl.innerText = now.toFormat('dd.LL.y HH:mm:ss');
}

/**
 * Линейное отображение значения из одного диапазона в другой.
 */
function map(value, in_min, in_max, out_min, out_max) {
    value = cap(value, in_min, in_max);
    return ((value - in_min) * (out_max - out_min)) / (in_max - in_min) + out_min;
}

/**
 * Ограничение значения заданными пределами.
 */
function cap(value, min, max) {
    return Math.min(Math.max(value, min), max);
}
```


#### 4.4.3 Сравнение с базовым требованием

| Критерий | Базовое задание | Реализация в проекте |
|----------|----------------|----------------------|
| Импорт Luxon | ✅ | ✅ |
| Использование `setLocale('ru')` | ✅ | ✅ |
| Ежександное обновление (`setInterval`) | ✅ | ✅ |
| Формат `dd.Ll.y HH:mm:ss` | ✅ | ✅ |
| Вывод в DOM-элемент | ✅ (hh) | ✅ (.time) |
| Дополнительно | — | 3D-кубики, управление мышью |


Как видно из таблицы, **все обязательные пункты задания выполнены в полном объёме**. Расширение функциональности не только не нарушило исходных требований, но и позволило создать более интересный и технически сложный проект.

### 4.5 Отказ от использования Bootstrap

В задании предлагалось подключить Bootstrap для адаптивного отображения и крупного вывода данных. Однако было принято решение не использовать Bootstrap, так как реализованный 3D-дизайн является более сложным и интересным с точки зрения frontend-разработки.

## 5. Сборка и локальный запуск

После настройки Webpack и написания кода выполняется команда:
```bash
npx webpack
```
Для локального тестирования запускается статический сервер:

```bash
npx serve .
```
Приложение доступно по адресу `http://localhost:3000`. На странице отображаются 3D-кубики, показывающие текущее время, и крупная строка с полной датой и временем.

![npx-webpack](/hugo-portfolio/images/npx-webpack.jpg)


![successful-launch-webpack](/hugo-portfolio/images/successful-launch-webpack.jpg)


## 6. Контейнеризация приложения с помощью Docker

### 6.1 Теоретические основы Docker

**Docker** — это платформа для разработки, доставки и запуска приложений в изолированных окружениях, называемых **контейнерами**. Контейнеры упаковывают код, зависимости, среду выполнения и системные библиотеки в единый образ, который может быть запущен на любой системе с установленным Docker. Это решает классическую проблему «работает на моей машине» и упрощает развёртывание приложений в различных средах.

Основные понятия Docker:

- **Образ (image)** — неизменяемый шаблон, из которого создаются контейнеры. Образ состоит из слоёв, каждый из которых соответствует одной инструкции в Dockerfile.

- **Контейнер (container)** — запущенный экземпляр образа. Контейнер изолирован от хостовой системы, но может взаимодействовать с ней через проброшенные порты и тома.

- **Dockerfil**e — текстовый файл с инструкциями по сборке образа. Каждая инструкция создаёт новый слой.

- **Слой (layer)** — кешируемая единица сборки. Повторные сборки используют кеш слоёв, что значительно ускоряет процесс.

- **Реестр (registry)** — хранилище образов (например, Docker Hub). Оттуда загружаются базовые образы.

### 6.2 Разработка Dockerfile

Для проекта был создан файл Dockerfile в корневой директории. Приведём его содержимое и подробно разберём назначение каждой инструкции.

![dockerfile](/hugo-portfolio/images/dockerfile.jpg)

**Анализ инструкций**

| Инструкция | Назначение | Пояснение |
|------------|------------|------------|
| `FROM node:20-alpine` | Задаёт базовый образ. | Используется официальный образ Node.js версии 20 на базе Alpine Linux — минималистичного дистрибутива, что обеспечивает малый размер итогового образа. |
| `WORKDIR /app` | Устанавливает рабочую директорию внутри контейнера. | Все последующие команды будут выполняться относительно `/app`. |
| `COPY package*.json ./` | Копирует файлы `package.json` и `package-lock.json` в рабочую директорию. | Выполняется до установки зависимостей, чтобы кешировать слой с `npm install` и ускорить повторные сборки. |
| `RUN npm install` | Устанавливает все зависимости, включая dev-зависимости. | Выполняется внутри контейнера, создавая слой с `node_modules`. |
| `COPY . .` | Копирует все остальные файлы проекта. | Идёт после `RUN npm install`, чтобы изменения в коде не инвалидировали кеш установки зависимостей. |
| `RUN npm run build` | Запускает сборку Webpack. | Генерирует `dist/main.js` внутри контейнера. |
| `EXPOSE 3000` | Сообщает Docker, что контейнер будет слушать порт 3000. | Носит информационный характер; для фактического проброса порта используется флаг `-p`. |
| `CMD ["npm", "serve", "."]` | Команда, выполняемая при старте контейнера. | Запускает статический сервер `serve` на порту 3000. |


**Оптимизация сборки с помощью `.dockerignore`**

Чтобы не копировать в контекст сборки временные и неиспользуемые файлы, создан файл `.dockerignore`


```text
node_modules
dist
.DS_Store
```
### 6.3 Сборка Docker-образа

В терминале, находясь в корне проекта, выполнена команда:

```bash
docker build -t dice-clock .
```

Флаг `-t dice-clock задаёт` имя образа. Процесс сборки отображается в терминале. При успешном завершении выводится сообщение `naming to docker.io/library/dice-clock:latest`.

![run](/hugo-portfolio/images/run3000.jpg)


### 6.4 Запуск контейнера
Для запуска контейнера с пробросом порта используется команда:

```bash
docker run -p 3000:3000 dice-clock
```

Флаг `-p 3000:3000` связывает порт 3000 хостовой машины с портом 3000 контейнера. После запуска в терминале появляется вывод сервера `serve`. В интерфейсе Docker Desktop также виден работающий контейнер.

Приложение успешно открывается в браузере по адресу `http://localhost:3000` и функционирует идентично локальному запуску.

![dockerDesktop](/hugo-portfolio/images/dockerDesktop.jpg)
![docker-run](/hugo-portfolio/images/docker-run.jpg)


### 6.5 Обоснование выбора технологий в Docker-части

- **Alpine Linux** выбран для минимизации размера образа.

- **Сборка внутри контейнера** гарантирует единое окружение.

- `serve прост` в использовании и не требует дополнительной настройки.

- **Проброс порта** через `-p` делает приложение доступным на хосте.


## 7. Заключение

В ходе выполнения лабораторной работы были достигнуты все поставленные цели:

1. **Освоена установка** и настройка Node.js на macOS через Homebrew.

2. **Настроен Webpack** для сборки проекта с поддержкой ES-модулей.

3. **Интегрирована библиотека Luxon** — реализовано ежесекундное обновление даты и времени в заданном формате.

4. **Предоставлен оригинальный интерфейс** в виде интерактивных 3D-часов. Отказ от Bootstrap в пользу уникального дизайна, при этом все требования задания выполнены полностью.

5. **Приложение упаковано в Docker-контейнер** с использованием Alpine-образа Node.js.

6. **Подготовлены скриншоты** всех ключевых этапов: сборка Webpack, работающее приложение, сборка и запуск Docker-контейнера.

Данная работа иллюстрирует владение современными инструментами фронтенд-разработки и DevOps-практиками. Полученный проект может быть использован как элемент портфолио.



---
# Этап №2
---

## 1. Цель работы

Создать веб-страницу с использованием Bootstrap 5, в которой по нажатию на красную кнопку открывается модальное окно. В окне выводится имя выполнившего задание и текущая дата/время, формируемое библиотекой Luxon и обновляющееся каждую секунду. Страница должна иметь три колонки в соотношении 2–8–2, кнопка должна занимать всю центральную колонку, а закрытие окна возможно через крестик и отдельную кнопку «Закрыть».

## 2. Исходный код Luxon и сохранение предыдущего проекта

В отличие от Этапа 1, где был разработан сложный 3D-интерфейс, здесь используется **базовый вариант из задания**:

```javascript
import { DateTime } from 'luxon';

setInterval(() => {
  hh.textContent = DateTime
    .local()
    .setLocale('ru')
    .toFormat('dd.LL.yyyy HH:mm:ss');
}, 1000);
```

Чтобы не изменять уже готовый проект с 3D-часами и сохранить его для портфолио в первоначальном виде, **была создана новая папка** `luxon-bootstrap-modal`. Это позволило разрабатывать Этап 2 изолированно, не затрагивая предыдущую реализацию, и демонстрировать оба подхода независимо.

## 3. Инициализация проекта и установка зависимостей

В терминале, находясь в папке `luxon-bootstrap-modal`, были выполнены команды:

```bash
npm init -y
npm install luxon
npm install -D webpack webpack-cli serve
```
Это установило Luxon, а также dev-зависимости для сборки и локального сервера.

## 4. Настройка Webpack

Создан файл `webpack.config.js` с поддержкой ES-модулей, так как исходный код использует `import`. Чтобы избежать ошибки `sourceType: module`, проект переведён в режим ES-модулей через `"type": "module"` в `package.json`. Конфигурация выглядит так:

```javascript
import path from 'path';
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

export default {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

## 5. JavaScript-код с проверкой элемента

В `src/index.js` был добавлен небольшой предохранитель, чтобы скрипт не падал, если элемент `hh` ещё не загружен:

```javascript
import { DateTime } from 'luxon';

setInterval(() => {
  const hh = document.getElementById('hh');
  if (hh) {
    hh.textContent = DateTime
      .local()
      .setLocale('ru')
      .toFormat('dd.LL.yyyy HH:mm:ss');
  }
}, 1000);
```

## 6. HTML-разметка с Bootstrap 5

**Основные элементы и их классы:**

- **Колонки (2–8–2)**: реализованы сеткой Bootstrap с классами `col-2` и `col-8`.

- **Высота страницы**: класс `min-vh-100` в `<body>` гарантирует, что страница занимает весь экран, а `h-100` у контейнера и ряда растягивает колонки по высоте.

- **Центральная кнопка**: `<button>` с классами `btn btn-danger` (красная кнопка), `btn-lg` (увеличенный размер), `w-100 h-100` (растяжение на всю ширину и высоту родительской колонки), `d-flex align-items-center justify-content-center` (выравнивание текста по центру). Жирный шрифт – `fw-bold`, размер текста – `fs-5`(примерно 20px).

- **Модальное окно**: стандартное окно Bootstrap с классами `modal`, `modal-dialog`, `modal-content`. В заголовке (`modal-title`) выводится имя выполнившего задание. Тело содержит `<div id="hh">` с классами `fs-1 fw-bold` (крупный жирный шрифт). Футер содержит кнопку «Закрыть» (`btn-secondary`) и крестик в заголовке (`btn-close`), которые закрывают окно через `data-bs-dismiss="modal"`.

Итоговый `index.html`:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Luxon + Bootstrap Modal</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="min-vh-100">
    <div class="container-fluid h-100">
        <div class="row h-100 justify-content-center">
            <!-- Левая колонка -->
            <div class="col-2"></div>
            <!-- Центральная колонка с кнопкой -->
            <div class="col-8 d-flex align-items-center justify-content-center bg-white">
                <button
                    class="btn btn-danger btn-lg w-100 h-100 fw-bold fs-5 d-flex align-items-center justify-content-center"
                    data-bs-toggle="modal" data-bs-target="#timeModal">
                    Показать время
                </button>
            </div>
            <!-- Правая колонка -->
            <div class="col-2"></div>
        </div>
    </div>
    
    <!-- Модальное окно -->
    <div class="modal fade" id="timeModal" tabindex="-1" aria-labelledby="modalTitle" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="modalTitle">Выполнила: Таисия Зверева</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Закрыть"></button>
                </div>
                <div class="modal-body text-center">
                    <div id="hh" class="fs-1 fw-bold"></div>
                </div>
                <div class="modal-footer justify-content-end">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Закрыть</button>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="dist/main.js"></script>
</body>
</html>
```
![html-luxon-stage2](/hugo-portfolio/images/html-luxon-stage2.jpg)

## 7. Сборка и локальный запуск

Сборка выполнена командой:

```bash
npx webpack
```

После успешной сборки (файл `dist/main.js` создан) запущен локальный сервер:

```bash
npx serve .
```

Приложение открыто в браузере по адресу `http://localhost:3000`. При нажатии на красную кнопку «Показать время» появляется модальное окно с именем и динамической датой/временем в формате `дд.мм.гггг чч:мм:сс`. Окно закрывается крестиком или кнопкой «Закрыть».

![luxon-bootstrap-stage2](/hugo-portfolio/images/luxon-bootstrap-stage2.jpg)

## 8. Заключение

Все требования Этапа 2 выполнены. Создание отдельной папки для этого этапа позволило сохранить в неизменном виде проект с 3D-часами и продемонстрировать оба подхода в портфолио. 
