title: Простой (синхронный) эмулятор REST-сервиса
date: 2015-09-24 14:48:00
tags:
   - soapui
   - groovy
   - mock
   - rest
   - service
category:
   - automation testing
---

В этой статье я покажу как можно реализовать простой, синхронный эмулятор REST-сервиса с помощью [SoapUI](http://soapui.org/). Для реализации REST-заглушки можно использовать множество других инструментов, но если у вас как и у меня уже есть куча эмуляторов SOAP-сервисов на SoapUI, то не вижу смысла плодить зоопарк :) К тому же за счет встроенного groovy, SoapUI позволяет делать достаточно гибкие эмуляторы.

<!-- more -->

## Создание проекта

>Для реализации ниже описанного примера использовался SoapUI версии 5.1.3, но пример должен так же работать и в других версиях.

Запускаем SoapUI и создаем новый REST проект. В появившемся окне ничего не меняем и жмем `ОК`.
![](/images/new_rest_project.png "New REST project")
После того как создастся проект, переименовываем его во что нибудь внятное, например `HelloWorld`. Далее добавляем MockService, который и будет выступать в качестве нашего эмулятора. Нажимаем ⌘+R в Mac OS или через контекстное меню проекта выбираем пункт `New REST MockService` и указываем его имя, например "HelloWorldService". Заходим в настройки созданного MockService и меняем параметр `Host` на `localhost` и по желанию меняем порт.

## Логирование запросов\ответов

Научим наш эмулятор логированию. Открываем в нашем MockService раздел `OnRequest Script` и пишем следующий код:
```groovy
// Получаем из запроса тип метода, параметры GET запроса (если они есть), заголовок запроса и его тело
def method = mockRequest.request.getMethod();
def uri = mockRequest.request.getUri();
def request = mockRequest.requestHeaders.toString() + "\n" + mockRequest.requestContent.toString();

log.info("Incoming ${method} request: ${uri}\n${request}");
```
Далее в разделе `AfterRequest Script` пишем:
```groovy
// Получаем заголовок ответа, тело ответа и имя операции
def action = mockResult.getMockOperation().getName();
def headers = mockResult.getResponseHeaders().toString();
def message = mockResult.getResponseContent();

log.info("Outgoing ${action} response:\n${headers}\n${message}");
```

## GET

Создадим эмуляцию ответа на GET запрос. Через контекстное меню MockService'а выбираем пункт `Add new mock action`, выбираем тип запроса GET и указываем `Resource path` - hello, т.к. наш сервис будет приветствовать пользователя по фамилии и имени из запроса.
![](/images/new_rest_mock_action.png "Add new mock action")
Далее добавляем в только что созданный action ответ (New MockResponse) и называем его `OK`. Устанавливаем status code на `200 - OK`, а content type на `application/json`. Ниже в Editor пишем следующее:
```json
{"greeting": "Hello, ${name} ${surname}!", "date": "${date}"}
```
В разделе `Script` пишем:
```groovy
// Получаем имя и фамилию из параметров запроса
def name = mockRequest.request.getParameter("name");
def surname = mockRequest.request.getParameter("surname");
// Текущая дата
def date = new Date();

// Устанавливаем параметры ответа
requestContext.name = name;
requestContext.surname = surname;
requestContext.date = date;
```

Все, наш эмулятор готов! Как запускать эмулятор, докрутить к нему https, конфиг и прочее - можно прочитать в других моих статьях.

Скачать готовый пример [HelloWorld](https://github.com/rmerkushin/soapui-synchronous-rest-service).