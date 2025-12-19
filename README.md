# Traefik proxy (repo-specific notes)

Файлы и назначение
- `docker-compose.yml`: один сервис `traefik` — обратный прокси (ports 80, 443). Монтирует `/var/run/docker.sock` и `traefik/letsencrypt`.
- `traefik/traefik.yml`: статическая конфигурация Traefik (entryPoints, Docker provider, ACME resolver `myresolver`).
- `traefik/letsencrypt/acme.json`: файл хранения ACME (сертификаты/ключи). Файл по умолчанию пустой — Traefik создаст/обновит его.

Быстрый старт
1. Создать внешнюю сеть (однократно):

```bash
docker network create traefik
```

2. Убедиться в правах на `acme.json` (Linux):

```bash
touch traefik/letsencrypt/acme.json
chmod 600 traefik/letsencrypt/acme.json
```

3. Запустить Traefik:

```bash
docker-compose up -d
```

4. Посмотреть логи:

```bash
docker-compose logs -f traefik
```

Изменения и конфигурация
- ACME email и поведение сохранены в [traefik/traefik.yml](traefik/traefik.yml). Если нужно изменить контактный email — отредактируйте этот файл.
- Настройка маршрутов выполняется через Docker (серивисы должны быть подключены к сети `traefik`).

Полезные команды
- Перезапуск Traefik: `docker-compose restart traefik`
- Проверка состояния контейнеров: `docker ps`

Безопасность
- `acme.json` содержит приватные ключи — держите права ограниченными (`600`) и не добавляйте файл в публичные репозитории.