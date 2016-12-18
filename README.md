# Elasticsearch, Logstash, Kibana and syslog/gelf messages

**Вполне себе годная Docker конфигурация для запуска ELK стека с типом сообщений syslog или gelf**

### Ликбез

*Elasticsearch* is a search engine based on Lucene. It provides a distributed, multitenant-capable full-text search engine with an HTTP web interface and schema-free JSON documents. Elasticsearch is developed in Java and is released as open source under the terms of the Apache License. 

*Kibana* is an open source data visualization plugin for Elasticsearch. It provides visualization capabilities on top of the content indexed on an Elasticsearch cluster. Users can create bar, line and scatter plots, or pie charts and maps on top of large volumes of data.[1]

*Logstash* is a tool for managing events and logs. When used generically the term encompases a larger system of log collection, processing, storage and searching activities.

*Syslog* is a standard for message logging. It permits separation of the software that generates messages, the system that stores them, and the software that reports and analyzes them. Each message is labeled with a facility code, indicating the software type generating the message, and assigned a severity label.

*GELF* is the "Graylog Extended Log Format" is a log format that avoids the shortcomings of classic plain syslog:

### Небольшой дампа создания 

#### Curator

Хорошо бы к этой связке подключить [Curator](https://github.com/elastic/curator) для управления индексами. Надо будет погуглить.

#### Дальнейшее развитие отправки логов в *Logstash*

Плюсом юзания *syslog* считаю то, что он нативно поддерживается механизмом логирования докера.
Конечно, если использовать какой-нибудь *AWS* то все, как по волшебству, станет проще, но хочется ранать все на кастомном жележе. 
Поэтому надо выходить из ситуации с минимальным количетсвом установленных тулов на докер хосте.

Возможно логичным развитием станет установка filebeat со сборкой своего базового образа, по дефолту его включающего... Но это требует дополнительных телодвижений на хост машине.

Хотя еще докер из коробки вроде держит *Gelf*, а *Logstash* по умолчанию включает плагин input [plugin for Gelf](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-gelf.html).
Это дает возможность не использовать *rsyslog* для отправки сообщений напрямую в *Logstash*, поэтому этот вариант возможно станет даже предпочтительнее предудущего.

Поэтому рассмотрим и с отправкой в формате syslog и gelf. 

**В репо лежит конфигурация для gelf. Для syslog пример можно посмотреть тут: [Syslog Logstash config example](https://www.elastic.co/guide/en/logstash/current/config-examples.html)**

### Инструксьен по запуску всего этого хозяйства

#### Запуск связки ELK

Понадобится установленный docker-compose. После клона репозитория, конфигурацию для *logstash* смотрим в */config-dir/logstash.conf*

Далее шаги:
- необходимо склонировать репозиторий;
- зайти в директорию, глянуть на docker-compose.yml файл;
- обнаружив там переменных окружения: ELASTICSEARCH_PATH_DATA, LOGSTASH_PATH_CONFIG выполнить экспорт, например:

```
export ELASTICSEARCH_PATH_DATA="/tmp/docker/volume/elk/data"
export LOGSTASH_PATH_CONFIG="/tmp/docker/volume/elk/config-dir"
```

- выполнить запуск docker-compose из рута склонированной директории:

```
$ docker-compose up -d
```

На этом этапе можно зайти на localhost:5601 и обнаружить админку кибаны, логи надо кидать на адрес контейнера с logstash, но порт 5000, который был проброшен наружу. Чтобы посмотреть адрес контейнера logstash надо выполнить команду:

```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' elk_logstash_1
```

где *"elk_logstash_1"*, имя запущенного контейнера с logstash. Имя контейнера глянуть можно через

```
docker ps
```

И сохраним в переменной окружения адрес контейнера *logstash*:

```
export LOGSTASH_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' elk_logstash_1)
```

Теперь надо только кидать логи в формате gelf на адрес $LOGSTASH_IP:5000

#### Запуск сервиса, для инициации отправки логов в logstash

Поднимем контейнер с nginx с отправкой логов в формате gelf:

```
docker run -it --rm --name nginx --log-driver=gelf --log-opt gelf-address=udp://$LOGSTASH_IP:5000 -p 80:80 nginx
```

Топаем на localhost:5601 и смотрим появляющиеся логи например при обновлении странички по адресу: localhost:80

Дополнительные настройки конфигурации logstash можно поправить в файле /config-dir/logstash.conf

На этом все!