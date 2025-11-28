# Photo Editor Web App — Руководство по установке

## Обзор

Это руководство описывает установку и настройку **Photo Editor Web App** на локальной машине или сервере.

## Системные требования

### Локальная разработка

- **ОС:** Windows 8+, macOS 10.14+, Ubuntu 18.04+
- **Node.js:** 16.0.0 или выше
- **npm или yarn:** последняя версия
- **Git:** для клонирования репозитория
- **Браузер:** Chrome, Firefox, Safari
- **Оперативная память:** минимум 5 ГБ
- **Место на диске:** 1 ГБ

### Продакшн-сервер

- **ОС:** Ubuntu 20.04+ или аналогичная Linux
- **Docker:** 20.10+ (опционально, но рекомендуется)
- **Node.js:** 16.0.0 LTS
- **nginx:** для проксирования запросов
- **SSL сертификат:** для HTTPS
- **Место на диске:** 2 ГБ (с логами)

## Установка для локальной разработки

### Шаг 1: Клонировать репозиторий

```bash
git clone https://github.com/photoeditor/photo-editor-web.git
cd photo-editor-web
```

### Шаг 2: Установить зависимости

```bash
npm install
```

Или с использованием yarn:

```bash
yarn install
```

**Ожидаемый результат:** в папке `node_modules` будут установлены все пакеты.

### Шаг 3: Создать файл конфигурации

Скопируйте пример файла конфигурации:

```bash
cp .env.example .env.development
```

Откройте `.env.development` и обновите значения:

```
REACT_APP_API_URL=http://localhost:3001
REACT_APP_API_KEY=dev-key-12345
NODE_ENV=development
```

### Шаг 4: Запустить приложение в режиме разработки

```bash
npm start
```

**Вывод в консоли:**

```
Compiled successfully!

You can now view photo-editor-web in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://192.168.1.100:3000

Note that the development build is not optimized.
To create a production build, use npm run build.
```

### Шаг 5: Открыть в браузере

Перейдите на `http://localhost:3000`. Приложение должно загрузиться с горячей перезагрузкой при изменении кода.

## Установка для продакшена (без Docker)

### Шаг 1: Подготовить сервер

Подключитесь к серверу по SSH и обновите пакеты:

```bash
sudo apt update
sudo apt upgrade -y
```

### Шаг 2: Установить Node.js и npm

```bash
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs
node --version  # проверить версию
npm --version
```

### Шаг 3: Клонировать репозиторий

```bash
sudo mkdir -p /var/www/photo-editor
sudo chown $USER:$USER /var/www/photo-editor
cd /var/www/photo-editor
git clone https://github.com/photoeditor/photo-editor-web.git .
```

### Шаг 4: Установить зависимости

```bash
npm install --production
```

### Шаг 5: Создать файл конфигурации продакшена

```bash
cp .env.example .env.production
```

Обновите значения в `.env.production`:

```
REACT_APP_API_URL=https://api.photoeditor.example.com
REACT_APP_API_KEY=prod-key-xxxxx
NODE_ENV=production
REACT_APP_LOG_LEVEL=error
```

### Шаг 6: Собрать приложение для продакшена

```bash
npm run build
```

Результат будет в папке `build/`.

### Шаг 7: Установить и настроить nginx

```bash
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

Создайте конфиг для nginx:

```bash
sudo nano /etc/nginx/sites-available/photoeditor
```

Содержимое конфига:

```nginx
server {
    listen 80;
    server_name photoeditor.example.com;
    
    # Редирект на HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name photoeditor.example.com;
    
    ssl_certificate /etc/letsencrypt/live/photoeditor.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/photoeditor.example.com/privkey.pem;
    
    root /var/www/photo-editor/build;
    index index.html;
    
    # Кэширование статических файлов
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Проксирование API запросов
    location /api/ {
        proxy_pass https://api.photoeditor.example.com;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # SPA маршрутизация
    location / {
        try_files $uri /index.html;
    }
}
```

Включите конфиг:

```bash
sudo ln -s /etc/nginx/sites-available/photoeditor /etc/nginx/sites-enabled/
sudo nginx -t  # проверить конфиг
sudo systemctl reload nginx
```

### Шаг 8: Установить SSL сертификат (Let's Encrypt)

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot certonly --nginx -d photoeditor.example.com
```

### Шаг 9: Запустить приложение с помощью PM2 (для автозагрузки)

```bash
sudo npm install -g pm2
pm2 start npm --name "photo-editor" -- start
pm2 startup
pm2 save
```

## Установка с использованием Docker

### Шаг 1: Установить Docker

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### Шаг 2: Создать Dockerfile

В корне проекта создайте файл `Dockerfile`:

```dockerfile
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:16-alpine
WORKDIR /app
RUN npm install -g serve
COPY --from=builder /app/build ./build
EXPOSE 3000
CMD ["serve", "-s", "build", "-l", "3000"]
```

### Шаг 3: Собрать и запустить контейнер

```bash
docker build -t photo-editor:latest .
docker run -p 3000:3000 --env-file .env.production photo-editor:latest
```

## Проверка установки

После установки проверьте следующее:

1. **Приложение загружается:**
   - Откройте `http://localhost:3000` (или ваш URL продакшена)
   - Должна загрузиться главная страница с кнопкой "Загрузить фото"

2. **API доступен:**
   ```bash
   curl https://api.photoeditor.example.com/health
   # Ожидаемый ответ: { "status": "OK" }
   ```

3. **Консоль браузера чиста:**
   - Нажмите F12, перейдите на вкладку "Console"
   - Не должно быть критических ошибок (warnings допустимы)

4. **Функциональность работает:**
   - Попробуйте загрузить тестовое изображение
   - Примените фильтр и скачайте результат

## Решение проблем

### Ошибка: "Cannot find module"

```bash
rm -rf node_modules package-lock.json
npm install
```

### Ошибка подключения к API

Проверьте переменные окружения в `.env.development` или `.env.production`.

### Порт 3000 уже занят

```bash
# Найти процесс, занимающий порт
lsof -i :3000
# Завершить процесс (замените PID на значение из вывода выше)
kill -9 PID
```

## Обновление приложения

Для обновления приложения на продакшене:

```bash
cd /var/www/photo-editor
git pull origin main
npm install --production
npm run build
sudo systemctl reload nginx
```

## Поддержка

- **Документация:** https://docs.photoeditor.example.com
- **Форум:** https://community.photoeditor.example.com
- **Email:** support@photoeditor.example.com
