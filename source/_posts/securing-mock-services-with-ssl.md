title: Эмуляция защищенного сервиса на SoapUI с SSL
date: 2015-06-17 18:24:00
tags:
   - soapui
   - ssl
   - https
category:
   - automation testing
---

![](/images/ssl.jpg "SSL")

<!-- more -->

## Создание keystore и настройка SoapUI

Для эмуляции защищенного сервиса нам понадобиться сделать keystore c сертификатом. Для этого воспользуемся утилитой из набора JDK - `keytool`.

>keytool -genkey -alias mock_service_alias -keyalg RSA -keystore keystore_name

![](/images/keytool-gen.png "keytool")

Далее нам необходимо настроить SoapUI на созданный keystore. Для этого открываем настройки SoapUI: File->Preferences и переходим на вкладку SSL Settings. Заполняем поля по аналогии с изображением ниже:
![](/images/ssl-settings.png "keytool")
Теперь нажимаем File->Save Preferences. Находим файл конфигурации soapui-settings.xml (должен быть в домашней директории SoapUI) и кладем его рядом с проектом эмулятора.

## Запуск эмулятора

Для того чтобы эмулятор был доступен по https, нужно указать mockservicerunner.sh путь к файлу конфигурации soapui-settings.xml.

>mockservicerunner.sh -m "MockServiceName" -s path_to_soapui_settings/soapui-settings.xml path_to_project_file

Теперь эмулятор будет доступен по адресу `https://localhost:8443/`.

