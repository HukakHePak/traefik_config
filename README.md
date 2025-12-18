Nginx + Certbot (Docker)

Коротко
- Этот репозиторий содержит `nginx` как обратный прокси и `certbot` для получения/продления TLS через HTTP‑01 (webroot).
- Конфиги nginx лежат в `nginx/conf.d/`. Webroot для ACME — `certbot/www/`. Сертификаты хранятся в `certbot/conf/` (монтируется как `/etc/letsencrypt`).

Основные принципы
- Для успешной валидации HTTP‑01 каждый vhost в `nginx/conf.d/` должен отдавать `/.well-known/acme-challenge/` из `/var/www/certbot` (см. пример в `oxo.hukakhepak.conf`).
- Первичное получение сертификата выполняется вручную (см. раздел ниже). После этого контейнер `certbot` запускает `certbot renew` по расписанию и продляет сертификаты автоматически.

Быстрые команды
- Поднять сервисы:

```sh
docker-compose up -d --build
```

- Остановить и удалить орфаны:

```sh
docker-compose down --remove-orphans
```

- Проверить конфигурацию nginx:

```sh
docker-compose run --rm --entrypoint "nginx -t" nginx
```

- Смотреть логи nginx / certbot:

```sh
docker-compose logs -f nginx
docker-compose logs -f certbot
```

Первичное получение сертификата (ручное)
- Пример для одного домена:

```sh
docker-compose run --rm --entrypoint "certbot certonly --webroot -w /var/www/certbot -d oxo.hukakhepak.ru --email nikak_ne_rak@mail.ru --agree-tos --no-eff-email" certbot
```

- Пример для SAN (несколько имён в одном сертификате):

```sh
docker-compose run --rm --entrypoint "certbot certonly --webroot -w /var/www/certbot -d example.com -d www.example.com --email nikak_ne_rak@mail.ru --agree-tos --no-eff-email" certbot
```

- После успешного получения сертификатов перезапустите `nginx`/compose:

```sh
docker-compose up -d
```

Добавление нового поддомена — минимальные шаги
1. Добавьте `server` блок в `nginx/conf.d/` с `server_name new.domain` и блоком ACME:

```nginx
location ^~ /.well-known/acme-challenge/ {
	root /var/www/certbot;
	try_files $uri =404;
}
```

2. Убедитесь, что DNS A‑запись указывает на сервер и порт 80 доступен извне.
3. Выпустите сертификат вручную (см. команду выше).
4. Перезапустите `docker-compose`.

Про `host.docker.internal`
- `host.docker.internal` резолвится внутри контейнера в адрес хоста и подходит, если целевое приложение публикует порт на хост (например `ports: "8000:8000"`). Работает на Docker Desktop (Windows/Mac). Для Linux это может не работать — в таком случае лучше подключать контейнеры к общей Docker сети и проксировать по `http://service_name:port`.

Примеры и замечания
- В `nginx/conf.d/oxo.hukakhepak.conf` добавлен ACME маршрут и proxy на `host.docker.internal:8000` — если приложение слушает на хосте на порту 8000, proxy будет работать.
- Никогда не включайте SSL‑блоки, которые ссылаются на несуществующие файлы `/etc/letsencrypt/live/.../fullchain.pem`, иначе nginx не запустится.

Если хотите, могу:
- добавить вспомогательный скрипт, который парсит `server_name` и формирует команду для первичной выдачи (автоматизация), или
- переключить workflow на `--nginx` плагин (требует иного образа/плагинов).

----

Файл обновлён.

