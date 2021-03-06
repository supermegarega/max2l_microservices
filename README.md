# max2l_microservices
max2l microservices repository

## Homework 13 Технология_контейнеризации Введение в Docker

### В процессе сделано
- Установлен `docker`
- Установлен `docker-compose`
- Запущено несколько контейнеров
- Изучены основные команды `docker`

### Как запустить проект:

- Ниже указными командами можно проверить версию `docker` и его текущее состояние. 
```
docker version
docker info
```
- Список запущенных контейнеров можно посмотреть командой
```
docker ps
```
- Список скаченных образов можно посмотреть командой
```
docker images
```
- Запустить контейнер из образа 
```
docker run имя_образа
```
- Запустить остановленный контейнер
```
docker start id_контейнера
```
- Присоединиться к запущенному контейнеру
```
docker exec -it id_контейнера bash
docker attach id_контейнера
```
- Для вывода списока всех контейнеров
```
docker ps -a
```
- Для удаления контейнера или образа 
```
docker rm -f id_контейнера
docker rmi -f id_образа 
```
- Создать новай образ из существующего контейнера 
```
docker commit id_контейнера имя_образа
```
- Остановить запущенный контейнер
```
docker stop -f id_контейнера
docker kill -f id_контейнера
```
- Посмотреть занятое на диске место.
```
docker system df
```
---
## Homework 14. Docker контейнеры

### В процессе сделано
 - Установлено ПО `docker-machine`
 - Создан новый проект в GCP
 - gcloud настроен для работы с новым проектом
 - В GCP создан Docker-хост
 - Проверена работа систем namspases (PID, net, user space). При запуске контейнера с namspases PID по умолчанию в контейнере виден только процесс htop, при запуске namspases PID видны все процессы хоста.
 - Создан `Docker` файл для запуска ПО `Puma` в контейнере
 - Создан образ docker
 - Создана учетная запись в ресурсе https://hub.docker.com/
 - Созданый образ загружен в репозиторий hub.docker.com
 - Произведен анализ изменений в запускаемом контейнере и созданого образа.
 - Создан шаблон для packer
 - Создана конфигурация для поднятия нескольких инстансев
 - Созданы плейбуки с использованием динамических инвентори для установки докера и запуска там нескольких приложений.

### Как запустить проект:
 - Проверка статуса Docker машин
```
docker-machine ls
```
  - Определение переменных окружения для работы docker с удаленным docker-хостом
```
eval $(docker-machine env docker-host)
```
 - Запуска контейнера c различными степенями изоляции namspases PID
```
docker run --rm -ti tehbilly/htop
docker run --rm --pid host -ti tehbilly/htop
```
 - Сборка образа docker
```
docker build -t reddit:latest .
```
 - Запуск контейнера используя namespase network host
```
docker run --name reddit -d --network=host reddit:latest
```
 - Поменять tag у образа и загрузить его в репозиторий на hub.docker.com 
```
docker tag reddit:latest max2l/otus-reddit:1.0
docker push max2l/otus-reddit:1.0
```
 - Запуск образа из удалённого репозитория
``` 
docker run --name reddit -d -p 9292:9292 max2l/otus-reddit:1.0 
```
 - Создание образа ОС с установленным docker 
```
cd docker-monolith/infra/
packer validate packer/packer_with_docker.json
packer build packer/packer_with_docker.json
```
 - Создание инстансев для работы `Puma`
```
cd docker-monolith/infra/terraform
terraform plan
terraform apply
```
  - Развертывание инфраструктуры.
```
cd docker-monolith/infra/ansible
ansible-playbook playbook/site.yml
```
---
## Homework 15. Docker-образа. Микросервисы

### В процессе сделано:
  - Созданы Docker файлы для сборки образов микросервисов
  - Произведена сборка микросервисов на основании ранее созданных файлов
  - Создан том `reddit_db` для хранения данных MongoDB
  - Контейнеры docker запушены с ранее созданным томом.
  - Изменены сетевые алиасы и определены переменные окружения для запуска Docker контейнеров.
  - Произведена пересборка образов на образе alpine:3.7. Место, занимаемое на диске после пересборки показано ниже.

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
max2l/ui            1.0                 aad5472f5a45        16 hours ago        777MB
max2l/ui            2.0                 d52ab9f8c18f        3 minutes ago       461MB
max2l/ui            3.0                 04d702032f85        6 seconds ago       206MB
max2l/comment       1.0                 3bd4aa40389b        17 hours ago        769MB
max2l/comment       2.0                 12fc7affcc34        10 minutes ago      198MB
max2l/post          1.0                 29a4cc69ad96        17 hours ago        102MB
max2l/post          2.0                 9fd9b7022067        About an hour ago   61.8MB
```
  - Произведена дополнительная оптимизация Docker файлов для уменьшения размера образов. В одном слое сделана установка ПО, сборка приложения и удаление ПО необходимого для сборки. Этот подход позволил еще больше сократить размер образа. Но я не думаю, что этот подход следует применять в дальнейшем. При его использовании усложняется отладка сборки и дальнейшее сопровождение образов. Для решения подобного рода задач лучше использовать отдельный Build сервер.

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
max2l/post          3.0                 a06ae092a763        2 minutes ago       59.5MB
max2l/comment       3.0                 634ff7b3ec7e        About an hour ago   53.6MB
max2l/ui            4.0                 d69f1960093b        About an hour ago   61.7MB
max2l/ui            3.0                 04d702032f85        3 hours ago         206MB
max2l/ui            2.0                 d52ab9f8c18f        3 hours ago         461MB
max2l/comment       2.0                 12fc7affcc34        3 hours ago         198MB
max2l/post          2.0                 9fd9b7022067        3 hours ago         61.8MB
max2l/ui            1.0                 aad5472f5a45        19 hours ago        777MB
max2l/comment       1.0                 3bd4aa40389b        19 hours ago        769MB
max2l/post          1.0                 29a4cc69ad96        19 hours ago        102MB
mongo               latest              a0f922b3f0a1        5 days ago          366MB
```
  - Произведен другой подход при оптимизации размера образов. Для сборки ПО используются промежуточный образ и после сборки все необходимые файлы копируются в конечный образ. Для этого используются несколько директив `FROM` в Docker файлах. Размер образов после сборки показан ниже.
```
max2l/ui 5.0 1bc0d5a4815a 18 minutes ago 48.9MB
max2l/ui 4.0 d69f1960093b 8 days ago 61.7MB
max2l/ui 3.0 04d702032f85 8 days ago 206MB
max2l/ui 2.0 d52ab9f8c18f 8 days ago 461MB
max2l/ui 1.0 aad5472f5a45 8 days ago 777MB
```
```
max2l/comment 4.0 17ddf9efd54a 27 minutes ago 40.8MB
max2l/comment 3.0 634ff7b3ec7e 8 days ago 53.6MB
max2l/comment 2.0 12fc7affcc34 8 days ago 198MB
max2l/comment 1.0 3bd4aa40389b 8 days ago 769MB
``` 

### Как запустить проект:
  - Создание docker machine в GCP
```
export GOOGLE_PROJECT=docker-201806
docker-machine create --driver google \
  --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
  --google-machine-type n1-standard-1 \
  --google-zone europe-west1-b \
  docker-host 
```
  - Сборка образов микросервисов
```
docker pull mongo:latest
docker build -t max2l/post:1.0 ./post-py
docker build -t max2l/comment:1.0 ./comment
docker build -t max2l/ui:1.0 ./ui
```
  - Создане сети для приложения
```
docker network create reddit
```
  - Запуск контейнеров 
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post max2l/post:1.0
docker run -d --network=reddit --network-alias=comment max2l/comment:1.0
docker run -d --network=reddit -p 9292:9292 max2l/ui:2.0
```
  - Создание volume
```
 docker volume create reddit_db
```
  - Запуск MongoDB  с томом `reddit_db`
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
```
  - Запуск контейнеров c другими сетевым алиасами
```
docker run -d --network=reddit --network-alias=post_db_new_alias --network-alias=comment_db_new_alias mongo:latest
docker run -d --network=reddit --network-alias=post_new_alias -e "POST_DATABASE_HOST=post_db_new_alias" max2l/post:1.0
docker run -d --network=reddit --network-alias=comment_new_alias -e "COMMENT_DATABASE_HOST=comment_db_new_alias" max2l/comment:1.0
docker run -d --network=reddit -p 9292:9292 -e "POST_SERVICE_HOST=post_new_alias" -e "COMMENT_SERVICE_HOST=comment_new_alias" max2l/ui:2.0
```
---
## Homework 16. Docker. сети, docker-compose.
### В процессе сделано:
  - Произведена установка `docker-compose`.
  - Запушен контейнер с сетевым драйвером `none`. Произведена диагностика работы сетевого стека контейнера.
  - Запущено несколько контейнеров Docker с образом `Nginx` в сетевом пространстве хоста. При запуске второго и последующего контейнера они падают с ошибкой. Это связано с тем, что порт 80 на хосте уже занят первым запущенным контейнером. Увидеть это можно выполнив команду
```
docker logs db4aeed79c69
2018/05/01 11:17:01 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/05/01 11:17:01 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/05/01 11:17:01 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/05/01 11:17:01 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/05/01 11:17:01 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/05/01 11:17:01 [emerg] 1#1: still could not bind()
```
  - При запуске контейнеров с драйвером `none` для каждого контейнера создается отдельный Network Name Space. Это необходимо для изоляции сетевого стека контейнеров друг от друга и от хоста. При запуске контейнеров с сетевым драйвером `host` они создаться в Network Name Space `default`.
  - При запуске контейнеров с сетевым драйвером `bridge` взаимосвязь контейнеров происходит за счет сетевого моста, что в свою очередь дает ряд новых особенностей.
    - Можно производить запуск нескольких контейнеров использующих фиксированные порты не опасаясь конфликта с сетевым стеком хоста на котором запушен Docker
    - Каждый запушенный `bridge` интерфейс имеет префикс сети создаваемой в Docker и он связан с сетевыми интерфейсом внутри контейнера входящего в эту сеть.
    - Создаваемые сети могут быть изолированы друг от друга.
    - Созданные в Docker сети транслируются в сеть сервера на котором он запушен.
    - В случае определения портов при запуске контейнера, эти порты будут транслироваться в сеть Docker хоста
    - Для трансляции портов используется `docker-proxy`
  - Настроен `docker-compose.yml` для автоматизации запуска контейнеров используя `docker-compose`
  - При запуске контейнеров через `docker-compose` необходимо определить переменные окружения в файле `.env`
  - Базовое имя проекта образуется из названия корневой директории. Задать другое базовое имя можно либо переименовав текущую директорию, либо задав переменную окружения COMPOSE_PROJECT_NAME, либо запустить проект с опцией `-p`. В случае запуска проекта с этой опцией все дальнейшие действия так же необдодимо производить с этой опцией. 
```
docker-compose -p new_project_base_name up -d
docker-compose -p new_project_base_name down
```
  - Для возможности редактирования кода каждого из приложений без пересборки образов.
    - Cозданы тома с типом `bind`.
    - Источники этих томов находятся в директориях `src/ui/`, `src/comment/`, `src/post-py/`.
    - Целевая директория этих томов находится в каталоге `/app` 
  - Для запуска приложений Puma в `docker-compose.override.yml` переопределены команды запуска по умолчанию ипи старте Docker контейнеров.
     
### Как запустить проект:
  - Создание docker machine в GCP
```
export GOOGLE_PROJECT=docker-201806
docker-machine create --driver google \
  --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
  --google-machine-type n1-standard-1 \
  --google-zone europe-west1-b \
  docker-host 
```
  - Список Network Name Space
```
sudo ip netns
```
  - Просмотр сетевых интерфейсов в определенном сетевом Name Space
```
sudo ip netns exec 234f51036f69 ip a
sudo ip netns exec default ip a
```
  - Запуск проекта reddit с сетевым драйвером `bridge` 
```
docker network create reddit
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post max2l/post:1.0
docker run -d --network=reddit --network-alias=comment max2l/comment:1.0
docker run -d --network=reddit -p 9292:9292 max2l/ui:1.0
```
 - Запуск контейнеров в изолированных сетях 
```
 docker network create back_net --subnet=10.0.2.0/24
 docker network create front_net --subnet=10.0.1.0/24
 docker run -d --network=front_net -p 9292:9292 --name ui max2l/ui:1.0
 docker run -d --network=back_net --name comment max2l/comment:1.0
 docker run -d --network=back_net --name post max2l/post:1.0
 docker run -d --network=back_net --name mongo_db --network-alias=post_db --network-alias=comment_db mongo:latest
 docker network connect front_net post
 docker network connect front_net comment
```
 - Список сетевых namespaces 
```
sudo ip netns
564e8fa2b2b7
52a698935e1b
default
```
 - Список запушенных контейнеров
```
docker ps
CONTAINER ID        IMAGE                        COMMAND                 CREATED             STATUS              PORTS               NAMES
2ebdb7ee5007        joffotron/docker-net-tools   "sh -c 'sleep 10000'"   2 minutes ago       Up 2 minutes                            net_test5
9c3ba2c1a484        joffotron/docker-net-tools   "sh -c 'sleep 10000'"   2 minutes ago       Up 2 minutes                            net_test4
f79e4fce0e14        joffotron/docker-net-tools   "sh -c 'sleep 10000'"   2 minutes ago       Up 2 minutes                            net_test3
cfaee642096f        joffotron/docker-net-tools   "sh -c 'sleep 10000'"   2 minutes ago       Up 2 minutes                            net_test1
c6014c9bfd54        joffotron/docker-net-tools   "sh -c 'sleep 10000'"   2 minutes ago       Up 2 minutes                            net_test2
```
 - Сетевые интерфейсы в Network Name Space `default`
```
sudo ip netns exec default ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 42:01:0a:84:00:02 brd ff:ff:ff:ff:ff:ff
    inet 10.132.0.2/32 brd 10.132.0.2 scope global ens4
       valid_lft forever preferred_lft forever
    inet6 fe80::4001:aff:fe84:2/64 scope link
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:37:a8:dd:3a brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```
  - Сетевые интерфейсы в Network Name Space созданом при запуске контейнера с сетевым драйвером `none`
```
sudo ip netns exec 0e4e37b7ec14 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```
```
sudo ip netns exec 234f51036f69 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```
  - Комманды для связи контейнеров с другими сетями docker.
```
docker network connect front_net post
docker network connect front_net comment
```
---
## Homework 17. Gitlab CI. Построение процесса непрерывной интеграции.
### В процессе сделано:
  - Создана виртуальная машина в GCP и произведена omnibus-установка `GitLab CI` с использованием docker-compose.
  - Отключена регистрация новых пользователей.
  - В группе `homework` Создан новый проект `example`
  - Произведена интеграция git репозитория `max2l_microservices` в созданном инстансе `GitLab CI`.
  - Описан `pipeline` в файле `.gitlab-ci.yml`.
  - Создан и зарегистрирован `Runner`.
  - Настроенно тестирование проекта с использованием `pipeline`.
  - Произведено тестирование проекта.
  - Настроенна интеграция `gitlab` и `slack`.
  - Для установки нескольких инстансев `Gitlab Runner` разработана конфигурация для `Terraform`. Количество поднимаемых интансев необходимо задать в переменной `count_instances` изменив файл `terraform/terraform.tfvars`. Для оптимизации времени разворачивания инстансев можно создать образ с установленным `docker` и `Girlab Runner` используя `Packer`.
  - Если необходимо произвести установку `Runner` не в облаке, а на "железных" серверах с предустановленной OC, то для этого можно использовать `Ansible`. Предварительно на серверах должны быть "разложены" ssh ключи. 
  - Если это действие необходимо произвести только один раз, то можно использовать утилиту `pdsh`. При этом на всех серверах должен быть настроем доступ по ssh ключам.
### Как запустить проект:
  - Интеграции git репозитоиря `GitLab CI`.
```
git checkout -b gitlab-ci-1
git remote add gitlab http://max2l_microservices/homework/example.git
git push gitlab gitlab-ci-1
```
  - Создание `Runner`.
```
docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
```
  - Регистрация `Runner`.
```
docker exec -it gitlab-runner gitlab-runner register
```
или 
```
docker exec -it gitlab-runner gitlab-runner register \
    --non-interactive \
    --url "http://35.233.54.249/" \
    --registration-token "Token for runer in gitlab" \
    --executor "docker" \
    --docker-image alpine:latest \
    --description "docker-runner" \
    --tag-list "linux,xenial,ubuntu,docker" \
    --run-untagged \
    --locked="false"
```
  - Запуск нескольких инстансев `Runner`.
    - Количество запускаемых инстансев задается в переменной `count_instances`. Для этого необходимо отредактировать файл `terraform/terraform.tfvars`
    - Далее необходимо выполнить команды
```
terraform play
terraform apply
```
---
