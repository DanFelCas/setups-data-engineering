# 🚀 ClickHouse Setup Guide (Ubuntu)

This guide walks you through installing ClickHouse on Ubuntu, configuring users and roles, enabling remote access, and applying basic security best practices.

---

## 📦 Installation

###  Set Up the ClickHouse APT Repository

You can install the latest stable or LTS release. The LTS version is recommended for production environments.

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

### 💾 Install ClickHouse Server and Client

```bash
sudo apt-get install -y clickhouse-server clickhouse-client
```

---

## 🚀 Start and Enable ClickHouse

```bash
sudo systemctl start clickhouse-server
sudo systemctl enable clickhouse-server
```

### Check Server Status

```bash
sudo systemctl status clickhouse-server
```

---

### Check connection

## 🔍 Test ClickHouse Client

```bash
clickhouse-client
```

If you have a password set:

```bash
clickhouse-client --password
```

---

## 🌐 Enable Remote Access

### Edit Configuration

```bash
sudo nano /etc/clickhouse-server/config.xml
```

Find and set:

```xml
<listen_host>0.0.0.0</listen_host>
```

> 💡 Optionally, enable HTTP UI tools like Tabix by ensuring `<http_server_default_response>` is not commented out.

### Apply Changes

```bash
sudo systemctl restart clickhouse-server
```

---

## 🔐 Create Admin User (XML Method - Optional)

> ⚠️ XML user creation is optional and less flexible than SQL. Prefer SQL unless needed for bootstrap.

### Back Up and Edit Config

```bash
sudo nano /etc/clickhouse-server/users.xml
```

Add before `<default>`:

```xml
<admin>
    <password_sha256_hex>your_sha256_password_here</password_sha256_hex>
    <networks>
        <ip>::/0</ip>
    </networks>
    <access_management>1</access_management>
    <named_collection_control>1</named_collection_control>
    <show_named_collections>1</show_named_collections>
    <show_named_collections_secrets>1</show_named_collections_secrets>
</admin>
```

Generate the password hash:

```bash
echo -n 'your_password' | sha256sum | awk '{print $1}'
```

Restart the server:

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

## 👤 Create Users and Roles via SQL (Recommended)

### Connect as Admin

```bash
clickhouse-client --user admin --password
```

### Create User

```sql
CREATE USER user_name IDENTIFIED WITH plaintext_password BY 'secure_password';
```

### Create Role

```sql
CREATE ROLE role_name;
```

Grant role to user:
```sql 
GRANT analyst TO daniel;
```

Make role default:
```sql
SET DEFAULT ROLE readonly TO user;
```
---

## 🎯 Grant Database Permissions

```sql
-- Grant basic read access
GRANT USAGE ON your_database.* TO daniel;
GRANT SELECT ON your_database.* TO daniel;

-- (Optional) Grant full access
GRANT ALL ON your_database.* TO daniel;
```

---

## 🔥 Configure Firewall for Remote Access (Optional)

```bash
sudo ufw allow 8123/tcp    # HTTP interface
sudo ufw allow 9000/tcp    # Native TCP interface
sudo ufw enable
```

> ⚠️ Only expose ports if needed, and restrict by IP range when possible.

---
