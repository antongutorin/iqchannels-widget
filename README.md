IQChannels Widget
=================

* [Использование](#использование)
  * [1\. Описание](#1-описание)
  * [2\. Принцип работы](#2-принцип-работы)
  * [3\. Установка](#3-установка)
    * [3\.1\. Загрузка API виджета](#31-загрузка-api-виджета)
    * [3\.2\. Инициализация виджета](#32-инициализация-виджета)
      * [3\.2\.1\. Защищенная зона (интернет\-банк)](#321-защищенная-зона-интернет-банк)
      * [3\.2\.2\. Анонимная версия (сайт)](#322-анонимная-версия-сайт)
    * [3\.3\. Настройка виджета](#33-настройка-виджета)
  * [4\. Программное управление виджетом](#4-программное-управление-виджетом)
    * [4\.1\. Методы](#41-методы)
    * [4\.3\. События](#43-события)
  * [5\. Push-уведомления](#5-push-уведомления)
  * [6\. Стилизация виджета](#6-стилизация-виджета)
* [Разработка](#разработка)

# Использование

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
        credentials: 'user-token'
    });
</script>
```

Где:
* `channel` — имя канала для подключения (тип канала в IQChannels должен быть `Веб / мобильный`).
* `credentials` — авторизационные данные для авторизации клиента в iSimpleBank.

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
        channel: 'support'
    });
</script>
```

### 3.3. Настройка виджета

Настройки виджета:
```html
<script type="text/javascript">
var widget = new IQChannelsWidget({ 
    channel: 'support',
    credentials: 'some user secret',
    width: 280,
    padBody: true,
    requireName: false,
    iconOptions: {
        show: true,
        color: "#000000",
        backgroundColor: "#FFFFFF",
        style: {}
    }
});
</script>
```

Параметры:
* `width` — ширина виджета в пикселях.
* `padBody` — автоматическое добавление паддинга к тегу `body` при открытии виджета.
* `requireName` — выключает требование представиться у анонимного клиента.
* `iconOptions` — настройки отображения кнопки виджета.
* `iconOptions.show` — отображать кнопку или нет.
* `iconOptions.style` — дополнительные CSS-стили для кнопки.
* `iconOptions.color` — цвет иконки.
* `iconOptions.backgroundColor` — цвет фона кнопки.

Для более глубокой стилизации виджета см. раздел "5. Стилизация виджета".

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

Также можно открывать виджет сразу с текстом сообщения:
```javascript
widget.open('Текст сообщения пользователя');
```

Текст сообщения также можно добавить с помощью метода `appendText()`:
```javascript
widget.appendText('Текст сообщения пользователя');
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

Управление пуш-токеном для нативных пушей через APNS (iPhone) и FCM (Android). Инструкции см. ниже.
```javascript
widget.setIPhonePushToken("my-iphone-token");
widget.setAndroidPushToken("my-android-token");
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

## 5. Push-уведомления
Виджет поддерживает нативные пуш-уведомления для мобильных телефонов при интеграции через
Ionic или React Native, но проставлять пуш-токен нужно вручную.

**Инструкция по настройке**

1. Зарегистрируйте телефон для получения пуш-уведомлений через нативное API. См. https://ionicframework.com/docs/native/push#usage

2. Получите токен через уведомление об успешной регистрации в нативном API:
```javascript
pushObject.on('registration').subscribe((registration: any) => console.log('Device registered', registration));
```

3. Перейдайте токен в виджет чата:
```javascript
// Для Айфона.
widget.setIPhoneToken(tokenString);

// Для Андроида.
widget.setAndroidToken(tokenString);
```

4. Пуш-токен будет отправлен на сервер чата, когда клиент будет авторизован.


## 6. Стилизация виджета
Стилизация виджета возможна в двух вариантах:
- Стилизация кнопки виджета.
- Стилизация окна с сообщениями.

Для стилизации окна с сообщениями требуется:
- Перейти в требуемый Проект.
- Перейти в раздел Каналы.
- Выбрать нужный канал с чатом.
- Перейти во вкладку Настройки.
- Указать кастомные стили виджета или указать ссылку на внешнюю таблицу стилей.
- Сохранить настройки.

Для стилизации иконки требуется переопределить стандарные стили кнопки с помощью CSS:
```css
/* Контейнер, в котором показывается переписка */
#iqchannels-widget-container {
  position: fixed;
  height: 100%;
  min-height: 100%;
  top: 0;
  right: 0;
  width: 0;
  z-index: 10;
  display: none;
}

/* Кнопка виджета */
#iqchannels-widget-icon {
  color: #ffffff;
  background-color: #ee5c13;
  position: fixed;
  right: 20px;
  bottom: 20px;
  width: 48px;
  height: 48px;
  border-radius: 24px;
  line-height: 48px;
  margin-right: 2px;
  margin-bottom: 2px;
}

/* Икона на кнопке виджета */
#iqchannels-widget-icon svg {
  fill: #ffffff;
  width: 24px;
  height: 24px;
  padding: 12px 12px;
}

#iqchannels-widget-icon:hover {
  width: 52px;
  height: 52px;
  border-radius: 26px;
  line-height: 52px;
  margin-right: 0;
  margin-bottom: 0;
}

#iqchannels-widget-icon:hover svg {
  width: 28px;
  height: 28px;
  padding: 12px 12px;
}

@media (max-width: 768px) {
  #iqchannels-widget {
    width: 100% !important;
  }

  #iqchannels-widget-container {
    width: 100% !important;
    height: 100%;
  }
} 
```


# Разработка

Зависимости:
- Make.
- Node 10.+.
- Npm.

Установка зависимостей:
```bash
$ make install
```

Запуск дев-версии:
```bash
$ make run
```

В браузере:
```
http://localhost:8080/example.html
http://localhost:8080/example1.html
http://localhost:8080/example2.html
...
http://localhost:8080/example10.html
```

Сборка проекта:
```bash
$ make build
```

Сборка для распространения:
```bash
$ make dist
```

Основные файлы приложения:
- `src/widget.js` кнопка виджета, которая загружается на сайте.
- `src/app` мессенджер виджета.
- `src/client` клиент для взаимодействия с сервером.
- `src/examples` примеры.
- `src/lib` стандартная библиотека.
- `src/schema` схема данных (в виджете только константы).
