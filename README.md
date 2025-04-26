# Ansible Playbook для развертывания стенда с Nginx, PHP-FPM, Python и Node.js на Ubuntu

Этот Ansible playbook автоматизирует развертывание веб-стенда, включающего Nginx, PHP-FPM (для работы с Laravel и WordPress), Python (для Flask и Django) и Node.js (для React и Angular) на сервере Ubuntu.

## Обзор

Playbook выполняет следующие задачи:

1. **Обновление и установка необходимых пакетов**:
   - Устанавливает Nginx для веб-сервера.
   - Устанавливает PHP и PHP-FPM, включая необходимые расширения.
   - Устанавливает Python и pip для работы с Flask и Django.
   - Устанавливает Node.js и npm для работы с React и Angular.

2. **Создание корневого каталога веб-приложения**:
   - Создает каталог `/var/www/html`, где будут размещаться веб-приложения.

3. **Настройка Nginx**:
   - Использует шаблон конфигурации Nginx для настройки сервера с поддержкой PHP и проксирования запросов к Flask и Django приложениям.

4. **Запуск и включение служб**:
   - Запускает и включает службы Nginx и PHP-FPM на автозагрузку.

5. **Создание образцов приложений**:
   - Создает простое приложение Flask и устанавливает проект Django.

6. **Установка зависимостей для Django**:
   - Устанавливает зависимости из файла `requirements.txt` для приложения Django.

## Файл конфигурации Nginx (nginx.conf.j2)

Создайте файл `nginx.conf.j2` в каталоге `templates/` с содержимым:

```nginx
server {
    listen 80;
    server_name your_domain.com;  # Замените на ваше доменное имя

    location / {
        root /var/www/html;
        index index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;  # Измените на вашу версию PHP
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location /flask/ {
        proxy_pass http://127.0.0.1:5000;  # Порт Flask приложения
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /django/ {
        proxy_pass http://127.0.0.1:8000;  # Порт Django приложения
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}