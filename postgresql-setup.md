# 🐘 PostgreSQL Setup Guide (Ubuntu)

This guide walks you through installing PostgreSQL on Ubuntu, configuring users, enabling remote access, and applying basic security best practices.

---

## 📦 Installation

### Install Latest Version

```bash
sudo apt update
sudo apt install -y postgresql
```

---

### Install Specific Version

#### Option 1: Automatic Repository Configuration

```bash
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

#### Option 2: Manual Repository Configuration

```bash
sudo apt install -y curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
. /etc/os-release
echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt ${VERSION_CODENAME}-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
sudo apt update
sudo apt install -y postgresql
```

---

### Check PostgreSQL Service Status

```bash
sudo systemctl status postgresql
```

---

## ⚙️ Basic Setup

### Set Postgres Password

```bash
sudo su - postgres
psql
```

Inside the `psql` prompt:

```sql
ALTER USER postgres WITH PASSWORD 'your_password';
```

---

### Create Roles and Users

#### Create Admin Role

```sql
CREATE ROLE admin WITH LOGIN SUPERUSER CREATEDB CREATEROLE INHERIT REPLICATION PASSWORD 'admin_password';
```

#### Create Regular User

```sql
CREATE USER your_user WITH PASSWORD 'user_password';
```

---

### Grant Permissions

```sql
-- Allow connection to the database
GRANT CONNECT ON DATABASE your_database TO your_user;

-- Grant access to public schema
GRANT USAGE ON SCHEMA public TO your_user;

-- Grant SELECT on existing tables
GRANT SELECT ON ALL TABLES IN SCHEMA public TO your_user;

-- Automatically grant SELECT on future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO your_user;
```

---

## 🌐 Enable Remote Connections

### Allow PostgreSQL to Listen on All IPs

Edit the config file:

```bash
sudo nano /etc/postgresql/*/main/postgresql.conf
```

Find and update:

```conf
listen_addresses = '*'
```

---

### Allow Remote Logins

Edit the authentication file:

```bash
sudo nano /etc/postgresql/*/main/pg_hba.conf
```

Add:

```conf
host    all             all             0.0.0.0/0               scram-sha-256
```

> ⚠️ Use a specific IP range instead of `0.0.0.0/0` for production environments.

---

### Apply Changes

```bash
sudo systemctl restart postgresql
```

---

## 🔐 Security Notes

- ✅ Prefer `scram-sha-256` over `md5` or `trust` authentication.
- ✅ Use firewalls and restrict incoming access to known IPs.
- ✅ Disable or secure the default `postgres` role if not needed.
- ✅ Enable SSL for encrypted connections when accessing remotely.
- ✅ Monitor and rotate credentials regularly.

---

## 🛠 Useful Commands

### PostgreSQL Prompt

```bash
sudo -u postgres psql
```

### Inside psql

```sql
-- List roles
\du

-- List databases
\l

-- Connect to a database
\c your_database

-- List schemas and tables
\dn
\dt
```

### Connect From a Client Machine

```bash
psql -h <server_ip> -U your_user -d your_database -W
```

---

## 💾 Backup & Restore (Optional)

### Backup

```bash
pg_dump -U your_user -d your_database -F c -f db.backup
```

### Restore

```bash
pg_restore -U your_user -d new_database db.backup
```

> 📝 You must create the target database (`new_database`) beforehand.

---

## 📖 Resources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [pg_hba.conf Explained](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)
- [pgAdmin (GUI Tool)](https://www.pgadmin.org/)
- [PostgreSQL Cheat Sheet](https://github.com/enochtangg/quick-SQL-cheatsheet)

---
