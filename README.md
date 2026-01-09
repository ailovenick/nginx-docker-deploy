# Nginx + PHP-FPM + SSL (Docker Compose)

Этот проект представляет собой модульную заготовку для веб-разработки. Вы можете использовать его как:
1. Просто Nginx для статических файлов (HTML/CSS/JS).
2. Nginx + PHP.
3. Nginx + SSL (HTTPS).
4. Полный стек: Nginx + PHP + SSL.

## Структура папок

*   `www/` — Ваши файлы сайта (index.php, index.html).
*   `nginx/conf.d/default.conf` — Главный файл конфигурации сайта.
*   `nginx/certs/` — Сюда нужно положить SSL сертификаты.
*   `nginx/snippets/` — Модули конфигурации (PHP, SSL), которые подключаются в главном конфиге.

---

## Инструкция: Сценарии использования

### Сценарий 1: Только статика (HTML) по HTTP
*Используется по умолчанию.*
1. Положите ваши `.html` файлы в папку `www/`.
2. Запустите: `docker-compose up -d`.
3. Сайт доступен по адресу `http://localhost`.

### Сценарий 2: Добавление PHP (без SSL)
1. Откройте `nginx/conf.d/default.conf`.
2. В блоке `server { listen 80; ... }`:
   * Закомментируйте старый `location /`.
   * Раскомментируйте блок `[ВАРИАНТ 2] Включение PHP`.
   * Раскомментируйте строку `include /etc/nginx/snippets/php_fastcgi.conf;`.
3. Перезапустите Nginx: `docker-compose restart nginx`.

### Сценарий 3: Включение SSL (HTTPS)
Для работы SSL нужны сертификаты.

**1. Генерация самоподписанного сертификата**
Выполните эту команду в терминале (PowerShell или CMD) в корне проекта:

```bash
docker run --rm -v %cd%/nginx/certs:/certs alpine/openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /certs/selfsigned.key -out /certs/selfsigned.crt -subj "/C=RU/ST=Moscow/L=Moscow/O=Localhost/CN=localhost"
```
*(Для Linux/Mac замените `%cd%` на `$PWD`)*.
Это создаст файлы `selfsigned.crt` и `selfsigned.key` в папке `nginx/certs`.

**2. Настройка конфига**
1. Откройте `nginx/conf.d/default.conf`.
2. Раскомментируйте ВЕСЬ нижний блок `server { listen 443 ssl; ... }`.
3. (Опционально) Раскомментируйте редирект в верхнем блоке `[ВАРИАНТ 3]`.
4. Перезапустите: `docker-compose restart nginx`.

### Сценарий 4: Отключение PHP (для экономии ресурсов)
Если вам нужен только статический сервер:
1. Откройте `docker-compose.yml`.
2. Закомментируйте весь сервис `php`.
3. В сервисе `nginx` закомментируйте секцию `depends_on`.
4. Убедитесь, что в `nginx/conf.d/default.conf` **НЕ** подключен файл `php_fastcgi.conf`.

---

## Полезные команды

*   Запустить проект: `docker-compose up -d`
*   Остановить проект: `docker-compose down`
*   Посмотреть логи Nginx: `docker-compose logs -f nginx`
*   Посмотреть логи PHP: `docker-compose logs -f php`
