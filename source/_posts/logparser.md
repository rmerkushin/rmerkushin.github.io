title: LogParser
date: 2015-04-28 01:04:00
tags:
   - sql
   - sqlite
   - logs
   - python
category:
   - automation testing
---

![](/images/bicycle.jpg "Bicycle")

Вдохновившись утилитой от Microsoft - [LogParser](https://technet.microsoft.com/ru-ru/scriptcenter/dd919274), решил написать свою, кросс-платформенную утилиту для парсинга логов с выборкой данных по SQL-запросу с преферансом и куртизанками :)

<!-- more -->

На данный момент утилита поддерживает только логи от log4j, но в будущем планируется добавление новых форматов. В утилите реализована полнная поддержка SQLite SQL, регулярные выражения в запросах, вывод в plain text и .xlsx и удаленный доступ к логам по sftp.

Подробнее о работе утилиты и ее настойке можно почитать на [github](https://github.com/rmerkushin/logparser). Утилита достпна для скачивания в виде бинарных файлов для Mac OS и Windows (сборку под windows особо не проверял т.к. сборка была из под виртуальной машины).