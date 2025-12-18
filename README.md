# Nginx + Certbot (Docker)

Коротко: в этом каталоге есть `docker-compose.yml`, `nginx/conf.d` и папка `certbot` с Dockerfile для планировщика (cron).

## Быстрый старт

1) Собрать и запустить сервисы:

```sh
docker-compose up -d --build
```

2) Первичное получение сертификатов для `hukakhepak.ru` и `www.hukakhepak.ru` (используется email `nikak_ne_rak@mail.ru`):

```sh
docker-compose run --rm certbot_init
```

3) После успешного получения сертификатов — перезапустите/поднимите всё:

```sh
docker-compose up -d
```

## Что сделано

- `docker-compose.yml` — сервисы `nginx`, `certbot` (сборка из `./certbot`) и `certbot_init` для первичного запроса.
- `docker-compose.yml` — сервисы `nginx`, `certbot` (сборка из `./certbot`) и `certbot_init` для первичного запроса.
- `nginx/conf.d/hukakhepak.conf` — пример конфига с ACME-challenge и проксированием (настроен для `hukakhepak.ru`).
- `certbot/Dockerfile` и `certbot/renew-cron.sh` — образ с cron, который проверяет `certbot renew` ежедневно (03:00).

## Настройки и рекомендации

- При первом запуске `certbot_init` конфиг настроен на `hukakhepak.ru` и использует email `nikak_ne_rak@mail.ru`.
- Если позже захотите изменить email или добавить домены, отредактируйте `docker-compose.yml` (`certbot_init`) и запустите команду получения сертификата заново.
- Порты 80 и 443 должны быть доступны извне при первичном запросе.
- Логи cron находятся в контейнере `certbot` в `/var/log/cron.log`.
- Для нескольких доменов добавьте `-d` параметры в `certbot_init`.

## Команды для отладки

Проверить логи nginx:

```sh
docker-compose logs -f nginx
```

Проверить логи certbot/cron:

```sh
docker-compose logs -f certbot
```

Запустить одноразовый certbot вручную (альтернатива `certbot_init`):

```sh
docker-compose run --rm --entrypoint "certbot certonly --webroot -w /var/www/certbot -d hukakhepak.ru -d www.hukakhepak.ru --email nikak_ne_rak@mail.ru --agree-tos --no-eff-email" certbot
```

---

Если хотите, подставлю ваши домены/email и удалю `note.txt` — скажите, какие значения использовать.
