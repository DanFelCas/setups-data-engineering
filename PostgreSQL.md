# Setup Steps PostgreSQL - Ubuntu

## Install 
### For latest version

apt install postgresql


### For specific version
- Automated repository configuration:
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

- To manually configure the Apt repository, follow these steps:
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
. /etc/os-release
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"
sudo apt update
sudo apt -y install postgresql

### Status check
sudo systemctl status postgresql

## Setup

- Set postgres user password
sudo su - postgres
psql 

ALTER user postgres with password 'your_password';

- create admin role
CREATE ROLE admin with login superuser createdb createrole inherit replication password 'admin_password';

- create user
CREATE USER username WITH PASSWORD 'user_password';


- user grants
GRANT CONNECT ON DATABASE your_database TO username;
GRANT USAGE ON SCHEMA public TO username;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO username;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO username; 



- let postgres listen all ips

sudo nano /etc/postgresql/*/main/postgresql.conf

look for "listen_addresses" replace with:

listen_addressess = '*'

this will enable all addresses

- Give permissions for users to log in 

this will allow all users to connect remotely
host    all     all     0.0.0.0/0   scram-sha-256


to apply changes

sudo systemctl restart postgresql