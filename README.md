# Smart Notes (Интеллектуальная система «Умный конспект»)

[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-009688.svg)](https://fastapi.tiangolo.com)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.2+-ee4c2c.svg)](https://pytorch.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.9+-5C3EE8.svg)](https://opencv.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16+-336791.svg)](https://www.postgresql.org/)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.32+-FF4B4B.svg)](https://streamlit.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

«Умный конспект» — это комплексная распределенная программная система корпоративного уровня для автоматизированного распознавания, очистки от оптических искажений, сегментации рукописных строк и семантического структурирования рукописных конспектов, лекций и математических формул в формат Markdown / LaTeX / TXT.

---



---

## 3. Матрица угроз и стратегии защиты (STRIDE / OWASP)

| Угроза / Вектор атаки | Категория | Риск | Архитектурное решение / Механизм защиты |
|---|---|---|---|
| **DoS / OOM через загрузку гигантских файлов** | Denial of Service | Высокий | Потоковая буферизация `StreamingUpload` в FastAPI с ограничением максимального размера (по умолчанию 25 МБ). Принудительное ограничение разрешения (макс. 4096px по длинной стороне) с пропорциональным ресайзом в памяти. |
| **Декомпрессионная бомба (Image Decompression Bomb)** | Denial of Service | Высокий | Защита на уровне PIL / OpenCV: проверка заголовков файла, лимит пикселей `Image.MAX_IMAGE_PIXELS = 64_000_000` и перехват исключений поврежденных буферов. |
| **Инъекции в SQL-запросы** | Tampering / Information Disclosure | Критический | Применение SQLAlchemy 2.0 ORM и параметризованных запросов с типизированными моделями. Отсутствие динамической интерполяции сырых строк в SQL. |
| **Path Traversal / Local File Inclusion** | Elevation of Privilege | Критический | Генерация случайных UUIDv4 имен файлов при сохранении на диск. Изоляция хранилища в каталоге `STORAGE_DIR`, строгая санитизация путей через `pathlib.Path.resolve()` и проверка выхода за пределы корня хранилища. |
| **Переполнение памяти видеокарты / RAM при инференсе** | Denial of Service | Высокий | Батчирование нарезанных строк с фиксированным `batch_size`, автоматическое переключение `torch.no_grad()`, очистка кеша PyTorch `torch.cuda.empty_cache()` в блоках `finally`. |
| **Некорректная бинаризация при неравномерном освещении** | Fault Tolerance | Средний | Двухэтапная нормализация: вычисление фона морфологическим закрытием/открытием, деление яркостного канала на оценку фона, адаптивный порог Гаусса с локальным окном. |

---

## 4. Быстрый запуск системы

### Вариант 1: Локальный запуск для разработки

```bash
# 1. Создание виртуального окружения
python3 -m venv venv
source venv/bin/activate

# 2. Установка зависимостей
pip install -r requirements.txt

# 3. Инициализация весов и базы данных
python3 scripts/generate_synthetic_weights.py

# 4. Запуск FastAPI бэкенда
uvicorn app.api.main:app --host 0.0.0.0 --port 8000 --reload

# 5. Запуск Streamlit фронтенда (в отдельном терминале)
streamlit run frontend/app.py --server.port 8501
```

---

## 5. Тестирование системы


```bash
# Запуск полного набора модульных и интеграционных тестов
pytest tests/ -v
```
