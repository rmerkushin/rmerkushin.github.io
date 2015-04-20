title: Простой (синхронный) эмулятор SOAP-сервиса
date: 2015-04-17 15:53:00
tags:
   - soapui
   - groovy
   - mock
   - soap
   - service
category:
   - automation testing
---

В этой статье я покажу как можно реализовать простой, синхронный эмулятор SOAP-сервиса с помощью [SoapUI](http://soapui.org/). Для реализации эмулятора я использовал пример WSDL с сайта [tutorialspoint](http://www.tutorialspoint.com/wsdl/wsdl_example.htm). Эмулятор будет поддерживать разные варианты ответов, логировать запросы и ответы, а так же настраиваться при помощи конфигурационного файла. Let's rock!

<!-- more -->

## Создание проекта

>Для реализации ниже описанного примера использовался SoapUI версии 5, но пример должен так же работать и в версии 4х.

После запуска SoapUI жмем ⌘+N в Mac OS или Ctrl+N в Windows\Linux, либо через меню `File->New SOAP project` создаем новый SOAP проект. Вводим `Project Name`, у нас это будет "HelloWorld" и выбираем нужную WSDL (вместо файла можно указать ссылку на уже готовый сервис). Выбираем пункт `Create Requests` если он не отмечен, это позволит автоматически создать примеры запросов по указанной WSDL для тестирования и отладки эмулятора.
![](/images/new_soap_project.png "New SOAP project")
После того как создастся проект, необходимо будет добавить MockService, который и будет выступать в качестве нашего эмулятора. Нажимаем ⌘+O в Mac OS или через контекстное меню проекта выбираем пункт `New SOAP MockService` и указываем его имя, например "HelloWorldService".

>В SoapUI версии 4х это можно сделать автоматически при создании проекта выбрав соответствующий пункт.

Теперь нам нужно добавить операции которые будет поддерживать эмулятор. Для этого выбираем наш только что созданный MockService и жмем ⌘+N в Mac OS или Ctrl+N в Windows\Linux, либо через контекстное меню выбираем пункт `New MockOperation`. В открывшемся окне выбираем нужную нам операцию. В этом примере WSDL всего одна операция, по этому просто жмем `OK` (в других случаях этот шаг нужно будет повторять столько раз, сколько нужно эмулировать операций). Если в дереве проекта открыть `Response 1` то можно увидеть сгенерированный SOAP ответ по указанной при создании проекта WSDL.
![](/images/operation_response.png "Operation response")
И так, основа нашего эмулятора уже готова, и если его запустить, то он будет вполне честно работать и отвечать всегда одним и тем же сообщением. Но это слишком скучно, по этому давайте немного "прокачаем" наш эмулятор.

## Динамический ответ
Мы будем получать имя из присланного запроса и отвечать приветствием с использованием этого имени. Для этого нам потребуется написать небольшой скрипт на [groovy](http://www.soapui.org/apidocs/index.html). SoapUI позволяет использовать groovy скрипты на разных уровнях mock'а, таких как сам MockService (при старте сервиса, при его остановке, при получении запроса и после запроса), MockOperation и MockResponse. Нас интересует уровень MockResponse. Для начала дадим нашему ответу какое-нибудь логичное название, например "successful", поскольку наш ответ будет эмулировать удачную обработку запроса. Теперь открываем наш ответ, и в элемент `greeting` вписываем  переменную `${greeting}` которая будет заменяться на нужный текст при выполнении скрипта.
```xml
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                  xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:urn="urn:examples:helloservice">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:sayHelloResponse soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <greeting xsi:type="xsd:string">${greeting}</greeting>
      </urn:sayHelloResponse>
   </soapenv:Body>
</soapenv:Envelope>
```
Теперь, в этом же окне открываем вкладку `Script` и вписываем туда следующий код:
```groovy
import com.eviware.soapui.support.GroovyUtils;

// Создаем экземпляр класса GroovyUtils
def groovyUtils = new GroovyUtils(context);

// Получаем значение элемента firstName из входящего запроса по xpath
def requestHolder = groovyUtils.getXmlHolder(mockRequest.requestContent);
def firstName = requestHolder.getNodeValue("//firstName");

// Устанавливаем значение переменной greeting
def greeting = "Привет, ${firstName}!";
requestContext.greeting = greeting;
```
И вуаля! Теперь наш эмулятор будет приветствовать нас по имени отправляемом в запросе. Чтобы убедиться в том, что эмулятор работает, откроем в дереве проекта элемент `HelloWorldService`. Перед тем как запустить эмулятор, нам нужно его настроить. Нажимаем на иконку в виде перекрещенных отвертки и гаечного ключа. Изменяем значение параметра `Host` на localhost и если у вас занят порт 8080, то и его тоже. Закрываем окно настроек эмулятора и жмем на иконку в виде зеленой стрелки.
![](/images/hello_world_service.png "HelloWorldService")
Теперь открываем в дереве проекта элемент `Request 1` для операции `sayHello`, изменяем значение элемента `firstName` на свое имя, меняем в адресной строке URL на [http://localhost:8080/]() и жмем иконку в виде зеленой стрелки. В правой части окна должен появится ответ с приветствием.
![](/images/first_response.png "Response")
Если вы не видите сообщения с приветствием, значит вы где-то ошиблись.

## Логирование входящих и исходящих сообщений
Чтобы проверить что приходило в наш эмулятор и что отправлялось в ответ, мы прикрутим к нашему эмулятору логер сообщений, который будет сохранять действия нашего эмулятора в файл лога SoapUI - `global-groovy.log`, а так же сохранять запросы в виде xml файлов. В дереве проекта открываем MockOperation - `sayHello`, из выпадающего списка `Dispatch` выбираем пункт `SCRIPT`. В появившемся текстовом поле ниже вписываем следующий код:
```groovy
import com.eviware.soapui.support.GroovyUtils;

// Создаем экземпляр класса GroovyUtils
def groovyUtils = new GroovyUtils(context);
// Получаем путь до директории с проектом
def projectDir = groovyUtils.projectPath;

// Получаем запрос и значение элемента firstName из входящего запроса по xpath
def requestHolder = groovyUtils.getXmlHolder(mockRequest.requestContent);
def requestMessage = requestHolder.getPrettyXml();
def firstName = requestHolder.getNodeValue("//firstName");

// Сохраняем запрос в директорию requests рядом с нашим проектом (директория должна существовать!)
def request = new File(projectDir, "/requests/sayHello_${firstName}.xml");
request.write(requestMessage, 'UTF-8');

// Сохраняем запрос в лог global-groovy.log
log.info("Received sayHello request: \n${requestMessage}");
```
Так же добавляем в конец скрипта для ответа "successful" следующий код:
```groovy
// Сохраняем ответ в лог global-groovy.log
def message = mockResponse.responseContent.replace('${greeting}', greeting);
log.info("Sended sayHello response: \n${message}");
```
Теперь если запустить эмулятор и отправлять в него запросы, то можно увидеть что все пришедшие запросы сохраняются в директории requests, а так же их можно увидеть в логе SoapUI. Для того чтобы посмотреть лог SoapUI, нужно открыть вкладку `script log` на нижней панели главного окна SoapUI.

## Конфигурирование эмулятора
Для конфигурирования эмулятора мы будем использовать старый добрый INI(conf) формат, т.к. он лучше дружит с кириллицей чем формат JAVA properties, а так же хорошо поддерживается другими ЯП на случай автоматической конфигурации эмулятора. Так же я покажу как подключать сторонние JAVA-библиотеки к SoapUI. Для работы с INI форматом я буду использовать [ini4j](http://ini4j.sourceforge.net/) библиотеку. Чтобы подключить JAVA-библиотеку к SoapUI ее необходимо положить в директорию `../SoapUI/bin/ext/`. Теперь создадим наш конфигурационный файл, settings.conf:
```ini
[Response]
; Возможные варианты: successful, custom, fault
type = successful
timeout = 0
greeting = Hey
custom = example
```
И так, у нашего эмулятора будет конфигурироваться вариант ответа, таймаут перед ответом на запрос и текст приветствия. Создадим новый вариант ответа, fault. В дереве проекта выбираем MockOperation `sayHello` и жмем ⌘+N в Mac OS или Ctrl+N в Windows\Linux, либо через контекстное меню выбираем пункт `New MockResponse`. Вводим имя нового варианта ответа - "fault". Далее открываем его и жмем на иконку в виде квадрата с восклицательным знаком внутри и подтверждаем создание fault сообщения и заполняем его например так:
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
   <soapenv:Body>
      <soapenv:Fault>
         <faultcode>SOAP-ENV:Client</faultcode>
         <faultstring xml:lang="EN">There was an error in the incoming SOAP request packet:  Client, InvalidXml</faultstring>
         <faultactor>http://example.org/HelloWorldService</faultactor>
      </soapenv:Fault>
   </soapenv:Body>
</soapenv:Envelope>
```
Теперь открываем наш скрипт для MockOperation - `sayHello` и приводим к такому виду:
```groovy
import org.ini4j.Ini;
import com.eviware.soapui.support.GroovyUtils;

// Создаем экземпляр класса GroovyUtils
def groovyUtils = new GroovyUtils(context);
// Получаем путь до директории с проектом
def projectDir = groovyUtils.projectPath;
// Загружаем конфигурационный файл
def conf = new Ini(new File(projectDir, "/settings.conf"));

// Получаем сообщение и значение элемента firstName из входящего запроса по xpath
def requestHolder = groovyUtils.getXmlHolder(mockRequest.requestContent);
def requestMessage = requestHolder.getPrettyXml();
def firstName = requestHolder.getNodeValue("//firstName");

// Сохраняем запрос в директорию requests рядом с нашим проектом (директория должна существовать!)
def request = new File(projectDir, "/requests/sayHello_${firstName}.xml");
request.write(requestMessage, 'UTF-8');

// Сохраняем запрос в лог global-groovy.log
log.info("Received sayHello request: \n${requestMessage}");

// Задержка перед ответом
def timeout = conf.get("Response", "timeout");
Thread.sleep(Integer.valueOf(timeout) * 1000);
log.info("Wait ${timeout} second(s) before sending response");

// Выбираем вариант ответа
def response = conf.get("Response", "type");
return response;
```
Так же изменим скрипт для ответа "successful":
```groovy
import org.ini4j.Ini;
import com.eviware.soapui.support.GroovyUtils;

// Создаем экземпляр класса GroovyUtils
def groovyUtils = new GroovyUtils(context);
// Получаем путь до директории с проектом
def projectDir = groovyUtils.projectPath;
// Загружаем конфигурационный файл
def conf = new Ini(new File(projectDir, "/settings.conf"));

// Получаем значение элемента firstName из входящего запроса по xpath
def requestHolder = groovyUtils.getXmlHolder(mockRequest.requestContent);
def firstName = requestHolder.getNodeValue("//firstName");

// Устанавливаем значение переменной greeting
def helloText = conf.get("Response", "greeting");
def greeting = "${helloText}, ${firstName}!";
requestContext.greeting = greeting;

// Сохраняем ответ в лог global-groovy.log
def message = mockResponse.responseContent.replace('${greeting}', greeting);
log.info("Sended sayHello response: \n${message}");
```

## Custom'ный ответ
Добавим к нашему эмулятору возможность загружать ответ из заранее подготовленного xml файла. Добавим новый MockResponse по аналогии с fault ответом и назовем его "custom". Далее заменим тело сообщения на переменную `${message}` и добавим следующий скрипт:
```groovy
import org.ini4j.Ini;
import com.eviware.soapui.support.GroovyUtils;

// Создаем экземпляр класса GroovyUtils
def groovyUtils = new GroovyUtils(context);
// Получаем путь до директории с проектом
def projectDir = groovyUtils.projectPath;
// Загружаем конфигурационный файл
def conf = new Ini(new File(projectDir, "/settings.conf"));

// Загружаем ответ из файла
def responseFile = conf.get("Response", "custom");
def xmlString = new File(projectDir, "/responses/${responseFile}.xml").getText("UTF-8");
def responseHolder = groovyUtils.getXmlHolder(xmlString);
def message = responseHolder.getPrettyXml();

// Устанавливаем значение переменной message
requestContext.message = message;

// Сохраняем ответ в лог global-groovy.log
log.info("Sended sayHello response: \n${message}");
```
Все, наш эмулятор готов!

## Скрипт для запуска эмулятора
Чтобы не держать эмулятор у себя на компьютере и не запускать SoapUI каждый раз, напишем небольшой скрипт для headless запуска и выключения эмулятора для Linux.
Start service:
```bash
DIR=$(cd $(dirname $0) && pwd)
SERVICE_RUNNER=/<path_to_soapui>/bin/mockservicerunner.sh
PROJECT_FILE=/<path_to_project_file>/HelloWorld.xml
PORT=<PORT>
LOCAL_PATH=/HelloWorldService

OUTPUT=$DIR/soapui.log
ERROR=$DIR/soapui-errors.log
PIDFILE=$DIR/service.pid

$SERVICE_RUNNER -m "HelloWorldService" -p $PORT -a $LOCAL_PATH $PROJECT_FILE >> $OUTPUT 2>> $ERROR &

echo $! > $PIDFILE
```
Stop service:
```bash
DIR=$(cd $(dirname $0) && pwd)

cat $DIR/service.pid | xargs kill -9
kill -9 `ps aux | grep HelloWorld | awk '{print $2}'`

cat /dev/null > $DIR/soapui-errors.log
cat /dev/null > $DIR/soapui.log
cat /dev/null > $DIR/global-groovy.log

rm -f $DIR/service.pid
```
После запуска, эмултор будет доступен по URL: `http://<ip_or_host_name>:<PORT>/<LOCAL_PATH>`

Скачать готовый пример [HelloWorld](https://github.com/rmerkushin/soapui-synchronous-service).