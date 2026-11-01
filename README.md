# kafka

## Структура Compose

Файлы Docker Compose находятся в `compose/`; основной файл — `compose/docker-compose.yml`.
Команды запускайте из корня проекта через Makefile: он задаёт `--project-directory`
на корень checkout. Build contexts, bind mounts, `env_file` и секреты остаются
относительными к этому корню; имена Compose project, сервисов, сетей и томов сохранены.
При явном вызове Docker используйте `docker compose --project-directory . -f compose/docker-compose.yml ...`.
Если в приватной `.env` или окружении задан `COMPOSE_FILE`, добавьте `compose/`
к прежним именам файлов, сохранив прежний набор overlays и порядок. Для плавного
перехода Make также распознаёт прежние короткие имена, если файлы уже перенесены.
Не меняйте project name и не удаляйте прежние тома для переноса файлов.

## Размещение после переноса 2 ноября 2026

WSL. UI — только профиль `tools`. Heap256–512 MiB; контейнер до2 GiB. Полный KRaft volume сохраняет cluster ID и consumer offsets. Advertised hostname `kafka:9092` требует DNS/extra_hosts на WSL у удалённых клиентов. Loopback9092 предоставляется через tailnet Serve.

Исходный стек на `server` выключен и защищён `.retired`; данные оставлены для отката. Совместимый forwarding обслуживает оставшихся клиентов старой VM. Не включать старых writers параллельно новым. После новых записей откат требует свежей копии данных. Автодоставка направлена на новый хост; `server` и `private-test-server` не являются целевыми площадками.

`make forward-up` запускает только TCP compatibility proxy на retired VM, без Kafka/Redis/БД. Конфигурация — `migration/forwarder.cfg`, все upstream — новые хосты. Это временная совместимость для registration и старых OTLP-клиентов; notes-bot подключается напрямую. После вывода последних клиентов остановить `notes-infra-forwarder`. `make check` валидирует обе самостоятельные конфигурации.
