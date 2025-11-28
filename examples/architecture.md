# Photo Editor Web App — Архитектура и обзор системы

## Обзор архитектуры

**Photo Editor Web App** построена по модульной архитектуре с разделением на фронтенд (веб-интерфейс) и бэкенд (API и обработка изображений).

```
┌─────────────────────────────────────────────────────────────┐
│                      Браузер пользователя                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         React приложение (Photo Editor)              │   │
│  │  ┌────────────┐  ┌──────────┐  ┌───────────────┐   │   │
│  │  │  Components│  │  Hooks   │  │  Redux Store  │   │   │
│  │  │  (Editor,  │  │(useImage,│  │(state mgmt)   │   │   │
│  │  │ Toolbar,   │  │ useFilter│  │               │   │   │
│  │  │ FilterPanl)│  │)         │  │               │   │   │
│  │  └────────────┘  └──────────┘  └───────────────┘   │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                   │
│                    HTTP запросы (Axios)                       │
│                           │                                   │
└───────────────────────────┼───────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │      Photo Editor API (Node.js)       │
        │                                        │
        │  ┌─────────────────────────────────┐  │
        │  │     Express маршруты             │  │
        │  │ /images/upload                   │  │
        │  │ /images/{id}/filters             │  │
        │  │ /images/{id}/crop                │  │
        │  │ /images/{id}/download            │  │
        │  └─────────────────────────────────┘  │
        │                                        │
        │  ┌─────────────────────────────────┐  │
        │  │  Сервисы обработки              │  │
        │  │  - ImageProcessor               │  │
        │  │  - FilterEngine                 │  │
        │  │  - FileStorage                  │  │
        │  └─────────────────────────────────┘  │
        └───────────────────────────────────────┘
                            │
         ┌──────────────────┼──────────────────┐
         │                  │                  │
         ▼                  ▼                  ▼
    ┌─────────┐      ┌──────────┐      ┌─────────────┐
    │  ФС     │      │  БД      │      │ Сервис CDN  │
    │ файлы   │      │ Metadata │      │ хранилище   │
    │изображ. │      │ пользов. │      │ изображений │
    └─────────┘      └──────────┘      └─────────────┘
```

## Компоненты системы

### 1. Frontend (React приложение)

**Назначение:** пользовательский интерфейс для редактирования изображений.

**Основные папки:**

- `components/` — React компоненты (Editor, Toolbar, FilterPanel, Preview)
- `hooks/` — собственные хуки для логики (useImage, useFilters, useHistory)
- `services/` — сервисы взаимодействия с API (api.js, imageProcessing.js)
- `store/` — Redux хранилище состояния
- `styles/` — CSS стили и темы
- `utils/` — утилиты и константы

**Технологии:**
- React 18.x
- Redux для управления состоянием
- Axios для HTTP запросов
- Canvas API для обработки изображений

### 2. Backend API (Node.js + Express)

**Назначение:** обработка запросов, управление изображениями, применение фильтров.

**Основные маршруты:**

```
POST   /api/v1/images/upload          # Загрузить изображение
POST   /api/v1/images/{id}/filters    # Применить фильтры
POST   /api/v1/images/{id}/crop       # Обрезать изображение
GET    /api/v1/images/{id}/download   # Скачать изображение
GET    /api/v1/health                 # Проверка здоровья сервера
```

**Структура папок:**

```
backend/
├── routes/          # Express маршруты
├── controllers/     # Обработчики запросов
├── services/        # Бизнес-логика (обработка изображений)
├── middleware/      # Middleware (аутентификация, валидация)
├── models/          # Модели БД
├── config/          # Конфигурация
└── utils/           # Утилиты
```

### 3. Хранилище изображений

**Назначение:** сохранение загруженных и отредактированных изображений.

**Варианты:**
- **Локальная файловая система** (для разработки) — `uploads/`
- **Amazon S3** или **Google Cloud Storage** (для продакшена)
- **CDN** для быстрой доставки пользователям

### 4. База данных

**Назначение:** сохранение метаданных пользователей и изображений.

**Модели:**

```
Users
├── id (UUID)
├── email
├── created_at
└── updated_at

Images
├── id (UUID)
├── user_id (FK)
├── filename
├── size (bytes)
├── format (jpeg, png, etc)
├── width, height
├── storage_path
├── created_at
└── updated_at

ImageEdits
├── id (UUID)
├── image_id (FK)
├── filters (JSON)
├── crop_data (JSON)
├── created_at
└── updated_at
```

## Поток данных

### Сценарий: Загрузить и отредактировать изображение

```
1. Пользователь нажимает "Загрузить фото"
   │
2. Браузер выбирает файл
   │
3. React компонент захватывает файл
   │
4. Axios POST запрос на /images/upload
   │
5. Backend принимает файл
   │
   ├─→ Валидирует формат и размер
   ├─→ Сохраняет на диск или в S3
   ├─→ Создаёт запись в БД
   └─→ Возвращает JSON с image_id, url
   │
6. Фронтенд получает ответ и отобразит изображение
   │
7. Пользователь сдвигает слайдер яркости
   │
8. Фронтенд обновляет превью (Canvas) локально
   │
9. Пользователь нажимает "Применить фильтры"
   │
10. Axios POST запрос на /images/{id}/filters с параметрами
    │
11. Backend обрабатывает изображение
    │
    ├─→ Загружает оригинал с диска/S3
    ├─→ Применяет фильтры (яркость, контраст и т.д.)
    ├─→ Сохраняет отредактированную копию
    └─→ Возвращает JSON с новым URL
    │
12. Фронтенд обновляет превью результатом
    │
13. Пользователь нажимает "Скачать"
    │
14. Браузер скачивает отредактированный файл
```

## Обработка изображений

### Используемые библиотеки

**Frontend:**
- **Canvas API** — встроенный в браузер для быстрой обработки пикселей
- **OpenCV.js** (опционально) — для сложных операций (детектирование лиц и т.д.)

**Backend:**
- **Sharp** (Node.js) — быстрая обработка изображений
- **ImageMagick** (если нужна максимальная гибкость)
- **Pillow** (Python) — если микросервис обработки на Python

### Применение фильтра (пример)

```javascript
// Backend: services/imageProcessor.js
const sharp = require('sharp');

async function applyFilters(imagePath, filters) {
  let image = sharp(imagePath);

  if (filters.brightness !== 0) {
    image = image.modulate({
      brightness: 1 + filters.brightness / 100,
    });
  }

  if (filters.contrast !== 0) {
    // Используем linear для контраста
    image = image.modulate({
      brightness: 1 + filters.contrast / 200,
    });
  }

  if (filters.saturation !== 0) {
    image = image.modulate({
      saturation: 1 + filters.saturation / 100,
    });
  }

  if (filters.blur > 0) {
    image = image.blur(filters.blur);
  }

  if (filters.grayscale) {
    image = image.grayscale();
  }

  if (filters.sepia) {
    // Применить матрицу сепии
    image = image.tint({ r: 112, g: 66, b: 20 });
  }

  return image;
}
```

## Масштабирование

### Для роста числа пользователей

1. **Разделить фронтенд и бэкенд** — развертывать отдельно
2. **Использовать CDN** (CloudFlare, Akamai) для раздачи статики
3. **Кэшировать изображения** на уровне HTTP (ETag, Cache-Control)
4. **Очередь задач** (Bull, RabbitMQ) для асинхронной обработки
5. **Микросервисы** — отдельный сервис для тяжелой обработки
6. **Load Balancer** (nginx, AWS ELB) для распределения нагрузки
7. **Репликация БД** — master-slave для чтения

### Оптимизация производительности

- **WebP формат** — более компактный, чем JPG
- **Ленивая загрузка** превью (lazy loading)
- **Сжатие статики** через gzip/brotli
- **Мониторинг** (New Relic, Datadog) для отслеживания узких мест
- **Профилирование** — найти медленные функции

## Безопасность

1. **HTTPS** — обязателен
2. **Валидация входных данных** — размер файла, формат, содержимое
3. **CORS** — ограничить запросы с нужных доменов
4. **Authentication** — JWT токены, OAuth 2.0
5. **Rate limiting** — ограничить количество запросов в минуту
6. **Сканирование файлов** — ClamAV для проверки вирусов
7. **Санитизация пути** — предотвратить path traversal атаки

## Мониторинг и логирование

### Логирование

```javascript
// services/logger.js
const logger = {
  info: (msg, data) => console.log(`[INFO] ${msg}`, data),
  error: (msg, error) => console.error(`[ERROR] ${msg}`, error),
  warn: (msg, data) => console.warn(`[WARN] ${msg}`, data),
};

// Использование
logger.info('Image uploaded', { imageId, size: 1024000 });
logger.error('Filter processing failed', error);
```

### Метрики

- Время обработки запроса
- Количество загруженных изображений в день
- Размер обработанных изображений (МБ)
- Ошибки и исключения

## Развёртывание

### Development
- Локально на компьютере (npm start)
- Hot reload при изменении кода

### Staging
- На тестовом сервере перед продакшеном
- Полная проверка функциональности

### Production
- На боевом сервере с максимальной надёжностью
- Docker контейнеры, Kubernetes оркестрация
- CI/CD pipeline (GitHub Actions, GitLab CI)

## Ссылки на компоненты

- **Frontend Repository:** https://github.com/photoeditor/photo-editor-web
- **Backend Repository:** https://github.com/photoeditor/photo-editor-api
- **Infrastructure as Code:** https://github.com/photoeditor/photo-editor-infra

## Контакты

- **Архитектор:** architect@photoeditor.example.com
- **DevOps:** devops@photoeditor.example.com
- **Канал Slack:** #photo-editor-dev
