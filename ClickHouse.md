# ClickHose Setup Guide (Ubuntu)

This guide walks you through installing ClickHouse on Ubuntu, configuring users, enabling remote access, and applying basic security best practices.

---

## Installation

### Setup Debian Ropository 

- You can replace stable with lts to use different release kinds based on your needs.
- You can download and install packages manually from packages.clickhouse.com.

#### Option 1: Install Latest Version

```bash
# Install prerequisite packages
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# Download the ClickHouse GPG key and store it in the keyring
curl -fsSL 'https://packages.clickhouse.com/rpm/lts/repodata/repomd.xml.key' | sudo gpg --dearmor -o /usr/share/keyrings/clickhouse-keyring.gpg

# Get the system architecture
ARCH=$(dpkg --print-architecture)

# Add the ClickHouse repository to apt sources
echo "deb [signed-by=/usr/share/keyrings/clickhouse-keyring.gpg arch=${ARCH}] https://packages.clickhouse.com/deb stable main" | sudo tee /etc/apt/sources.list.d/clickhouse.list

# Update apt package lists
sudo apt-get update
```

#### Option 2: Install Specific Version

```bash
# Install prerequisite packages
sudo apt-get install apt-transport-https ca-certificates dirmngr

# Add the ClickHouse GPG key to authenticate packages
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754

# Add the ClickHouse repository to apt sources
echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
    
# Update apt package lists
sudo apt-get update

# Install ClickHouse server and client packages
sudo apt-get install -y clickhouse-server clickhouse-client

# Start the ClickHouse server service
sudo service clickhouse-server start

# Launch the ClickHouse command line client
clickhouse-client # or "clickhouse-client --password" if you set up a password.
```

### Install Clickhouse Server and Client

```bash
sudo apt-get install -y clickhouse-server clickhouse-client
```

---

## Start ClickHouse

```bash
sudo systemctl start clickhouse-server
```

### Check connection

```bash
clickhouse-client
```

If you set a password then you will need to 
```bash
clickhouse-client --password
```

---

## Basic Setup

## Enable Remote Connections

### Allow ClickHouse to Listen on All IPs

Edit the config file:

```bash
sudo nano /etc/clickhouse-server/config.xml
```

Find and update:

```xml
<listen_host>0.0.0.0</listen_host>
```

Set up web UI Tabix
Find http_server_default_response and uncomment it
```xml

```

---

### Apply Changes

```bash
sudo systemctl restart clickhouse-server
```

---

### Set Default Password
run
```bash
echo -n 'your_password' | sha256sum | tr -d '-'
```

Then on the /etc/clickhouse-server/config.xml file replace it:

```xml
<password_sha256_hex>'your_password'</password_sha256_hex>
```

---

### Create Roles and Users

#### Create Admin Role
```bash
sudo nano /etc/clickhouse-server/users.xml
```

Then include right before default:
```xml
        <admin>
            <password_sha256_hex>strong_password</password_sha256_hex>
            <networks>
                <ip>::/0</ip>
            </networks>
            <access_management>1</access_management>
            <named_collection_control>1</named_collection_control>
            <show_named_collections>1</show_named_collections>
            <show_named_collections_secrets>1</show_named_collections_secrets>
        </admin>
```

#### Create Regular User

```sql
CREATE USER user_name IDENTIFIED WITH plaintext_password BY 'my_secure_password';

```

#### Create Role
```sql
CREATE ROLE role_name;
```

Grant role to user
```sql
GRANT analyst TO daniel;
```

Make role default:
```sql
SET DEFAULT ROLE readonly TO user;
```
---

### Grant Permissions

```sql
-- Grant access to public schema
GRANT USAGE ON your_database.* TO your_user;

-- Grant SELECT on existing tables
GRANT SELECT ON your_database.* TO your_user;
```

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

## 🤝 Contributing

Pull requests are welcome! If you want to contribute major changes, please open an issue first to discuss what you'd like to add or modify.

---
