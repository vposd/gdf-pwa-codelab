# DevFest PWA Codelab - Stage 3

На этом этапе добавим кеширование динамических данных. Для хранения будем использовать IndexedDB.

[Назад в оглавление](../README.md)

## IndexedDB

IndexedDB - это способ постоянного хранения данных внутри клиентского браузера, другими словами это NOSQL хранилище на стороне клиента. Что позволяет создавать веб-приложения с богатыми возможностями обращения к данным независимо от доступности сети, ваши приложения могут работать как онлайн, так и оффлайн.

**Обычная последовательность шагов при работе с IndexedDB**

 - Открыть базу данных
 - Создать хранилище объектов в базе данных, над которой будут выполняться наши операции
 - Запустить транзакцию и выдать запрос на выполнение какой-либо операции с базой данных, например, добавление или извлечение данных
 - Ждать завершения операции, "слушая" событие DOM, на которое должен быть установлен наш обработчик
 - Сделать что-то с результатами (которые могут быть найдены в возвращаемом по нашему запросу объекте)

[Подробнее про IndexedDB](https://developers.google.com/web/ilt/pwa/working-with-indexeddb)

### Создаем хранилище

Вынесем логику создания бд и доступа к хранилищу в отдельный файл `public/scripts/store.js`:

```js
const store = {
  db: null,

  init() {
    if (store.db) { return Promise.resolve(store.db); }

    return idb.open('devfest', 1, upgradeDb => {
      if (!upgradeDb.objectStoreNames.contains('news')) {
        upgradeDb.createObjectStore('news', { keyPath: 'id' });
      }
    })
      .then(db => store.db = db);
  },

  news(mode) {
    return store.init().then(db => db.transaction('news', mode).objectStore('news'));
  }
}
```

Подключим наш скрипт и библиотеку idb-promised в `index.html`:

```html
    <script src="scripts/store.js" async></script>
    <script src="scripts/idb-promised.js" async></script>
```

[idb-promised](https://github.com/jakearchibald/idb) - библиотека, предоставляющая удобный интерфейс для работы c IndexedDB используя промисы вместо событий. Просто для удобства и наглядности.

Также не забудем добавить скрипты в список для кеширования, в сервис воркере `public/sw.js`, переменная `filesToCache`:

```js
  'scripts/store.js',
  'scripts/idb-promised.js',
```

Итак, теперь у нас есть хранилище для наших новостей.

### Сохраняем динамические данные

Добавляем тело функции `saveNewsDataLocally`. Кэп подсказывает, что функция будет сохранять наши новости локально, в IndexedDB:

```js
function saveNewsDataLocally(news) {
  return store.news('readwrite')
    .then(newsStore => {
      return Promise.all(news.map(newsItem => newsStore.put(newsItem)))
        .catch(() => {
          tx.abort();
          throw Error('News were not added to the store');
        });
    });
}
```

Помимо сохранения, необходимо еще новости и читать из IndexedDB. Добавляем тело функции `getLocalNewsData`:

```js
function getLocalNewsData() {
  return store.news('readonly').then(newsStore => newsStore.getAll());
}
```

Теперь можно модифицировать функцию для получения данных с бэкенда `loadNetworkFirst`:

```js
function loadNetworkFirst() {
  return getServerData()
    .then(news => {
      updateUI(news);
      toggleLoaderHidden(true);
      saveNewsDataLocally(news)
        .then(() => {
          setLastUpdated(new Date());
          showMessage('success', 'Данные сохранены для работы в оффлайне');
        })
        .catch(() => {
          showMessage('error', 'Чёта данные не могут быть сохранены :(');
        });
    }, error => {
      console.log('Запрос к бэкенду упал, возможно мы оффлайн...', error);
      toggleLoaderHidden(true);
      getLocalNewsData()
        .then(offlineNews => {
          if (!offlineNews.length) {
            showMessage('warning', 'Вы работаете оффлайн, cохраненных данных нет');
          } else {
            showMessage('warning', 'Вы работаете оффлайн, и просматриваете сохраненные данные от ', getLastUpdated());
            updateUI(offlineNews);
          }
        }).catch(e => console.error(e))
    });
}
```

Добавляем обработчик события `online`, чтобы данные обновились при изменении статуса на online:

```js
window.addEventListener('online', () => {
  vm.container.innerHTML = '';
  loadNetworkFirst();
});
```

## Итог

Теперь при получении данных, они отправляются в хранилище IndexedDB. Если сервер не отвечает, мы предполагаем что мы оффлайн (не важно по какой причине - упал сервер или просто нет соединения), и показываем соответствующее сообщение и сохраненные новости.

Однако если мы попытаемся добавить новость в оффлайн режиме, то она не дойдет до бэкенда, и просто потеряется. В следующем этапе добавим фоновую синхронизацию.

[Поехали дальше, добавлять фоновую синхронизацию](stage-4.md)