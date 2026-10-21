# Django Starter Template with PostgreSQL, MinIO (S3) & Docker

[![Python Version](https://img.shields.io/badge/python-3.12%20%7C%203.14-blue.svg)](https://www.python.org/)
[![Django Version](https://img.shields.io/badge/django-6.1.2-green.svg)](https://www.djangoproject.com/)
[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?logo=docker&logoColor=white)](https://www.docker.com/)
[![Database](https://img.shields.io/badge/database-PostgreSQL-336791.svg)](https://www.postgresql.org/)
[![Storage](https://img.shields.io/badge/storage-MinIO%20%2F%20S3-C72C48.svg)](https://min.io/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A production-ready Django starter template pre-configured with **Docker & Docker Compose**, **PostgreSQL** (via `psycopg 3`), **MinIO / AWS S3** object storage (via `django-storages` and `boto3`), secure signed URLs, and decoupled environment configuration.

---

## Table of Contents

- [Features](#features)
- [Project Architecture](#project-architecture)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Environment Configuration](#environment-configuration)
- [Quickstart with Docker (Recommended)](#quickstart-with-docker-recommended)
- [Manual Setup (Without Docker)](#manual-setup-without-docker)
- [Docker Architecture & Services](#docker-architecture--services)
- [MinIO Bucket Setup](#minio-bucket-setup)
- [Media & Storage Architecture](#media--storage-architecture)
- [Production Recommendations](#production-recommendations)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- **Fully Containerized with Docker**: Pre-configured `Dockerfile` and `docker-compose.yaml` with live code reloading for seamless development.
- **Modern Django (6.x)**: Clean project layout separating configuration (`root/`) and modular applications (`apps/`).
- **PostgreSQL Database**: Configured with PostgreSQL 15 in Docker and the modern `psycopg 3` database adapter.
- **MinIO / AWS S3 Object Storage**: Configured with `django-storages` and `boto3` for scalable cloud file storage.
- **Secure File Access**: Media files use private access with temporary signed URLs (querystring authorization with 1-hour expiration).
- **Decoupled Settings**: Secure environment-based configuration via `python-decouple`.
- **Pre-structured Media Organization**: Ready-to-use directory paths for profile pictures, documents, and temporary files.

---

## Project Architecture

```text
DjangoJinjaTemplateDocker/
├── apps/                     # Modular Django application(s)
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── django_media/             # Media storage structure
│   ├── documents/
│   ├── profile-pictures/
│   └── temp/
├── root/                     # Project configuration & settings
│   ├── asgi.py
│   ├── settings.py           # Core settings with decouple & storage setup
│   ├── storages.py           # Custom S3 / MinIO storage backends
│   ├── urls.py
│   └── wsgi.py
├── templates/                # Global template directory
├── .env_exmple               # Template for environment variables
├── .gitignore
├── docker-compose.yaml       # Docker Compose multi-container setup
├── Dockerfile                # Web service container specification
├── manage.py
└── requirement.txt           # Project dependencies
```

---

## Tech Stack

- **Containerization**: [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)
- **Framework**: [Django 6.1.2](https://www.djangoproject.com/)
- **Database**: [PostgreSQL 15](https://www.postgresql.org/) with [psycopg 3.3.6](https://www.psycopg.org/)
- **Storage / S3 Client**: [django-storages 1.14.6](https://django-storages.readthedocs.io/) & [boto3 1.43.x](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- **Configuration**: [python-decouple 3.8](https://github.com/HBNetwork/python-decouple)

---

## Prerequisites

Make sure you have installed:
- [Docker](https://docs.docker.com/get-docker/) & [Docker Compose](https://docs.docker.com/compose/install/)
- [Git](https://git-scm.com/)
- *(Optional for manual setup)* Python 3.12+

---

## Environment Configuration

1. Clone the repository:
   ```bash
   git clone https://github.com/Xadjime/DjangoJinjaTemplateDocker.git
   cd DjangoJinjaTemplateDocker
   ```

2. Copy the example environment file and create your `.env`:
   ```bash
   cp .env_exmple .env
   ```

3. Configure your `.env` variables:
   ```ini
   # MinIO / S3 Configuration
   MINIO_ENDPOINT=MINIO_ENDPOINT
   MINIO_ACCESS_KEY=MINIO_ACCESS_KEY
   MINIO_SECRET_KEY=MINIO_SECRET_KEY
   MINIO_BUCKET_NAME=MINIO_BUCKET_NAME
   MINIO_USE_HTTPS=False

   # PostgreSQL Configuration (used for local manual setup or external DB)
   DB_NAME=DB_NAME
   DB_USER=DB_USER
   DB_PASSWORD=DB_PASSWORD
   DB_HOST=DB_HOST
   DB_PORT=DB_PORT
   ```

### Environment Variables Breakdown

| Variable | Description | Default / Example |
| :--- | :--- | :--- |
| `MINIO_ENDPOINT` | MinIO or S3 endpoint host and port | `localhost:9000` or `s3.amazonaws.com` |
| `MINIO_ACCESS_KEY` | MinIO Access Key (or AWS Access Key ID) | `MINIO_ACCESS_KEY` |
| `MINIO_SECRET_KEY` | MinIO Secret Key (or AWS Secret Access Key)| `MINIO_SECRET_KEY` |
| `MINIO_BUCKET_NAME`| Target bucket name for uploaded media files| `MINIO_BUCKET_NAME` |
| `MINIO_USE_HTTPS` | Whether to use SSL/HTTPS connection | `False` |
| `DB_NAME` | PostgreSQL database name | `DB_NAME` |
| `DB_USER` | PostgreSQL user | `DB_USER` |
| `DB_PASSWORD` | PostgreSQL user password | `DB_PASSWORD` |
| `DB_HOST` | Database host (`db` within Docker, `localhost` locally) | `db` |
| `DB_PORT` | PostgreSQL port | `5432` |

---

## Quickstart with Docker (Recommended)

Run the entire stack with a single command:

### 1. Build and Start the Containers

```bash
docker compose up --build -d
```

This starts:
- **`web`**: Django application running at `http://localhost:8000` with hot-reload enabled.
- **`db`**: PostgreSQL 15 database exposed on host port `5433` (internal port `5432`).

### 2. Apply Database Migrations

```bash
docker compose exec web python manage.py migrate
```

### 3. Create a Superuser

```bash
docker compose exec web python manage.py createsuperuser
```

### 4. Access the Application

- **Web Application**: [http://localhost:8000/](http://localhost:8000/)
- **Django Admin**: [http://localhost:8000/admin/](http://localhost:8000/admin/)

### Useful Docker Commands

```bash
# View real-time logs
docker compose logs -f web

# Run migrations
docker compose exec web python manage.py makemigrations
docker compose exec web python manage.py migrate

# Open a bash shell inside the web container
docker compose exec web bash

# Open PostgreSQL shell (psql)
docker compose exec db psql -U postgres -d Django_Jinja_Docker_db

# Stop all containers
docker compose down

# Stop all containers and remove database volume
docker compose down -v
```

---

## Manual Setup (Without Docker)

If you prefer to run Python directly on your host machine:

### 1. Create and Activate a Virtual Environment

```bash
# Linux / macOS
python3 -m venv .venv
source .venv/bin/activate

# Windows (Command Prompt)
python -m venv .venv
.venv\Scripts\activate.bat

# Windows (PowerShell)
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 2. Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirement.txt
```

### 3. Configure Database Connection

In `.env`, set `DB_HOST=localhost` and ensure your PostgreSQL instance is running on `DB_PORT=5432` (or `5433` if connecting to the Dockerized DB).

### 4. Run Migrations & Start Server

```bash
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

---

## Docker Architecture & Services

The application is orchestrated via `docker-compose.yaml`:

| Service | Base Image | Ports | Description |
| :--- | :--- | :--- | :--- |
| **`web`** | `python:3.12-slim` | `8000:8000` | Django web service with live volume mounting (`.:/app`) |
| **`db`** | `postgres:15-alpine` | `5433:5432` | PostgreSQL database with persistent volume `postgres_data` |

### Adding MinIO to Docker Compose (Optional)

If you would like to run MinIO inside the same Docker Compose stack, you can append the following service to `docker-compose.yaml`:

```yaml
  minio:
    image: minio/minio:latest
    container_name: minio_storage
    restart: unless-stopped
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: MINIO_ROOT_USER
      MINIO_ROOT_PASSWORD: MINIO_ROOT_PASSWORD
    ports:
      - "9000:9000"  # S3 API
      - "9001:9001"  # Web Console
    volumes:
      - minio_data:/data
```

And add `minio_data:` under `volumes:`.

---

## MinIO Bucket Setup

1. Open the MinIO web console in your browser: [http://localhost:9001](http://localhost:9001).
2. Log in with your MinIO credentials:
   - **Username**: `Username`
   - **Password**: `Password`
3. Navigate to **Buckets** -> **Create Bucket**.
4. Enter the bucket name specified in your `.env` (e.g. `my-django-bucket`).
5. Click **Create**.

---

## Media & Storage Architecture

Custom storage backends are defined in `root/storages.py`:

- **`MediaStorage`**: Handles user uploaded files (`FileField`, `ImageField`).
  - Stored under the `media/` prefix in your bucket.
  - Configured with `AWS_DEFAULT_ACL = 'private'` and `AWS_QUERYSTRING_AUTH = True`.
  - When accessing a file via `.url`, Django generates a cryptographically signed URL valid for 3600 seconds (1 hour).
- **`StaticStorage`**: Handles static assets if remote collection is needed.

---

## Production Recommendations

Before deploying to production:
1. Set `DEBUG=False` in `root/settings.py`.
2. Generate a secure, unique `SECRET_KEY` and store it in `.env`.
3. Set your production domain in `ALLOWED_HOSTS`.
4. Enable HTTPS for your S3/MinIO endpoint (`MINIO_USE_HTTPS=True`).
5. Replace `runserver` in Docker with a production WSGI/ASGI server like **Gunicorn** or **Uvicorn** behind an **Nginx** reverse proxy.

---

## Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the [issues page](https://github.com/<your-username>/DjangoJinjaTemplateDocker/issues).

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
