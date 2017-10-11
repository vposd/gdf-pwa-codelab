# DevFest PWA Codelab - Introduction

### Что сделаем:

Из существующего проекта сделаем свое PWA с оффлайном и добавлением на homescreen

### Что узнаем:

 - Что такое manifest, зачем он нужен и как его запилить
 - Как добавить service worker
 - Как симулировать оффлайн режим и дебажить service worker в Chrome DevTools
 - Как реализовать простую стратегию кеширования
 - Как реализовать фоновую синхронизацию данных и обновлять данные, даже когда приложение закрыто

Что нам понадобится:

 - Базовые знания HTML, CSS, JS
 - Браузер Google Chrome - [актуальная версия](https://www.google.com/chrome/browser/desktop/index.html)
 - Любимый редактор кода
 - [Node.js](https://nodejs.org/en/) и npm (можно yarn)

## Проект

Если установлен git, то стягиваем исходный код проекта

    git clone https://github.com/vpozdnyakoff/dev-fest-pwa

Если git не установлен, просто качаем zip-архив с кодом. [Скачать zip](https://github.com/vpozdnyakoff/dev-fest-pwa/archive/master.zip).
Устанавливаем зависимости:

    npm install

Запуск приложения:

    npm start

Проект - небольшой сайт с контентом. В нашем случае - новости. Можно смотреть их и добавлять.

## Этапы

 - [Добавление манифеста и сервис-воркера](stages/stage-1.md)
 - [Кеширование статических ресурсов](stages/stage-2.md)
 - [Кеширование динамических ресурсов](stages/stage-3.md)
 - [Фоновая синхронизация](stages/stage-4.md)

 