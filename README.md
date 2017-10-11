# Пример работы с Docker и docker-machine в GCE

Создать докер-машину в GCE


    docker-machine create --driver google \
        --google-project XXX \
        --google-zone europe-west1-b \
        --google-machine-type g1-small \
        --google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) \
        docker-host

Собрать образ


    docker build -t reddit:latest .

Запустить приложение в докер-машине


    eval $(docker-machine env docker-host)
    docker run --name reddit -d --network=host reddit:latest

Создать волюм и добавить сеть:


    docker volume create reddit_db
    docker network create reddit

Запуск приложения на микросервисах (при запуске подставить нужные теги образов, это только пример):


    docker run -d --network=reddit \
        -v reddit_db:/data/db \
        --network-alias=post_db \
        --network-alias=comment_db \
        mongo:latest \
    && docker run -d --network=reddit \
        --network-alias=post \
        ujhgj/post:1.0 \
    && docker run -d --network=reddit \
        --network-alias=comment \
        ujhgj/comment:1.0 \
    && docker run -d --network=reddit \
        -p 9292:9292 \
        ujhgj/ui:3.0
