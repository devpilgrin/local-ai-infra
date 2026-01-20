# local-ai-infra

Docker Compose инфраструктура для локального развертывания AI/ML стека с поддержкой GPU. Полнофункциональное окружение для разработки RAG-систем, обработки документов и работы с LLM.

## Описание

Готовый к использованию набор Docker-контейнеров для построения AI-инфраструктуры на локальной машине. Включает векторные базы данных, графовые БД, системы обработки документов, объектное хранилище и event streaming.

## Стек технологий

### Базы данных
- **PostgreSQL 16** — реляционная база данных
- **Qdrant** — векторная база данных для embeddings
- **Neo4j 5.15** — графовая БД для knowledge graphs (с плагином APOC)

### Обработка документов
- **Docling** — OCR и парсинг документов (поддержка CPU/NVIDIA GPU/AMD GPU)
- **Nginx** — статический файловый сервер для извлеченных изображений

### Хранилище и стриминг
- **MinIO** — S3-совместимое объектное хранилище
- **Apache Kafka 3.8.1** — брокер сообщений и event streaming
- **Kafka UI** — веб-интерфейс для управления Kafka

## Требования

- **Docker** >= 20.10
- **Docker Compose** >= 2.0
- **Свободное место на диске**: минимум 10 GB
- **Опционально для GPU**: NVIDIA Container Toolkit или ROCm (для AMD)

## Быстрый старт

### 1. Клонирование репозитория

```bash
git clone https://github.com/devpilgrin/local-ai-infra.git
cd local-ai-infra
```

### 2. Настройка переменных окружения

Создайте файл `.env` в корне проекта:

```env
# PostgreSQL
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_password
POSTGRES_DB=aidb

# MinIO
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=your_minio_password

# Neo4j
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_neo4j_password
```

### 3. Запуск инфраструктуры

#### С процессором (CPU)
```bash
docker compose --profile cpu up -d
```

#### С NVIDIA GPU
```bash
docker compose --profile gpu-nvidia up -d
```

#### С AMD GPU
```bash
docker compose --profile gpu-amd up -d
```

### 4. Проверка статуса

```bash
docker compose ps
```

## Доступ к сервисам

| Сервис | URL | Описание |
|--------|-----|----------|
| **Docling UI** | http://localhost:5001 | Интерфейс обработки документов |
| **Qdrant** | http://localhost:6333 | API векторной БД |
| **Neo4j Browser** | http://localhost:7474 | Веб-интерфейс Neo4j |
| **MinIO Console** | http://localhost:9001 | Управление объектным хранилищем |
| **MinIO API** | http://localhost:9000 | S3-совместимый API |
| **Kafka** | localhost:9092 | Брокер сообщений |
| **Kafka UI** | http://localhost:8081 | Управление Kafka |
| **Static Files** | http://localhost:8080 | Извлеченные изображения |
| **PostgreSQL** | localhost:5432 | База данных (внутренний доступ) |

## Структура данных

Все данные сохраняются в локальных директориях:

```
.
├── data/
│   ├── postgres/       # База данных PostgreSQL
│   ├── qdrant/         # Векторные данные
│   ├── neo4j/          # Графовая БД
│   ├── minio/          # Объектное хранилище
│   ├── kafka/          # Логи и данные Kafka
│   └── docling/        # Обработанные документы
└── shared/
    ├── docling-scratch/      # Временные файлы Docling
    └── extracted-images/     # Извлеченные изображения
```

## Использование

### MinIO Buckets

При первом запуске автоматически создаются бакеты:
- `ai-data` — общие данные для AI
- `models` — хранение моделей
- `documents` — загруженные документы

### Neo4j

Подключение через Bolt протокол:
```
bolt://localhost:7687
```

### Kafka Topics

Создавайте топики через Kafka UI или программно:
```bash
docker exec -it kafka kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic your-topic-name
```

## Конфигурация Docling

Сервис поддерживает три профиля:

- **cpu** — работает на CPU (для всех систем)
- **gpu-nvidia** — использует NVIDIA GPU (требует NVIDIA Container Toolkit)
- **gpu-amd** — использует AMD GPU через ROCm

Для GPU необходимо указать соответствующий профиль при запуске.

## Остановка сервисов

```bash
# Остановить все контейнеры
docker compose down

# Остановить и удалить все данные
docker compose down -v
```

## Health Checks

Все критичные сервисы имеют health checks:
- PostgreSQL: проверка готовности БД
- Kafka: проверка доступности порта
- MinIO: проверка API endpoint
- Neo4j: выполнение тестового Cypher-запроса
- Docling: проверка /health endpoint

## Troubleshooting

### Проблемы с правами доступа (Windows)

Docling запускается от root для корректной работы bind mount в Windows.

### Недостаточно памяти для Neo4j

Измените параметры в docker-compose.yml:
```yaml
NEO4J_dbms_memory_heap_max__size=1G  # уменьшите с 2G
```

### Kafka не запускается

Убедитесь, что порт 9092 свободен:
```bash
netstat -an | grep 9092
```
