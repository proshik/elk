# Elasticsearch, Logstash, Kibana and syslog messages

**Вполне себе годная Docker конфигурация для запуска ELK стека с обработкой syslog сообщений**

### Ликбез

*Elasticsearch* is a search engine based on Lucene. It provides a distributed, multitenant-capable full-text search engine with an HTTP web interface and schema-free JSON documents. Elasticsearch is developed in Java and is released as open source under the terms of the Apache License. 

*Kibana* is an open source data visualization plugin for Elasticsearch. It provides visualization capabilities on top of the content indexed on an Elasticsearch cluster. Users can create bar, line and scatter plots, or pie charts and maps on top of large volumes of data.[1]

*Logstash* is a tool for managing events and logs. When used generically the term encompases a larger system of log collection, processing, storage and searching activities.

*Syslog* is a standard for message logging. It permits separation of the software that generates messages, the system that stores them, and the software that reports and analyzes them. Each message is labeled with a facility code, indicating the software type generating the message, and assigned a severity label.

### Небольшой дампа создания 

#### Curator

Хорошо бы к этой связке подключить [Curator](https://github.com/elastic/curator) для управления индексами. Надо будет погуглить.

#### Дальнейшее развитие отправки логов в *Logstash*

Плюсом юзания *syslog* считаю то, что он по любому стоит на любом линуксе и нативно поддерживается механизмом логирования докера.
Конечно, если использовать какой-нибудь *AWS* то все, как по волшебству, станет проще, но хочется ранать все на кастомном жележе. 
Поэтому надо выходить из ситуации с минимальным количетсвом установленных тулов на докер хосте.

Возможно логичным развитием станет установка filebeat со сборкой своего базового образа, по дефолту его включающего... Но это потом.

Хотя еще докер из коробки вроде держит *Gelf*, а *Logstash* по умолчанию включает плагин input [plugin for Gelf](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-gelf.html).
Это дает возможность не использовать *rsyslog* для отправки сообщений напрямую в *Logstash*, поэтому этот вариант возможно станет даже предпочтительнее предудущего.

Да мне не нравится, когда приходится ставить или настраивать хоть что-то лишнее!

### Инструксьен по запуску всего этого хозяйства

//todo