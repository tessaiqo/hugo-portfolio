+++
title = "Приложение с Luxon через сборщик Vite"
date = "2026-04-25"
description = "Отчёт по заданию"
tags = ["hugo", "theme", "web-development", "open-source"]
categories = ["web"]
featured = false
+++
---
# Этап №3
---

![luxon-vite3-result](/hugo-portfolio/images/luxon-vite3-result.jpg)

## 1. Цель работы

Перенести страницу Этапа 2 с CDN-подключений на локальную сборку с помощью Vite, минимизировать использование компонентов Bootstrap путём выборочного импорта SCSS-модулей, сравнить размер итогового бандла с версией на Webpack.

## 2. Создание проекта и установка зависимостей

Создана новая папка `luxon-vite-stage3`. Инициализирован npm-проект и установлены необходимые пакеты:

```bash
npm init -y
npm install luxon bootstrap@5.3.3
npm install -D vite sass
```
В `package.json`добавлены скрипты для разработки, сборки и предпросмотра:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

Итоговый `package.json`:

```json
{
  "name": "luxon-vite-stage3",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "bootstrap": "^5.3.3",
    "luxon": "^3.7.2"
  },
  "devDependencies": {
    "sass": "^1.99.0",
    "vite": "^8.0.10"
  }
}
```

## 3. HTML-разметка (`index.html`)

Использована та же вёрстка, что и в Этапе 2: сетка 2-8-2, красная кнопка, модальное окно. Все CDN-ссылки убраны, вместо них подключается модуль `/src/main.js` с атрибутом `type="module"`.

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Vite + Bootstrap + Luxon</title>
</head>
<body class="min-vh-100">
  <div class="container-fluid h-100">
    <div class="row h-100 justify-content-center">
      <div class="col-2"></div>
      <div class="col-8 d-flex align-items-center justify-content-center bg-white">
        <button class="btn btn-danger btn-lg w-100 h-100 fw-bold fs-5 d-flex align-items-center justify-content-center"
                data-bs-toggle="modal" data-bs-target="#timeModal">
          Показать время
        </button>
      </div>
      <div class="col-2 "></div>
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

  <script type="module" src="/src/main.js"></script>
</body>
</html>
```

## 4. JavaScript-логика (`src/main.js`)

В `main.js` импортируется только необходимый JS-компонент Bootstrap – `Modal`. Это уменьшает размер JS-бандла, так как остальные компоненты (Carousel, Tooltip и пр.) исключены. Luxon подключается целиком.

```javascript
// Импорт стилей (SCSS, собранный только из нужных модулей)
import './styles.scss';

// Импорт только нужного JS-компонента Bootstrap (Modal)
import { Modal } from 'bootstrap';

// Импорт Luxon
import { DateTime } from 'luxon';

// Запуск обновления времени
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

## 5. Минимизация CSS: выборочный импорт SCSS

Вместо полного `bootstrap.min.css` (~160 КБ) создан файл `src/styles.scss`, в котором перечислены **только реально используемые модули Bootstrap**:


```scss
// Позволяет избежать лишних стилей и сократить CSS-бандл.
// 1. Основные функции и переменные
@import "bootstrap/scss/functions";
@import "bootstrap/scss/variables";
@import "bootstrap/scss/maps";
@import "bootstrap/scss/mixins";
@import "bootstrap/scss/utilities";

// 2. Базовые стили (root, reboot, type) 
@import "bootstrap/scss/root";
@import "bootstrap/scss/reboot";
@import "bootstrap/scss/type";

// 3. Сетка – колонки col-2, col-8
@import "bootstrap/scss/containers";
@import "bootstrap/scss/grid";

// 4. Кнопки – классы btn, btn-danger, btn-secondary, btn-lg
@import "bootstrap/scss/buttons";

// 5. Модальное окно (modal + close + transitions)
@import "bootstrap/scss/transitions";
@import "bootstrap/scss/modal";
@import "bootstrap/scss/close";

// 6. API утилит (генерирует классы вроде d-flex, w-100, h-100, text-center, fw-bold, fs-*)
@import "bootstrap/scss/helpers";
@import "bootstrap/scss/utilities/api";
```

## 6. Сборка проекта и размер бандла

Выполнена команда:

```bash
npm run build
```

Vite сообщил о результатах:

```text
dist/index.html                   1.71 kB │ gzip:  0.76 kB
dist/assets/index-hfONS6O4.css  116.65 kB │ gzip: 16.80 kB
dist/assets/index-50KtwJXB.js   152.00 kB │ gzip: 46.11 kB
✓ built in 781ms
```
Таким образом, несжатые размеры:

- CSS – 16.65 КБ

- JS – 152 КБ

После gzip-сжатия (которое применяется при передаче по сети) общий объём составил **16.80 КБ + 46.11 КБ = 62.91 КБ**.


## 7. Сравнение с Этапом 2 (Webpack + CDN)

Для Этапа 2 был повторно запущен `npx webpack` в папке c этапом 2. Получен следующий размер JS-бандла:

```text
asset main.js 271 KiB [emitted] (name: main)
```

Сжатие gzip этого файла дало 62 153 байта (**~60.7 КБ**).
Bootstrap на Этапе 2 загружался с CDN: CSS (~23 КБ gzip) и JS (~23 КБ gzip).

Общий gzip для Этапа 2: **60.7 КБ + 23 КБ + 23 КБ ≈ 106.7 КБ**.

**Сравнительная таблица:**
| Этап                     | Способ Bootstrap       | JS gzip (КБ) | CSS gzip (КБ) | Общий gzip (КБ) |
|--------------------------|------------------------|--------------|---------------|----------------|
| **2** (Webpack+CDN)          | CDN (3 запроса)        | 60.7         | 23.0          | **~106.7**         |
| **3** (Vite+SCSS)            | Локальный бандл        | 46.1         | 16.8          | **62.9**           |

**Вывод**: В Этапе 3 достигнуто снижение суммарного gzip-размера на **~43.8 КБ (41%)** по сравнению с Этапом 2, несмотря на то, что Bootstrap теперь включён внутрь бандла. Это стало возможным благодаря более эффективной сборке Vite, выборочному импорту SCSS-модулей и использованию только нужных компонентов JS.

___

## 8. Сравнение Webpack и Vite

| Критерий | Webpack (Этап 2) | Vite (Этап 3) |
|----------|------------------|----------------|
| **Принцип сборки** | Объединяет все модули в один бандл с помощью функциональных зависимостей, требует полной пересборки при изменении любого файла. | Использует нативные ES-модули в браузере (dev-режим), «на лету» преобразует и отдаёт только изменённые модули. |
| **Скорость запуска dev-сервера** | Медленная (секунды), зависит от размера проекта. | Мгновенная (миллисекунды), потому что не собирает весь проект при старте. |
| **Горячая замена модулей (HMR)** | Пересобирает и заменяет затронутые модули (иногда с заметной задержкой). | Моментально обновляет только изменённый модуль, сохраняя состояние приложения. |
| **Конфигурация** | Требует явного описания лоадеров и плагинов (CSS, JSX, SCSS и т.д.), файл `webpack.config.js` может быть объёмным. | Минимальный конфиг «из коробки»; поддержка TypeScript, JSX, SCSS без дополнительных настроек. |
| **Оптимизация бандла (tree-shaking)** | Зависит от используемых плагинов (например, `TerserPlugin`), требует дополнительной настройки. | Встроенный tree-shaking на основе Rollup, автоматически удаляет неиспользуемый код. |
| **Размер бандла (пример)** | JS-бандл: 271 КБ (gzip ~60.7 КБ). Bootstrap подгружается отдельно через CDN, что увеличивает общий размер. Webpack сохраняет больше «обвязки» модулей (runtime), а также включает Luxon целиком; CDN-версия Bootstrap добавляет по 23 КБ CSS и 23 КБ JS. | JS-бандл: 152 КБ (gzip 46.1 КБ), CSS: 116.65 КБ (gzip 16.8 КБ). Весь Bootstrap собран внутрь, но суммарный gzip меньше (~62.9 КБ против ~106.7 КБ). Vite лучше удаляет неиспользуемый CSS, использует современный минификатор esbuild/Rollup, а также содержит только необходимые компоненты Bootstrap. |
| **Экосистема** | Огромное количество плагинов и лоадеров, 10+ лет развития. | Быстрорастущая экосистема, многие проекты переходят на Vite из-за скорости. |

**Вывод:**
Webpack обеспечивает максимальную гибкость и контроль, но требует более сложной настройки и даёт большие временные затраты при разработке. Vite предлагает «из коробки» быстрый dev-сервер, встроенную поддержку большинства технологий и более эффективную оптимизацию бандла, что подтверждается уменьшением gzip-размера на **41%** при сохранении идентичного внешнего вида приложения.

## 9. Команда для сборки проекта

Сборка выполняется командой `npm run build`, которая определена в `package.json` и запускает `vite build`. Для локальной разработки используется `npm run dev` (мгновенная горячая замена модулей).

## 9. Публикация 

Ссылка на репозиторий с исходниками: <https://github.com/tessaiqo/luxon-vite-stage3)>
Ссылка на GitHub Pages: <https://tessaiqo.github.io/luxon-vite-stage3/>

## 10. Заключение

- Проект успешно переведён на Vite.

- Выборочный импорт SCSS-модулей Bootstrap позволил сократить CSS-бандл с ~160 КБ до ~116 КБ, а gzip – до 16.8 КБ.

- Общий gzip-размер (JS+CSS) составил **62.9 КБ**, что значительно меньше, чем у Этапа 2 (106.7 КБ), при идентичном внешнем виде страницы.

- Применение техник tree-shaking и импорта только нужных JS-компонентов подтверждает понимание принципов оптимизации фронтенд-приложений.







