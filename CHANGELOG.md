## Changelog

### 0.1.3

- Игнорирование сервисов LetsEncrypt/Certbot на уровне builder и scheduler:
  - сервисы с именами `letsencrypt` или `certbot`;
  - сервисы с лейблами `com.dokku.letsencrypt: "true"` или `com.dokku.certbot: "true"`.
  Это гарантирует, что сертификатами управляет только внутренний плагин Dokku `letsencrypt`, а не docker-compose.
- Добавлены дополнительные триггеры шедулера для режима `DOKKU_SCHEDULER=compose`:
  - `scheduler-destroy` — выполняет `docker compose down` для проекта приложения;
  - `scheduler-ps` — проксирует `docker compose ps` для просмотра процессов/контейнеров;
  - `scheduler-run` — выполняет one-off команды через `docker compose run --rm` (использует `PROCESS_TYPE` как имя сервиса, по умолчанию `web`).

### 0.1.2

- Поддержка внешних инстансов плагинов Dokku (`dokku-postgres`, `dokku-redis`) через лейбл `com.dokku.datastore`:
  - `postgres:<name>` → переиспользуется контейнер `dokku.postgres.<name>`, если он уже запущен;
  - `redis:<name>` → переиспользуется контейнер `dokku.redis.<name>`, если он уже запущен.
- Улучшена логика подключения внешних сервисов к сети:
  - убран повторный `docker network connect` для контейнеров, которые уже находятся в сети проекта;
  - сеть `<project>_default` гарантированно создаётся при необходимости
  
### 0.1.1

- Добавлена поддержка внешних сервисов через лейбл `com.dokku.external`:
  - если контейнер уже существует, он не пересоздаётся и только подключается к сети проекта compose;
  - если контейнера нет, сервис создаётся обычным образом и попадает в ту же сеть.
- Обновлена логика шедулера:
  - `docker compose up` вызывается только для реально нужных сервисов;
  - для существующих внешних сервисов выполняется `docker network connect` к сети `<project>_default`.
- Добавлены проверки наличия `jq` и более аккуратное использование `DOCKER_BIN`.

### 0.1.0

- Начальная версия плагина:
  - интеграция с Dokku builder (`BUILDER_TYPE=compose`);
  - интеграция с Dokku scheduler (`DOKKU_SCHEDULER=compose`);
  - поиск и валидация compose-файла;
  - сборка сервисов с `build` и выбор основного образа (`web` или `com.dokku.primary-image=true`).


