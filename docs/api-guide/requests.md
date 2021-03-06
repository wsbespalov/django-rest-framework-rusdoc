source: request.py

# Requests

> Если вы делаете web REST сервис ... вам следует игнорировать request.POST.
>
> &mdash; Malcom Tredinnick, [Django developers group][cite]

Класс `Request` REST фреймворка расширяет стандарт `HttpRequest`, добавляя поддержку для REST парсинга запроса и аутентификацию.

---

# Парсинг запроса

Request объект предоставляет гибкий парсинг запросов который позволяет вам обращать запросы с JSON или другими типами медия таким же способом как бы вы работали с данными форм.

## .data

`request.data` возвращает распарсенные данные тела запроса. Это похоже на стандартные атрибуты `request.POST` и `request.FILES` кроме этого:

* Он включает все данные, включая *file и non-file*
* Он поддерживает обработку данных HTTP методов кроме `POST`, что значит вы можете иметь доступ к данным `PUT` и `PATCH` запросов.
* Он поддерживате гибкую обработку запросов вместо простой поддержки данных форм. Для примера вы можете обработать входящий JSON таким же способом как вы обрабатываете входящие данные форм.

Подробнее смотри в [документации парсинга][parsers documentation].

## .query_params

`request.query_params` - это более корректно названый синоним для `request.GET`.

Для ясности кода мы рекомендуем использовать `request.query_params` вместо стандартного для Джанго `request.GET`. Делая так, поможет вам держать ваш код более правильным и очевидным - любой HTTP тип метода может включать запрос параметров, а не просто `GET` запрос.

## .parsers

Класс `APIView` или `@api_view` декоратор будет устанавливать это свойство автоматически, устанавливая список экземпляров `Parser` основанных на `parser_classes` установленых во view или основаных на настройках `DEFAULT_PARSER_CLASSES`.

Обычно это свойство вам не нужно.

---

**Примечание:** Если клиент отправляет неправильно сформированные данные, `request.data` может выкинуть `ParseError` исключение. По-умолчанию класс `APIView` или `@api_view` декоратор будет ловить ошибки и возвращать ответ `400 Bad Request`.

Если клиент отправляет запрос с content-type которые не может быть распарсен будет вызвано исключение `UnsupportedMediaType`, которое по умолчанию будет поймано и вернет `415 Unsupported Media Type` ответ.

---

# Соглосования данных

Запрос раскрывает некоторые свойства которые позволяют вам определить результаты на этапе взаимодействия данных. Это позволяет вам создавать поведение такое как выбор различных сериализационных схем для различных типов медия.

## .accepted_renderer

Экземпляр рендера который был выбран на этапе взаимодействия данных

## .accepted_media_type

Строка представляющая тип медия принятый на этапе взаимодействия данных.

---

# Аутентификация

REST framework предоставляет гибкую, по запросную аутентификацию, которая дает вам возможность:

* Использовать различные аутентификационные права для различных частей вашего API
* Поддержку использования множества аутентификационных прав.
* Предоставляет как пользователям так и токенам информацию связаную с входящими запросами.

## .user

`request.user` обычно возвращает экземпляр `django.contrib.auth.models.User`, хотя поведение зависит от использованной аутентификационной схемы.

Если запрос авторизован, по-умолчанию значение `request.user` - это экземпляр `django.contrib.auth.models.AnonymousUser`.

Подробности смотри в [документациия аутентификации][authentication documentation].

## .auth

`request.auth` возвращает любую дополнительный аутентификационный контекст. Точное поведение `request.auth` зависит от аутентификационных прав которые используются, но это обычно бывает экземпляр токена с которым был авторизован запрос.

Если запрос не авторизован или если нет дополнительных данных, то по-умолчанию значение `request.auth` будет `None`.

Подробности смотри в [документациия аутентификации][authentication documentation].

## .authenticators

Класс `APIView` или `@api_view` декоратор будет устанавливать это свойство автоматически, устанавливая список экземпляров `Authentication` основанных на `authentication_classes` установленых во view или основаных на настройках `DEFAULT_PARSER_CLASSES`.

Обычно это свойство вам не нужно.

---

# Улучшения

REST framework поддерживает несколько улучшений такие как формы для `PUT`, `PATCH` и `DELETE`.

## .method

`request.method` возвращает строку **uppercased** представляющую HTTP метод запроса.

Браузерные формы `PUT`, `PATCH` и `DELETE` прозрачно поддерживаются.

Подробности смотри в [документации к улучшения браузера][browser enhancements documentation].

## .content_type

`request.content_type` возвращает строку представляющую тип тела HTTP запроса, или пустую строку если никакого медия типа не было предоставлено.

Вам вероятно не понадобится прямой доступ к типу контента запроса, так как вы скорее всего будете пологаться на стандартное поведение запроса у REST Fremework.

Если вам необходим доступ к типу контента запроса, вам следует использовать свойство `.content_type` вместо `request.META.get('HTTP_CONTENT_TYPE')`, так как этот спрособ предлагает прозрачную поддержку контента не из форм на основе браузера.

Для подробной информации смотрите [документацию][browser enhancements documentation].

## .stream

`request.stream` возвращает поток представляющий содержание тела запроса.

Вам вероятно не понадобится прямой доступ к типу контента запроса, так как вы скорее всего будете пологаться на стандартное поведение запроса у REST Fremework.

Если вам необходим доступ к сырым данным напрямую, вам следует использовать свойство `.stream` вместо `request.content`, так как этот спрособ предлагает прозрачную поддержку контента не из форм на основе браузера.

Подробности смотри в [документации][browser enhancements documentation].

---

# Standard HttpRequest attributes

Так как `Request` REST Framework расширяет `HttpRequest` Django, то все другие стандартные атрибуты и методы также доступны. Для примера словари `request.META` и `request.session` доступны как обычно.

Имейте ввиду что в связи с особенностями реализации класса `Request` он не наследуется от класса `HttpRequest`, но используется композиция.


[cite]: https://groups.google.com/d/topic/django-developers/dxI4qVzrBY4/discussion
[parsers documentation]: parsers.md
[authentication documentation]: authentication.md
[browser enhancements documentation]: ../topics/browser-enhancements.md
