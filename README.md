IQChannels widget
=================

## 1. Описание
Виджет позволяет интегрировать функцию мессенджера на любой вебсайт.


## 2. Принцип работы
Виджет состоит из двух частей:
* Подключаемый JavaScript с API для работы с виджетом.
* Виджет, который открывается в скрытом айфрейме поверх сайта.

На сайт устанавливается скрипт API, который после инициализации добавляет скрытый iFrame c виджетом.
API предоставляет простой интерфейс по управлению виджетом, открыть, закрыть и т.д.

Виджет всегда занимает фиксированную позицию справа и при открытии устанавливает отступ для контента всей страницы, 
так чтобы виджет не перекрывал собой никакую часть сайта.


## 3. Установка
* Предположим, IQChannels установлен и доступен по адресу http://localhost:3001/
* Пример виджета можно посмотреть по адресу http://localhost:3001/widget/example.html


### 3.1. Загрузка API виджета 
Для этого в тег HEAD добавляем:
```
<script type="text/javascript" src="http://localhost:3001/widget/widget.min.js"></script>
```


### 3.2. Инициализация виджета
Виджет может быть инициализирован как анонимный (для размещения на любом сайте без авторизации), 
либо в защищенном режиме, если он встраивается личный кабинет, защищенный паролем (интернет-банк).


#### 3.2.1. Защищенная зона (интернет-банк)
Для инициализации виджета необходимо знать название канала для подключения и авторизационные данные клиента.
Далее в любом месте тега BODY вставляем скрипт:
```html
<script type="text/javascript">
    var widget = new IQChannelsWidget({ 
        channel: 'support',
        credentials: 'some user secret',
        width: 280
    });
</script>
```

Где:
* `channel` — имя канала для подключения (тип канала в IQChannels должен быть `Веб / мобильный`).
* `credentials` — авторизационные данные для авторизации клиента в iSimpleBank.
* `width` — ширина виджета в пикселях.

После этого виджет полностью готов к работе.

**Важно!**
При вызоде клиента из защищенной зоны (разлогинивание), важно вызвать метод `logout()` у виджета, подробнее ниже.


#### 3.2.2. Анонимная версия (сайт)
Инициализация полностью идентична, за исключением:

* Параметр `credentials` необходимо оставить пустым, либо не передавать вообще.
* Тип канала в IQChannels должен быть `Веб — анонимный`.
* Вызывать метод `logout()` виджета не нужно. Анонимные клиенты имеют возможность самостоятельно разлогиниться 
  и удалить историю перепискииз их браузера.

Например:
```html
<script type="text/javascript">
    var widget = new IQChannelsWidget({ 
        channel: 'support-site',
        width: 280
    });
</script>
```

### 3.3. Настройка кнопки управления виджетом
По умолчанию кнопка открытия/закрытия виджета отображается в левом нижнем углу экрана, 
имеет фиксированную позицию и цвет.

Для настройки отображения кнопки, конструктор виджета может принимать дополнительный параметр, `iconOptions`:
```html
<script type="text/javascript">
var widget = new IQChannelsWidget({ 
    channel: 'support',
    credentials: 'some user secret',
    width: 280,
    iconOptions: {
        show: true,
        color: "#000000",
        backgroundColor: "#FFFFFF"
    }
});
</script>
```

Где:
* `show` — отображать кнопку или нет.
* `color` — цвет иконки.
* `backgroundColor` — цвет фона.


## 4. Программное управление виджетом
В случае если кнопка управления виджетом по умолчанию не устраивает, она может быть отключена 
и реализована самостоятельно.

Для этого виджет предоставляет набор методов и событий, используя которые, 
можно реализовать интерфейс управления виджетом любой сложности.
 
### 4.1. Методы
После инициализации виджет находится в скрытом состоянии. Чтобы показать его пользователю, 
необходимо вызвать метод `open()`:
```javascript
widget.open();
```

Скрыть виджет можно либо кликнув по кнопке закрыть в интерфейсе виджета, либо программно:
```javascript
widget.close();
```

Также API предоставляет метод смены состояния `toggle`, который показывает или скрывает виджет:
```javascript
widget.toggle();
```

Если виджет расположен в защищенной зоне и при инициализации использовались `credentials`, 
то при выходе из защищенной зоны необходимо в обязательном порядке вызвать метод `logout()`, 
чтобы закрыть текущую сессию пользователя и предотвратить неавторизованный доступ к переписк:
```javascript
widget.logout();
```

Уничтожение виджета, удаляет чат и кнопку:
```javascript
widget.destroy();
```

### 4.3. События
События на которые можно подписаться использую стандартный для JavasScript интерфейс `EventEmitter`:
```javascript
widget.on('close', function () {
    console.log('Виджет закрыт');
});
```

```javascript
widget.on('open', function () {
    console.log('Виджет открыт');
});
```

```javascript
widget.on('message', function () {
    console.log('Новое сообщение');
});
```
С помощью события `message` например, можно проиграть звук, если виджет скрыт, а пользователю пришло сообщение.

```javascript
widget.on('unread', function (count) {
    console.log('Количество непрочитанных сообщений', count);
});
```

С помощью события `unread` можно показывать пользователю количество непрочитанных сообщений. 
Событие возникает всякий раз, когда значение меняется. При первоначальном получении списка сообщений, 
если непрочитанных нет, то события не будет. Предполагается, что подписчик инициализирует счетчик сообщений как 0.  
