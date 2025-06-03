# 🚀 Apache Airflow Setup Guide (Production-Ready with CeleryExecutor)

This guide walks you through deploying **Apache Airflow with Docker Compose**, using a **CeleryExecutor**, **PostgreSQL**, and **Redis**. It follows best practices for a production-like setup.

---

## 📦 1. Classic Installation (Official Apache Airflow Method)

The Apache Airflow team provides an official `docker-compose.yaml` for quick, production-like deployment.

### Step 1: Download the Official Docker Compose File

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.9.3/docker-compose.yaml'
```

> ⚙️ If you're using your own version, refer to [`docker-compose.yaml`](https://github.com/your-org/airflow-setup/blob/main/docker-compose.yaml)

### Step 2: Create Required Directories

```bash
mkdir -p ./dags ./logs ./plugins
```

> These will be mounted into the containers to persist DAGs, logs, and plugins.

### Step 3: Create Minimal `.env` File

Only the `AIRFLOW_UID` is needed for permission mapping:

```env
AIRFLOW_UID=$(id -u)
```

> Save this as `.env` in the root of the project.

### Step 4: Initialize the Airflow Database

```bash
docker compose up airflow-init
```

This sets up:
- Metadata DB
- Required tables
- Default admin user (hardcoded in compose file)

### Step 5: Start All Airflow Services

```bash
docker compose up
```

Or run in the background:

```bash
docker compose up -d
```

---

## 🔧 2. Custom Setup Requirements

If you prefer a more configurable or production-hardened setup, follow this custom layout.

### Prerequisites

- Docker & Docker Compose installed
- Full `.env` file (see below)
- Folder structure:

```
airflow-setup/
├── airflow/
│   ├── dags/
│   ├── logs/
│   ├── plugins/
│   └── config/
├── airflow_db_init.sql
├── Dockerfile.airflow
├── docker-compose.yaml
├── requirements.txt
└── .env
```

> 📁 Files:
> - [`docker-compose.yaml`](./docker-compose.yaml)
> - [`Dockerfile.airflow`](./Dockerfile.airflow)

---

### Full `.env` File for Custom Setup

```env
AIRFLOW_UID=50000

AIRFLOW_DB=airflow_db
AIRFLOW_DB_USER=airflow
AIRFLOW_DB_PASSWORD=airflow

FERNET_KEY=<your_generated_fernet_key>

_AIRFLOW_WWW_USER_USERNAME=admin
_AIRFLOW_WWW_USER_PASSWORD=admin
_AIRFLOW_WWW_USER_FIRSTNAME=Admin
_AIRFLOW_WWW_USER_LASTNAME=User
_AIRFLOW_WWW_USER_EMAIL=admin@example.com

POSTGRES_USER=airflow
POSTGRES_PASSWORD=airflow
```

> 🔐 **Generate Fernet Key**:
> ```bash
> python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
> ```

---

## 📄 3. `requirements.txt` Example

To include additional Python dependencies:

```txt
pandas==2.2.2
requests==2.31.0
boto3==1.34.103
apache-airflow-providers-postgres==5.10.0
apache-airflow-providers-amazon==8.15.0
```

In your [`Dockerfile.airflow`](https://github.com/your-org/airflow-setup/blob/main/Dockerfile.airflow):

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

---

## 🚀 4. Step-by-Step Deployment

### Step 1: (Optional) Initialize the Database via SQL

```sql
-- airflow_db_init.sql
CREATE USER airflow WITH PASSWORD 'airflow';
CREATE DATABASE airflow_db OWNER airflow;
```

> 📄 File: [`airflow_db_init.sql`](https://github.com/your-org/airflow-setup/blob/main/airflow_db_init.sql)

### Step 2: Build and Launch

```bash
docker compose up --build
```

This will:
- Build from [`Dockerfile.airflow`](https://github.com/your-org/airflow-setup/blob/main/Dockerfile.airflow)
- Use the environment from `.env`
- Start all Airflow components

---

## 🧩 5. Services Breakdown

| Service              | Role                                  | Port           |
|----------------------|---------------------------------------|----------------|
| `db`                 | PostgreSQL metadata DB                | 5432           |
| `redis`              | Celery message broker                 | 6379           |
| `airflow-webserver`  | Web UI                                | 8080           |
| `airflow-scheduler`  | DAG scheduling engine                 | —              |
| `airflow-worker`     | Executes tasks                        | —              |
| `airflow-triggerer`  | Executes deferrable operators         | —              |
| `flower`             | Optional Celery UI                    | 5555           |
| `airflow-init`       | Initialization + admin user creation  | —              |

---

## ✅ 6. Post-Setup Verification

- UI: [http://localhost:8080](http://localhost:8080)
- Logs:
  ```bash
  docker compose logs -f airflow-webserver
  ```
- Flower (if enabled): [http://localhost:5555](http://localhost:5555)

---

## 🧼 7. Cleanup

```bash
docker compose down -v
```

This will:
- Stop all services
- Remove volumes and networks

---

## 📁 8. Mounted Volumes

| Host Folder         | Container Path           | Purpose              |
|---------------------|--------------------------|-----------------------|
| `./airflow/dags/`   | `/opt/airflow/dags/`     | DAG definitions       |
| `./airflow/logs/`   | `/opt/airflow/logs/`     | Task logs             |
| `./airflow/plugins/`| `/opt/airflow/plugins/`  | Custom plugins        |
| `./airflow/config/` | `/opt/airflow/config/`   | Optional configs      |

---

## ⚠️ 9. Notes & Tips

- `.env` must match the compose file's expectations.
- DAGs auto-refresh without needing restarts.
- `airflow-init` is required for first-time setup.
- Wait for Redis/Postgres to be healthy before other services start.

---

## 📚 10. References

- [Apache Airflow Docs](https://airflow.apache.org/docs/)
- [CeleryExecutor Guide](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
