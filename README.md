# Nginx + PHP-FPM + SSL (Docker Compose - Alpine Edition)

Этот проект — легкая и модульная заготовка для веб-сервера на базе **Alpine Linux**.

## Структура папок

*   `www/` — Ваши файлы сайта (index.php, index.html).
*   `logs/` — **Здесь появляются логи сервера:**
    *   `logs/nginx/access.log` — Кто заходил на сайт.
    *   `logs/nginx/error.log` — Ошибки веб-сервера.
    *   `logs/php/php-error.log` — Ошибки PHP скриптов.
*   `nginx/conf.d/default.conf` — Главный файл конфигурации сайта.
*   `nginx/certs/` — Сюда нужно положить SSL сертификаты.
*   `nginx/snippets/` — Модули конфигурации (PHP, SSL).

---

## Сценарии использования

1.  **Статика (HTTP):** Работает из коробки.
2.  **PHP:** Раскомментируйте `include` в `nginx/conf.d/default.conf`.
3.  **SSL (HTTPS):** Сгенерируйте сертификаты и раскомментируйте блок `server` на порту 443.

---

## Работа с SSL сертификатами

Для работы HTTPS (порт 443) требуются два файла в папке `nginx/certs/`:
- `selfsigned.crt` (Публичный сертификат)
- `selfsigned.key` (Приватный ключ)

### Как сгенерировать сертификаты (одной командой)

**Для командной строки (CMD):**
```bash
docker run --rm -v %cd%/nginx/certs:/certs alpine/openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /certs/selfsigned.key -out /certs/selfsigned.crt -subj "/C=RU/ST=Moscow/L=Moscow/O=Development/CN=localhost"
```

**Для PowerShell:**
```powershell
docker run --rm -v ${PWD}/nginx/certs:/certs alpine/openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /certs/selfsigned.key -out /certs/selfsigned.crt -subj "/C=RU/ST=Moscow/L=Moscow/O=Development/CN=localhost"
```

**Разбор параметров:**
*   `-newkey rsa:2048` — создает новый приватный ключ RSA длиной 2048 бит.
*   `-keyout` — путь, по которому будет сохранен приватный ключ.
*   `-out` — путь для сохранения самого сертификата.
*   `-subj` — задает данные владельца сертификата в одну строку (чтобы не отвечать на вопросы интерактивно):
    *   `/C=RU` — Страна (Country).
    *   `/ST=Moscow` — Регион/Область (State).
    *   `/L=Moscow` — Город (Locality).
    *   `/O=Development` — Название организации (Organization).
    *   `/CN=localhost` — Доменное имя (Common Name), для которого выпускается сертификат.

---

## Безопасность и Root права

Вы можете заметить, что процессы в контейнере запускаются от имени `root`. Это нормально и безопасно:

1.  **Nginx:** Главный процесс стартует как `root`, чтобы иметь право открыть привилегированные порты (80 и 443). После старта он сразу создает рабочие процессы ("workers") от имени обычного пользователя `nginx`.
2.  **PHP:** Аналогично, менеджер процессов запускается как root, но сами скрипты выполняются от пользователя `www-data`.

---

## Запуск

1.  **Запуск всех сервисов:**
    ```bash
    docker-compose up -d
    ```

2.  **Перезапуск для применения настроек:**
    ```bash
    docker-compose restart
    ```

## Примечание по логам
Теперь логи пишутся в файлы в папке `logs/` на вашем компьютере. Команда `docker logs` может показывать меньше информации, так как поток перенаправлен в файлы.
