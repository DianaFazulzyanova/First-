# Photo Editor API — Документация

## Обзор

**Photo Editor API** предоставляет REST-эндпоинты для программного редактирования изображений. Используйте API для автоматизации обработки фотографий в ваших приложениях.

## Базовая информация

- **Базовый URL:** `https://api.photoeditor.example.com/v1`
- **Аутентификация:** API Key (передавайте в заголовке `Authorization: Bearer YOUR_API_KEY`)
- **Формат ответа:** JSON
- **Лимит запросов:** 1000 запросов/час на один API Key

## Эндпоинты

### 1. Загрузить изображение

**POST** `/images/upload`

Загружает изображение на сервер.

**Параметры запроса (Form Data):**

| Параметр | Тип | Обязательный | Описание |
|----------|-----|--------------|---------|
| `file` | File | Да | JPG, PNG, GIF, WebP (макс. 20 МБ) |
| `public` | Boolean | Нет | Доступно ли изображение публично (по умолчанию `false`) |

**Пример запроса:**

```bash
curl -X POST https://api.photoeditor.example.com/v1/images/upload \
  -H "Authorization: Bearer your_api_key" \
  -F "file=@photo.jpg" \
  -F "public=false"
```

**Пример ответа (200 OK):**

```json
{
  "id": "img_12345abc",
  "filename": "photo.jpg",
  "size": 2048576,
  "format": "jpeg",
  "width": 1920,
  "height": 1080,
  "url": "https://storage.photoeditor.example.com/img_12345abc.jpg",
  "created_at": "2025-11-28T10:30:00Z"
}
```

**Коды ошибок:**
- `400` — Файл не выбран или неподдерживаемый формат.
- `413` — Файл слишком большой (> 20 МБ).
- `401` — Не авторизованы (неверный или отсутствующий API Key).

---

### 2. Применить фильтры

**POST** `/images/{image_id}/filters`

Применяет фильтры к загруженному изображению.

**Параметры пути:**

| Параметр | Тип | Описание |
|----------|-----|---------|
| `image_id` | String | ID изображения (из ответа загрузки) |

**Параметры тела запроса (JSON):**

| Параметр | Тип | Описание |
|----------|-----|---------|
| `brightness` | Integer | -100 до +100 |
| `contrast` | Integer | -100 до +100 |
| `saturation` | Integer | -100 до +100 |
| `blur` | Integer | 0–20 пикселей |
| `grayscale` | Boolean | Чёрно-белое |
| `sepia` | Boolean | Сепия |

**Пример запроса:**

```bash
curl -X POST https://api.photoeditor.example.com/v1/images/img_12345abc/filters \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "brightness": 20,
    "contrast": 10,
    "saturation": -15,
    "grayscale": false
  }'
```

**Пример ответа (200 OK):**

```json
{
  "id": "img_12345abc",
  "filters": {
    "brightness": 20,
    "contrast": 10,
    "saturation": -15,
    "blur": 0,
    "grayscale": false,
    "sepia": false
  },
  "preview_url": "https://storage.photoeditor.example.com/img_12345abc_preview.jpg",
  "applied_at": "2025-11-28T10:35:00Z"
}
```

---

### 3. Обрезать изображение

**POST** `/images/{image_id}/crop`

Обрезает изображение по указанным координатам.

**Параметры тела запроса (JSON):**

| Параметр | Тип | Описание |
|----------|-----|---------|
| `x` | Integer | X-координата верхнего левого угла (в пикселях) |
| `y` | Integer | Y-координата верхнего левого угла (в пикселях) |
| `width` | Integer | Ширина области (в пикселях) |
| `height` | Integer | Высота области (в пикселях) |

**Пример запроса:**

```bash
curl -X POST https://api.photoeditor.example.com/v1/images/img_12345abc/crop \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "x": 100,
    "y": 50,
    "width": 800,
    "height": 600
  }'
```

**Пример ответа (200 OK):**

```json
{
  "id": "img_12345abc",
  "width": 800,
  "height": 600,
  "crop": {
    "x": 100,
    "y": 50,
    "width": 800,
    "height": 600
  },
  "url": "https://storage.photoeditor.example.com/img_12345abc_cropped.jpg"
}
```

---

### 4. Скачать отредактированное изображение

**GET** `/images/{image_id}/download`

Скачивает отредактированное изображение.

**Параметры запроса (Query):**

| Параметр | Тип | Описание |
|----------|-----|---------|
| `format` | String | Формат: `jpeg`, `png`, `webp` (по умолчанию `jpeg`) |
| `quality` | Integer | Качество: 60–100 (по умолчанию 85) |

**Пример запроса:**

```bash
curl -X GET "https://api.photoeditor.example.com/v1/images/img_12345abc/download?format=png&quality=90" \
  -H "Authorization: Bearer your_api_key" \
  -o edited_photo.png
```

**Ответ:** Бинарные данные изображения (Content-Type: image/jpeg, image/png или image/webp).

---

## Коды ошибок (глобальные)

| Код | Описание |
|-----|---------|
| `400` | Некорректный запрос (неверные параметры) |
| `401` | Не авторизованы |
| `403` | Доступ запрещён |
| `404` | Ресурс не найден |
| `429` | Превышен лимит запросов |
| `500` | Ошибка сервера |

## Примеры на Python

```python
import requests
import json

API_KEY = "your_api_key"
BASE_URL = "https://api.photoeditor.example.com/v1"
HEADERS = {"Authorization": f"Bearer {API_KEY}"}

# Загрузить изображение
with open("photo.jpg", "rb") as f:
    files = {"file": f}
    resp = requests.post(f"{BASE_URL}/images/upload", headers=HEADERS, files=files)
    image_id = resp.json()["id"]
    print(f"Загружено: {image_id}")

# Применить фильтры
filters_data = {"brightness": 20, "contrast": 10}
resp = requests.post(
    f"{BASE_URL}/images/{image_id}/filters",
    headers=HEADERS,
    json=filters_data
)
print(f"Фильтры применены: {resp.json()}")

# Скачать результат
resp = requests.get(
    f"{BASE_URL}/images/{image_id}/download?format=png&quality=90",
    headers=HEADERS
)
with open("edited_photo.png", "wb") as f:
    f.write(resp.content)
print("Изображение сохранено")
```

## Поддержка

- **Email:** api-support@photoeditor.example.com
- **Документация:** https://docs.photoeditor.example.com
- **Статус сервера:** https://status.photoeditor.example.com
