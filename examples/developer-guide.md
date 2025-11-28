# Photo Editor Web App — Руководство для разработчиков

## Введение

Это руководство предназначено для разработчиков, которые хотят расширять, модифицировать или делать контрибьюции в проект **Photo Editor Web App**.

## Архитектура проекта

```
photo-editor-web/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── manifest.json
├── src/
│   ├── components/
│   │   ├── Editor/
│   │   │   ├── Editor.jsx
│   │   │   └── Editor.css
│   │   ├── Toolbar/
│   │   ├── FilterPanel/
│   │   └── Preview/
│   ├── hooks/
│   │   ├── useImage.js
│   │   ├── useFilters.js
│   │   └── useHistory.js
│   ├── services/
│   │   ├── api.js
│   │   ├── imageProcessing.js
│   │   └── storage.js
│   ├── utils/
│   │   ├── constants.js
│   │   ├── helpers.js
│   │   └── validators.js
│   ├── styles/
│   │   └── index.css
│   ├── App.jsx
│   └── index.jsx
├── tests/
│   ├── components/
│   ├── hooks/
│   └── utils/
├── docs/
│   ├── API.md
│   ├── ARCHITECTURE.md
│   └── CONTRIBUTING.md
├── .env.example
├── .gitignore
├── package.json
├── Dockerfile
└── README.md
```

## Стек технологий

| Технология | Версия | Назначение |
|------------|--------|-----------|
| React | 18.x | UI-фреймворк |
| Redux | 4.x | Управление состоянием |
| Axios | 0.27+ | HTTP-клиент |
| Webpack | 5.x | Сборщик модулей |
| Jest | 27+ | Тестирование |
| ESLint | 8.x | Линтинг |
| Babel | 7.x | Транспиляция |

## Настройка окружения разработки

### Шаг 1: Установить инструменты

```bash
# Node.js (https://nodejs.org)
node --version  # должно быть 16.0.0+

# Git
git --version

# Редактор кода (VS Code рекомендуется)
code --version
```

### Шаг 2: Клонировать и установить

```bash
git clone https://github.com/photoeditor/photo-editor-web.git
cd photo-editor-web
npm install
```

### Шаг 3: Установить расширения VS Code (опционально)

- **ES7+ React/Redux/React-Native snippets** (dsznajder.es7-react-js-snippets)
- **ESLint** (dbaeumer.vscode-eslint)
- **Prettier** (esbenp.prettier-vscode)
- **Jest** (firsttris.vscode-jest-runner)

### Шаг 4: Запустить в режиме разработки

```bash
npm start
```

Приложение откроется на `http://localhost:3000` с горячей перезагрузкой.

## Структура компонентов

### Главные компоненты

#### `Editor.jsx`

Основной компонент для редактирования изображений.

```jsx
// src/components/Editor/Editor.jsx
import React, { useState } from 'react';
import { useImage } from '../../hooks/useImage';
import Toolbar from '../Toolbar/Toolbar';
import FilterPanel from '../FilterPanel/FilterPanel';
import Preview from '../Preview/Preview';

const Editor = () => {
  const { image, loadImage, updateImage } = useImage();
  const [filters, setFilters] = useState({});

  const handleImageLoad = (file) => {
    loadImage(file);
  };

  const handleFilterChange = (newFilters) => {
    setFilters(newFilters);
    updateImage(newFilters);
  };

  return (
    <div className="editor">
      <Toolbar onImageLoad={handleImageLoad} />
      <div className="editor-container">
        <Preview image={image} filters={filters} />
        <FilterPanel onFilterChange={handleFilterChange} />
      </div>
    </div>
  );
};

export default Editor;
```

### Собственные хуки (Custom Hooks)

#### `useImage.js`

Управляет состоянием изображения.

```jsx
// src/hooks/useImage.js
import { useState } from 'react';
import { canvasToBlob } from '../utils/helpers';

export const useImage = () => {
  const [image, setImage] = useState(null);

  const loadImage = (file) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => setImage(img);
      img.src = e.target.result;
    };
    reader.readAsDataURL(file);
  };

  const updateImage = (filters) => {
    // логика применения фильтров
  };

  return { image, loadImage, updateImage };
};
```

## Работа с API

### `services/api.js`

```jsx
// src/services/api.js
import axios from 'axios';

const API = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
  headers: {
    Authorization: `Bearer ${process.env.REACT_APP_API_KEY}`,
  },
});

export const uploadImage = (file) => {
  const formData = new FormData();
  formData.append('file', file);
  return API.post('/images/upload', formData);
};

export const applyFilters = (imageId, filters) => {
  return API.post(`/images/${imageId}/filters`, filters);
};

export const downloadImage = (imageId, format = 'jpeg', quality = 85) => {
  return API.get(`/images/${imageId}/download`, {
    params: { format, quality },
    responseType: 'blob',
  });
};

export default API;
```

## Тестирование

### Запуск тестов

```bash
# Запустить все тесты
npm test

# Запустить в режиме watch
npm test -- --watch

# Проверить покрытие
npm test -- --coverage
```

### Пример теста

```jsx
// src/components/Editor/__tests__/Editor.test.jsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import Editor from '../Editor';

describe('Editor Component', () => {
  test('renders editor with toolbar', () => {
    render(<Editor />);
    const toolbar = screen.getByRole('button', { name: /загрузить/i });
    expect(toolbar).toBeInTheDocument();
  });

  test('loads image when file is selected', () => {
    render(<Editor />);
    const uploadButton = screen.getByRole('button', { name: /загрузить/i });
    fireEvent.click(uploadButton);
    // дополнительные проверки
  });
});
```

## Лучшие практики

### Кодирование

1. **Используйте функциональные компоненты** вместо классовых.
2. **Используйте хуки** для управления состоянием (useState, useEffect, useContext).
3. **Извлекайте логику в Custom Hooks** для переиспользования.
4. **Проверяйте PropTypes** или TypeScript для типизации.

```jsx
import PropTypes from 'prop-types';

const FilterPanel = ({ filters, onFilterChange }) => {
  // компонент
};

FilterPanel.propTypes = {
  filters: PropTypes.object.isRequired,
  onFilterChange: PropTypes.func.isRequired,
};
```

5. **Используйте CSS Modules или styled-components** для стилизации.

### Организация файлов

- **Один компонент = одна папка** с JSX, CSS и тестами.
- **Разделяйте логику** на слои: компоненты, хуки, сервисы, утилиты.
- **Используйте относительные импорты** в одной папке, абсолютные для разных.

### Документирование

```jsx
/**
 * Компонент для применения фильтров к изображению.
 * @param {Object} filters - текущие значения фильтров
 * @param {Function} onFilterChange - callback при изменении фильтра
 * @returns {JSX.Element}
 */
const FilterPanel = ({ filters, onFilterChange }) => {
  // ...
};
```

## Создание нового компонента

Пример: создание компонента фильтра яркости.

### 1. Создать папку компонента

```bash
mkdir -p src/components/BrightnessFilter
```

### 2. Создать файл компонента

```jsx
// src/components/BrightnessFilter/BrightnessFilter.jsx
import React from 'react';
import PropTypes from 'prop-types';
import './BrightnessFilter.css';

const BrightnessFilter = ({ value = 0, onChange }) => {
  const handleChange = (e) => {
    onChange(parseInt(e.target.value, 10));
  };

  return (
    <div className="brightness-filter">
      <label>Яркость</label>
      <input
        type="range"
        min="-100"
        max="100"
        value={value}
        onChange={handleChange}
      />
      <span>{value}</span>
    </div>
  );
};

BrightnessFilter.propTypes = {
  value: PropTypes.number,
  onChange: PropTypes.func.isRequired,
};

export default BrightnessFilter;
```

### 3. Создать стили

```css
/* src/components/BrightnessFilter/BrightnessFilter.css */
.brightness-filter {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 0;
}

.brightness-filter label {
  min-width: 80px;
  font-weight: 500;
}

.brightness-filter input[type="range"] {
  flex: 1;
}

.brightness-filter span {
  min-width: 40px;
  text-align: right;
}
```

### 4. Создать тест

```jsx
// src/components/BrightnessFilter/__tests__/BrightnessFilter.test.jsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import BrightnessFilter from '../BrightnessFilter';

describe('BrightnessFilter', () => {
  test('calls onChange with correct value', () => {
    const handleChange = jest.fn();
    render(<BrightnessFilter onChange={handleChange} />);
    
    const input = screen.getByRole('slider');
    fireEvent.change(input, { target: { value: '50' } });
    
    expect(handleChange).toHaveBeenCalledWith(50);
  });
});
```

## Команды npm

| Команда | Описание |
|---------|---------|
| `npm start` | Запуск в режиме разработки |
| `npm test` | Запуск тестов |
| `npm run build` | Собрать для продакшена |
| `npm run lint` | Проверить стиль кода |
| `npm run lint:fix` | Исправить ошибки стиля автоматически |
| `npm run eject` | Выкинуть конфигурацию (необратимо!) |

## Git workflow

```bash
# 1. Создать новую ветку
git checkout -b feature/add-brightness-filter

# 2. Сделать изменения и коммиты
git add .
git commit -m "feat: add brightness filter component"

# 3. Запушить ветку
git push origin feature/add-brightness-filter

# 4. Создать Pull Request на GitHub

# 5. После слияния удалить локальную ветку
git branch -d feature/add-brightness-filter
```

## Debugging

### React DevTools

Установите расширение **React Developer Tools** для браузера. Позволяет инспектировать компоненты и состояние.

### Redux DevTools

Используйте Redux DevTools для отладки состояния:

```bash
npm install --save-dev redux-devtools-extension
```

### Логирование

```jsx
// Используйте console для отладки
console.log('Image loaded:', image);
console.error('Error applying filters:', error);

// Или используйте специализированную библиотеку
import debug from 'debug';
const log = debug('app:editor');
log('Filters applied:', filters);
```

## Поддержка и ссылки

- **GitHub Issues:** https://github.com/photoeditor/photo-editor-web/issues
- **GitHub Discussions:** https://github.com/photoeditor/photo-editor-web/discussions
- **Email:** dev-support@photoeditor.example.com
- **React документация:** https://react.dev
- **Redux документация:** https://redux.js.org

## Примеры контрибьюции

Если хотите внести вклад:

1. Форкните репозиторий.
2. Создайте ветку (`git checkout -b feature/your-feature`).
3. Сделайте изменения с хорошей документацией.
4. Напишите или обновите тесты.
5. Создайте Pull Request с описанием.

Спасибо за помощь! 🎉
