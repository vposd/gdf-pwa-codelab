# DevFest PWA Codelab - Stage 4

На этом этапе добавим фоновую синхронизацию.

[Назад в оглавление](../README.md)

## Background Sync

Background Sync - это новый веб-API, который позволяет отложить действия до тех пор, пока у пользователя не будет стабильного соединения. Это полезно, когда нужно отправить данные, но соединения нет.

**Схема работы**

 - Создадим хранилище `outbox` в IndexedDB - там будут храниться новости, которые ожидают отправки на бэкенд.
 - При добавлении данных кидаем данные в хранилище outbox и инициализируем событие `sync` для сервис воркера.
 - В сервис воркере в обработчике события `sync`, получаем данные с хранилища `outbox`, формируем запросы к бэкенду и пробуем отправить. Если отправка удалась - удаляем отправленные данные из `outbox`.
 - ...
 - PROFIT

[Подробнее про BackgroundSync](https://developers.google.com/web/updates/2015/12/background-sync)

### Модифицируем наше хранилище

Добавим инициализацию и доступ к хранилищу `outbox`. Для этого немного модифицируем файл `public/store.js`:

```js
const store = {
  db: null,

  init() {
    if (store.db) { return Promise.resolve(store.db); }

    return idb.open('devfest', 1, upgradeDb => {
      if (!upgradeDb.objectStoreNames.contains('news')) {
        upgradeDb.createObjectStore('news', { keyPath: 'id' });
      }
      if (!upgradeDb.objectStoreNames.contains('outbox')) {
        upgradeDb.createObjectStore('outbox', { keyPath: 'id' });
      }
    })
      .then(db => store.db = db);
  },

  outbox(mode) {
    return store.init().then(db => db.transaction('outbox', mode).objectStore('outbox'));
  },

  news(mode) {
    return store.init().then(db => db.transaction('news', mode).objectStore('news'));
  }
}
```

### Меняем схему отправки данных

Модифицируем функцию `addServerData` в файле `public/app.js`. Вместо отправки запроса на бэкенд, добавляем данные в outbox хранилище, инициализируем событие синхронизации для сервис воркера:

```js
function addServerData() {
  const jsonDate = new Date();
  jsonDate.setHours(jsonDate.getHours() - jsonDate.getTimezoneOffset() / 60);

  const data = {
    id: jsonDate.valueOf(),
    publishedAt: jsonDate.toJSON(),
  };
  const inputIds = ['author', 'title', 'description', 'url'];  
  inputIds.forEach(id => Object.assign(data, { [id]: document.getElementById(id).value }));
  inputIds.forEach(id => document.getElementById(id).value = '');

  updateUI([data]);
  saveNewsDataLocally([data]);
  toggleDialogVisible(false);

  store.outbox('readwrite')
    .then(outbox => outbox.put(data))
    .then(() => navigator.serviceWorker.ready)
    .then(registration => registration.sync.register('outbox'));
}
```

Добавляем отправку события sync при получении данных:

```js
function loadNetworkFirst() {
  <...>
  .then(() => navigator.serviceWorker.ready)
  .then(registration => registration.sync.register('outbox'));
}
```

В сервис воркере добавляем обработчик события `sync`. В нем мы получаем данные из хранилища outbox, и отправляем данные на бэкэнд. Если запрос выполнился успешно, удаляем соответствующие данные из хранилища:

```js
self.addEventListener('sync', e => {
  if (e.tag === 'outbox') {
    store.outbox('readonly')
      .then(outboxStore => outboxStore.getAll())
      .then(newsItems => {
        return Promise.all(
          newsItems.map(newsItem => {
            const headers = new Headers({ 'Content-Type': 'application/json' });        
            const body = JSON.stringify(newsItem);
            return fetch('/api/add', { method: 'POST', headers, body })
              .then(response => response.json())
              .then(data => {
                if (data.success) {
                  store.outbox('readwrite').then(outboxStore => outboxStore.delete(newsItem.id));
                }
              })
          })
        );
      })
    .catch(e => console.error(e));
  }
});
```

Чтобы наше хранилище `store` было доступно в сервис воркере, импортируем необходимые скрипты:

```js
importScripts('scripts/idb-promised.js');
importScripts('scripts/store.js');
```

## Итог

Теперь при недоступности бэкэнда, данные не потеряются. Если соединение вернется, при попытке отправки новой новости либо при получении данных, отправятся все неотправленные.
Данные отправятся при получении соединения, даже если вкладка закрыта.