# DevFest PWA Codelab - Stage 1

На этом этапе добавим manifest и service worker.

[Назад в оглавление](../README.md)

## Manifest

Манифест веб-приложения предоставляет информацию о приложении (такую как имя, авторство, иконку и описание) в формате JSON-файла. Цель манифеста — установить веб-приложение на домашний экран устройства, предоставляя пользователю более быстрый доступ и больше возможностей.
Подробная [инфа тут](https://developer.mozilla.org/ru/docs/Web/%D0%9C%D0%B0%D0%BD%D0%B8%D1%84%D0%B5%D1%81%D1%82).

### Как добавить

В нашем проекте создаем файл `manifest.json`  в папке `public`.

Добавим туда скелет конфига:

```json
{
  "short_name": "",
  "name": "",
  "icons": [
    {
      "src":"",
      "sizes": "",
      "type": ""
    }
  ],
  "start_url": "",
  "background_color": "",
  "theme_color": "",
  "display": ""
}
```

> **Немного про параметры:**
Параметров может быть больше, но нам достаточно небольшого набора.
> - **short_name** - предоставляет короткое удобочитаемое имя приложения. Предназначено для использования там, где недостаточно места для отображения полного имени приложения.
> - **name** - предоставляет удобочитаемое имя для приложения, предназначенное для отображения для пользователя, например, в списке других приложений или как подпись к иконке.
> - **icons** - определяет массив объектов изображений, которые могут использованы как иконки приложения в различных контекстах. К примеру, они могут быть использованы для представления приложения среди списка других приложений или для интеграции его с переключателем задач ОС и/или настроек системы.
> - **start_url** - определяет URL, который загружается, когда пользователь запустил приложение с устройства. Если относительный URL, базовый URL будет URL манифеста.
> - **background_color** - определяет ожидаемый цвет фона для веб-приложения. Это значение повторяет то, что уже доступно в стилях приложения, но может быть использовано браузерами для отрисовки цвета фона приложения после того, как манифест станет доступен, но до того, как стили загрузятся. Это создает плавный переход между запуском приложения и загрузкой содержимого приложения.
> - **theme_color** - определяет цвет темы по умолчанию для приложения. Иногда влияет на то, как приложение отображается ОС (например, в переключателе задач Android цвет темы окружает приложение).  
> - **display** - определяет предпочтительный для разработчика режим отображения приложения.

Заполним конфиг примерно так:

```json
{
  "short_name": "DevFest PWA",
  "name": "DevFest PWA sample application for codelab",
  "icons": [
    {
      "src": "images/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "images/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "start_url": ".",
  "background_color": "#242e36",
  "theme_color": "#242e36",
  "display": "standalone"
}
```

Это только пример. Можно дать волю фантазии и наконфигурировать что-то свое. [Валидатор манифеста](https://manifest-validator.appspot.com/) в помощь.

Теперь добавим этот манифест в `index.html`:

```html
    <link rel="manifest" href="manifest.json">
```

### Полезные тэги

Добавим пару тройку полезных тегов для мобильных браузеров:

Иконка для Safari:

```html
    <link rel="apple-touch-icon" sizes="192x192" href="images/icon-192x192.png">
```

Иконка - маска для Safari:

```html
    <link rel="mask-icon" href="images/safari-pinned-tab.svg" color="#5bbad5">
```

Имя приложения:

```html
    <meta name="application-name" content="DevFest PWA">
    <meta name="apple-mobile-web-app-title" content="DevFest PWA">
```
Теги для возможности отображения в полноэкранном режиме:

```html
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
```

Тема браузера:

```html
    <meta name="theme-color" content="#242e36">
    <meta name="apple-mobile-web-app-status-bar-style" content="#242e36" />
```

## Service worker

Offline-режим, периодическая фоновая синхронизация, push-уведомления — этот функционал нативных приложений уверенно приходит в web. Сервис воркеры предоставляют для этого техническую возможность.

Service Worker — это js-скрипт, который браузер запускает в фоновом режиме, отдельно от страницы, открывая дверь для возможностей, не требующих веб-страницы или взаимодействия с пользователем. Сегодня чаще всего они выполняют такие функции как push-уведомления, кеширование и фоновая синхронизация. Ключевая их особенность — это возможность перехватывать и обрабатывать сетевые запросы, включая программное управление кешированием ответов.

Больше о service workers: [web fundamentals](https://developers.google.com/web/fundamentals/primers/service-workers/), [mdn](https://developer.mozilla.org/ru/docs/Web/API/Service_Worker_API).

**Добавление**

Добавим файл `sw.js` в папку `public`:

```js
self.addEventListener('install', e => {
  console.log('[SW] Install');
});

self.addEventListener('activate', e => {
  console.log('[SW] Activate');
});
```

Подключим его в `index.html`. Добавим в конец тега `body` код:

```html
      <script>
        if ('serviceWorker' in navigator) {
          navigator.serviceWorker.register('sw.js')
            .then(registration => console.log('Service Worker registration successful with scope: ', registration.scope))
            .catch(err => console.warn('Service Worker registration failed: ', err));
        }
      </script>
```

После обновления страницы service worker инициализируется и будет жить в отдельном потоке. Это видно в Chrome DevTools. `DevTools -> Application -> Service Worker.` Там же видно наш манифест.

## Итог

Добавили манифест и сервис-воркер. Это только основа, никаких фичей PWA пока нет.

[Поехали дальше, кешировать статические ресурсы](stage-2.md)