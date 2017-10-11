# DevFest PWA Codelab - Stage 2

На этом этапе добавим настроим простую стратегию кеширования. Результатом будет доступ с сайту оффлайн, и возможность добавление на основной экран устройства.

[Назад в оглавление](../README.md)

## Управление кешем

Гибко управлять кешированием позволяет Cache API.
Интерфейс Cache API представляет собой механизм хранения пары объектов Request / Response, которые кешируются, например, как часть жизненного цикла сервис воркера. Интерфейс Cache API доступен как в области видимости окна, так и в области видимости воркеров. Не обязательно использовать его вместе в сервис воркерами, даже если интерфейс определен в их спецификации. Однако для оффлайн доступа мы будем управлять кешированием именно в сервис воркере.

Для вызывающего скрипта может быть множество именованных объектов Cache. Разработчик сам определяет реализацию того, как скрипт (например, в  ServiceWorker) управляет обновлением Cache. Записи в Cache не будут обновлены, пока не будет выполнен явный запрос; их время жизни не истечет до момента удаления.
[Больше инфы про Cache API](https://developer.mozilla.org/ru/docs/Web/API/Cache)

### Кешируем статические ресурсы

Для начала определим, какие ресурсы мы добавим в кэш. В наш `public/sw.js` добавим массив:

```js
const filesToCache = [
  '/',
  '/sw.js',
  '/manifest.json',
  '/index.html',  
  '/scripts/app.js',
  '/images/icon.png',
  '/images/favicon-16x16.png',
  '/images/favicon-32x32.png',
  '/images/ic_add_white_24px.svg',
  '/images/icon.png',
  '/images/safari-pinned-tab.svg',
  '/styles/style.css',
  '//fonts.googleapis.com/css?family=Roboto:400,500',
  '//fonts.gstatic.com/s/roboto/v16/oMMgfZMQthOryQo9n22dcuvvDin1pK8aKteLpeZ5c0A.woff2',
  '//fonts.gstatic.com/s/roboto/v16/RxZJdnzeo3R5zSexge8UUZBw1xU1rKptJj_0jans920.woff2'
];
```

Определим название для хранилища. Сделаем название динамичным, для удобства периодического обновления кэша:

```js
const cacheName = 'cache-' + Number(new Date);
```

Меняем обработчик события `install`:

```js
self.addEventListener('install', e => {
  console.log('[SW] Install');
  e.waitUntil(
    caches.open(cacheName).then(cache => {
      console.log('[SW] Caching app shell');
      return cache.addAll(filesToCache);
    })
  );
});
```

Теперь наши ресурсы будут закешированы. Меняем обработчик события `activate` - добавим проверку и удаление старого кэша:

```js
self.addEventListener('activate', e => {
  console.log('[SW] Activate');
  e.waitUntil(
    caches.keys().then(keyList => {
      return Promise.all(keyList.map(key => {
        if (key !== cacheName) {
          console.log('[SW] Removing old cache', key);
          return caches.delete(key);
        }
      }));
    })
  );
  return self.clients.claim();
});
```

Теперь при запуске веб-страницы, сервис воркер будет кешировать необходимые ресурсы.
Добавим обработчик события `fetch` - для выдачи данных из кэша, если таковые там имеются. Иначе все предыдущие шаги бессмысленны.

```js
self.addEventListener('fetch', e => {
  const isApi = (new URL(e.request.url).pathname.match(/\/api\//g) || []).length > 0;
  if (e.request.method !== 'GET' || isApi) {
    return;
  }
  console.log('[SW] Fetch', e.request.url);  
  e.respondWith(
    caches.match(e.request).then(response => {
      return response || fetch(e.request);
    })
  );
});
```

Мы фильтруем все запросы, у которых тип не GET, а также запросы к бэкенду.

Теперь мы чистим данные сайта, перезагружаем страницу, и проходим проверку для добавления приложения на домашний экран устройства. Если мы захостим приложение на https, при заходе на него оно попросится добавиться.

## Итог

Приложение теперь доступно оффлайн, оболочка приложения закеширована.
Приложение добавляется на домашний экран устройства.
Однако при доступе оффлайн не доступен контент - это мы исправим в следующем разделе.

[Поехали дальше, кешировать динамические ресурсы](stages/stage-3.md)