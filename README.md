### Kittygram_final

![Xarfex](https://github.com/Xarfex/kittygram_final/actions/workflows/main.yml/badge.svg)

## Что это такое
- Данный проект - это возможность посмотреть на котов
- Похвастать своими или просто красивыми усатыми друзьями
- Рассказать про их активности и достижения
- Можете вносить правки в свои посты про хвостатых друзей.

## Инструментарий
- *Языки*: Python, HTML, CSS, JavaScript, Django, React, YAML-словарь
- *Сервисы*: Nginx, Workflow, GitHub Actions(CI/CD), Docker

## Адрес
```https://valerykittygram.hopto.org```

## Как развернуть проект
В этом проекте мы используем методику CI/CD, так что файл workflow триггерит на "push" в любую ветку репозитория и разворачивает проект на сервере.
Но если мы говорим про локальную разработку:
- Установим Docker(Linux):
`sudo apt update`
`sudo apt install curl`
Скрипт для установки Docker
`curl -fSL https://get.docker.com -o get-docker.sh`
Запуск скрипта `sudo sh ./get-docker.sh`
Дополнительно к Docker установите утилиту Docker Compose: `sudo apt-get install docker-compose-plugin`

- Качаем образы с Docker Hub:
Добавим в корневую директорию проекта файл .env(необходимые параметры указаны ниже)
Образ PostgreSQL `docker run --name db \
                --env-file .env \
                -v pg_data:/var/lib/postgresql/data \
                postgres:13.10`
Изменить файл settings.py, чтобы он использовал переменные окружения:
```
# Добавьте import
import os

...
# Этими строчками замените текущую настройку DATABASES
DATABASES = {
    'default': {
        # Меняем настройку Django: теперь для работы будет использоваться
        # бэкенд postgresql
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('POSTGRES_DB', 'django'),
        'USER': os.getenv('POSTGRES_USER', 'django'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD', ''),
        'HOST': os.getenv('DB_HOST', ''),
        'PORT': os.getenv('DB_PORT', 5432)
    }
}
...
```
Собираем образы: 
```
cd kittygram_frontend  # В директории frontend...
docker build -t username/kittygram_frontend .  # ...сбилдить образ, назвать его taski_frontend
cd ../kittygram_backend  # То же в директории backend...
docker build -t username/kittygram_backend .
cd ../kittygram_gateway  # ...то же и в gateway
docker build -t username/kittygram_gateway .
```
Делаем локальный файл docker-compose.yml:
```
version: '3'

volumes:
  pg_data:

services:
  db:
    image: postgres:13.10
    env_file: .env
    volumes:
      - pg_data:/var/lib/postgresql/data
  backend:
    build: ./kittygram_backend/
    env_file: .env
  frontend:
    env_file: .env
    build: ./kittygram_frontend/
# Добавляем новый контейнер: gateway.
  gateway:
    # Сбилдить и запустить образ, 
    # описанный в Dockerfile в папке gateway
    build: ./kittygram_gateway/
    # Ключ ports устанавливает
    # перенаправление всех запросов с порта 9000 хоста
    # на порт 80 контейнера.
    ports:
      - 9000:80
```
- Шлюз в сеть контейнеров готов: `sudo docker compose up`
- Подготавливаем статику в settings.py и docker-compose.yml:
```
# Собрать статику Django
docker compose exec backend python manage.py collectstatic
# Статика приложения в контейнере backend 
# будет собрана в директорию /app/collected_static/.

# Теперь из этой директории копируем статику в /backend_static/static/;
# эта статика попадёт на volume static в папку /static/:
docker compose exec backend cp -r /app/collected_static/. /backend_static/static/
```
- Перезапуск Docker Compose: `docker compose stop && docker compose up --build`


## Файл .env
Файл с переменными окружения ожидает следующие значения:
- TOKEN "Секретный токен"
- POSTGRES_USER=kittygram_user
- POSTGRES_PASSWORD=kittygram_password
- POSTGRES_DB=kittygram
- DB_HOST=db
- DB_PORT=5432
- LOCAL_DEBUG "True - для локальной разработки"
- HOSTS "Allowed hosts"

## Автор
Сычев Валерий, профиль на Github: `https://github.com/Xarfex/`
