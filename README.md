# Лабораторная работа: Управление взаимодействием нескольких контейнеров

## Цель работы

Выполнив данную работу, студент сможет управлять взаимодействием нескольких контейнеров, используя Docker для развертывания PHP-приложения с серверами Nginx и PHP-FPM.

## Задание

Создать PHP-приложение на базе двух контейнеров:
- **nginx** (frontend)
- **php-fpm** (backend)

## Подготовка

Для выполнения работы необходимо:
1. Установить Docker на компьютер.
2. Иметь опыт выполнения лабораторной работы №3.

## Выполнение

### 1. Подготовка репозитория

- Создайте репозиторий `containers06` и склонируйте его на компьютер:
  ```sh
  git clone https://github.com/yourusername/containers06.git
  cd containers06
  ```
- В директории `containers06` создайте структуру каталогов:
  ```sh
  mkdir -p mounts/site
  ```
- Скопируйте в `mounts/site` код сайта на PHP из предыдущих лабораторных работ.

### 2. Настройка .gitignore
Создайте файл `.gitignore` в корне проекта и добавьте в него строки:
```
# Ignore files and directories
mounts/site/*
```

### 3. Настройка Nginx

Создайте файл `containers05/nginx/default.conf` со следующим содержимым:
```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### 4. Создание Docker-сети

Создайте сеть `internal` для контейнеров:
```sh
docker network create internal
```

### 5. Запуск контейнеров

#### Запуск backend-контейнера
```sh
docker run -d --name backend \
    --network internal \
    -v $(pwd)/mounts/site:/var/www/html \
    php:7.4-fpm
```

#### Запуск frontend-контейнера
```sh
docker run -d --name frontend \
    --network internal \
    -v $(pwd)/mounts/site:/var/www/html \
    -v $(pwd)/containers05/nginx/default.conf:/etc/nginx/conf.d/default.conf \
    -p 80:80 \
    nginx:1.23-alpine
```

### 6. Тестирование

Перейдите в браузере по адресу: [http://localhost](http://localhost). Если отображается базовая страница nginx, попробуйте перезагрузить страницу.

## Выводы

1. Контейнеры успешно взаимодействуют друг с другом через созданную сеть `internal`.
2. Nginx обрабатывает HTTP-запросы и передает PHP-скрипты на обработку контейнеру `php-fpm`.
3. Правильная настройка конфигурации Nginx и монтирование директорий позволяет корректно запускать PHP-приложение в контейнерах.

## Ответы на вопросы

### 1. Каким образом в данном примере контейнеры могут взаимодействовать друг с другом?
Контейнеры взаимодействуют через созданную сеть `internal`. Контейнер `frontend` (Nginx) отправляет PHP-запросы контейнеру `backend` (PHP-FPM) по имени `backend`, используя `fastcgi_pass backend:9000`.

### 2. Как видят контейнеры друг друга в рамках сети internal?
Контейнеры могут обращаться друг к другу по именам сервисов (`frontend` и `backend`). Docker автоматически настраивает DNS-резолвинг в пределах сети `internal`, позволяя `frontend` обращаться к `backend` по его имени.

### 3. Почему необходимо было переопределять конфигурацию nginx?
Стандартная конфигурация Nginx не поддерживает обработку PHP-файлов. Переопределение конфигурации необходимо для корректной передачи PHP-запросов в контейнер `php-fpm`.

