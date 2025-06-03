# 🧰 Data Platform Setup Guides

This repository contains complete setup guides for building and managing a modern analytics stack on Ubuntu. Each guide is designed to help you install, configure, and secure popular tools used in data engineering, analytics, and visualization.

## 📚 Available Guides

| Tool             | Description                                               |
|------------------|-----------------------------------------------------------|
| [ClickHouse](./clickhouse-setup.md) | High-performance columnar OLAP database, optimized for real-time analytics. |
| [PostgreSQL](./postgresql-setup.md) | Powerful open-source relational database for transactional workloads.        |
| [Apache Superset](./superset-setup.md) | Modern data exploration and visualization platform.                        |
| [Apache Airflow](./airflow_setup/airflow-setup.md) | Workflow orchestration tool for authoring, scheduling, and monitoring ETL pipelines. |

---

## 🛠️ Recommended Environment

- Ubuntu Server 22.04 or 24.04
- sudo/root privileges
- Internet access for package installation
- (Optional) Corporate proxy configured if required

---

## ✅ Setup Order Recommendation

If you are setting up the full stack, the suggested order is:

1. **PostgreSQL** – used as a metadata store for Superset and optionally Airflow.
2. **ClickHouse** – data warehouse for high-speed analytical queries.
3. **Apache Superset** – connects to PostgreSQL/ClickHouse for visualization.
4. **Apache Airflow** – optional, for orchestrating ETL pipelines into these databases.

---

## 🧾 How to Use the Guides

Each guide is a standalone Markdown file that includes:

- Installation steps
- Secure configuration tips
- User and role management
- Optional performance and access tweaks
- Testing instructions

You can open any guide directly in GitHub or clone the repo and read them locally.

```bash
git clone https://git@github.com:DanFelCas/setups-data-engineering.git
cd setups-data-engineering
```

---
