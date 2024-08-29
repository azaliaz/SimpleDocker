## Part 1. Готовый докер


##### Взять официальный докер образ с **nginx** и выкачать его при помощи `docker pull`
![docker pull](./image/1.png)
##### Проверяем наличие докер образа через `docker images`
![docker images](./image/2.png)
##### Запустить докер образ через `docker run -d [image_id|repository]`
![docker run -d nginx](./image/3.png)
##### Проверить, что образ запустился через `docker ps`
![docker ps](./image/4.png)
##### Посмотреть информацию о контейнере через `docker inspect [container_id|container_name]`
- Полученная информация:
![docker inspect distracted_franklin](./image/5.png)
##### По выводу команды определить и поместить в отчёт размер контейнера, список замапленных портов и ip контейнера
- размер контейнера:
![docker inspect distracted_franklin | grep "Size"](./image/6.png)

- список замапленных портов:
![docker inspect --format '{{json .NetworkSettings.Ports}}' distracted_franklin | jq](./image/7.png)

- ip контейнера:
![docker inspect --format '{{ .NetworkSettings.IPAddress }}' distracted_franklin](./image/8.png)
##### Остановить докер образ через `docker stop [container_id|container_name]`

- Вывод команды `docker stop distracted_franklin`
![docker stop distracted_franklin](./image/9.png)

##### Проверить, что образ остановился через `docker ps`

- Вывод команды `docker ps`
![docker ps](./image/10.png)
##### Запускаем докер с портами 80 и 443 в контейнере, замапленными на такие же порты на локальной машине, через команду *run* 
- Вывод команды `docker run -d -p 80:80 -p 443:443 nginx`:
![docker run -d -p 80:80 -p 443:443 nginx](./image/11.png)
##### Проверяем, что в браузере по адресу *localhost:80* доступна стартовая страница **nginx**
![страница в браузере](./image/12.png)
##### Перезапустить докер контейнер через `docker restart [container_id|container_name]`
![docker restart](./image/13.png)
##### Проверяем любым способом, что контейнер запустился
- Вывод команды `docker ps`:
![docker ps](./image/14.png)

## Part 2. Операции с контейнером

##### Прочитать конфигурационный файл *nginx.conf* внутри докер контейнера через команду *exec*
- Вывод команды `docker exec distracted_franklin cat /etc/nginx/nginx.conf`
![.conf](./image/15.png)
##### Создать на локальной машине файл *nginx.conf* и настроить в нем по пути */status* отдачу страницы статуса сервера **nginx**
- Создаем с помощью команды `touch nginx.conf` и прописываем в нем порт 80, также location = /status {stub_status on}; для того, чтобы появилась страница по адресу localhost/status с отображением статуса
[nginx.conf](./image/16.png)
##### Скопировать созданный файл *nginx.conf* внутрь докер образа через команду `docker cp`
![copy nginx.conf](./image/17.png)
##### Перезапустить **nginx** внутри докер образа через команду *exec*
- Вывод команды `docker exec distracted_franklin nginx -s reload`
![exec](./image/18.png)
##### Проверить, что по адресу *localhost:80/status* отдается страничка со статусом сервера **nginx**
![localhost/status](./image/19.png)
##### Экспортировать контейнер в файл *container.tar* через команду *export* и остановить контейнер
![export](./image/20.png)
##### Удалить образ через `docker rmi [image_id|repository]`, не удаляя перед этим контейнеры
![docker rmi](./image/21.png)
##### Удалить остановленный контейнер
![docker rm](./image/22.png)
##### Импортировать контейнер обратно через команду *import*
- вывод команды `sudo docker import -c 'cmd ["nginx", "-g", "daemon off;"]' -c 'ENTRYPOINT ["/docker-entrypoint.sh"]' container.tar import_nginx`:
![docker rm](./image/23.png)
##### Запустить импортированный контейнер
![docker run](./image/24.png)
##### Проверить, что по адресу *localhost:80/status* отдается страничка со статусом сервера **nginx**
![localhost:80/status](./image/25.png)


## Part 3. Мини веб-сервер

##### Написать мини сервер на **C** и **FastCgi**, который будет возвращать простейшую страничку с надписью `Hello World!`
##### Для этого необходимо создать .c файл, в котором будет описана логика сервера (в нашем случае - вывод сообщения Hello World!), а также конфиг nginx.conf, который будет проксировать все запросы с порта 81 на порт 127.0.0.1:8080
![server.c](./image/28.png)
![nginx.conf](./image/26.png)
##### Теперь выкачаем новый docker-образ и на его основе запустим новый контейнер `docker run -d -p 81:81 nginx`
![docker run -d -p 81:81 nginx](./image/27.png)
##### Скопировать созданные файлы `docker cp nginx.conf distracted_taussig:/etc/nginx/` `docker cp server.c distracted_taussig:/home`:
![cp](./image/29.png)
##### Для того чтобы запустить написанный мини сервер, необходимо установить требуемые утилиты для запуска мини веб-сервера на FastCGI, в частности spawn-fcgi и libfcgi-dev `apt-get install -y spawn-fcgi libfcgi-dev`
![apt-get install -y spawn-fcgi libfcgi-dev](./image/30.png)
##### скомпилируем и запустим наш мини веб-сервер через команду spawn-fcgi на порту 8080
![spawn-fcgi -p 8080 ./server](./image/31.png)
![nginx -s reload](./image/32.png)

##### Проверить, что в браузере по *localhost:81* отдается написанная вами страничка
![localhost:81](./image/33.png)

## Part 4. Свой докер

#### Написать свой докер образ, который:
##### 1) собирает исходники мини сервера на FastCgi из [Части 3] 
##### 2) запускает его на 8080 порту
##### 3) копирует внутрь образа написанный *./nginx/nginx.conf*
##### 4) запускает **nginx**.

![dockerfile](./image/37.png)
![run.sh](./image/34.png)
![server.c](./image/35.png)

##### Собрать написанный докер образ через `docker build` при этом указав имя и тег
- Вывод команды `docker build -t distracted_taussig:1.0 .`
![docker build](./image/36.png)

##### Проверить через `docker images`, что все собралось корректно
![docker images](./image/38.png)
##### Запустить собранный докер образ с маппингом 81 порта на 80 на локальной машине и маппингом папки *./nginx* внутрь контейнера по адресу, где лежат конфигурационные файлы **nginx**'а `docker run -it -p 80:81 -v /Users/azaliagm/DO5_SimpleDocker-1/src/part4/nginx.conf:/etc/nginx/nginx.conf -d distracted_taussig:1.0 bash`
![docker](./image/39.png)

##### Проверим, что по localhost:80 доступна страничка написанного мини сервера
![localhost:80](./image/40.png)
##### Дописать в *./nginx/nginx.conf* проксирование странички */status*, по которой надо отдавать статус сервера **nginx**
![/status](./image/42.png)
##### Перезапустить докер образ
- используем команды  и `docker exec -it distracted_taussig /bin/bash` `nginx -s reload`
![docker](./image/41.png)
*Если всё сделано верно, то, после сохранения файла и перезапуска контейнера, конфигурационный файл внутри докер образа должен обновиться самостоятельно без лишних действий*
##### Проверяем, что теперь по *localhost:80/status* отдается страничка со статусом **nginx**
![localhost:80/status](./image/43.png)


## Part 5. **Dockle**


##### Просканировать образ из предыдущего задания через `dockle [image_id|repository]`
- Вывод команды: 
![dockle](./image/44.png)
##### Исправляем образ так, чтобы при проверке через dockle не было ошибок и предупреждений
![build](./image/45.png)

- Снова просканируем образ
![dockle](./image/46.png)
- Команда  `dockle -ak NGINX_GPGKEY -i CIS-DI-0010 distracted_taussig:1.0` выполняет проверку Dockerfile на соответствие определенному аспекту безопасности, связанному с использованием переменной окружения NGINX_GPGKEY.
## Part 6. Базовый **Docker Compose**
##### Написать файл *docker-compose.yml*, с помощью которого:
##### 1) Поднять докер контейнер из [Части 5](#part-5-инструмент-dockle) _(он должен работать в локальной сети, т.е. не нужно использовать инструкцию **EXPOSE** и мапить порты на локальную машину)_
##### 2) Поднять докер контейнер с **nginx**, который будет проксировать все запросы с 8080 порта на 81 порт первого контейнера
##### Замапить 8080 порт второго контейнера на 80 порт локальной машины

##### Остановить все запущенные контейнеры
![stop](./image/47.png)
##### Собрать и запустить проект с помощью команд `docker-compose build` и `docker-compose up`
- `docker-compose build`
![docker-compose build](./image/48.png)

- `docker-compose up`
![docker compose up -d](./image/49.png)

##### Проверить, что в браузере по *localhost:80* отдается написанная вами страничка, как и ранее
![localhost:80](./image/50.png)

