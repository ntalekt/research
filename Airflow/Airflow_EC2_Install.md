# Apache Airflow on EC2 Ubuntu 18.04LTS
## EC2 instance
Leveraged a general purpose t2.micro Ubuntu Server 18.04 LTS (HVM), SSD Volume Type - ami-06397100adf427136
## Update the instance
```b
sudo apt-get update
sudo apt-get upgrade -y
```
## Install Python and pip
```
sudo apt-get install python-setuptools -y
sudo apt-get install python-pip -y
sudo pip install --upgrade pip
```
## Install PostgreSQL & Configure
1. Install postgreSQL
```
sudo apt-get install postgresql postgresql-contrib -y
sudo apt-get install python-psycopg2 -y
```
2. Access psql
```
sudo -u postgres psql
```
3. Create new user, database and grant access
```sql
CREATE ROLE ubuntu;
ALTER ROLE "ubuntu" WITH LOGIN;
CREATE DATABASE airflow;
GRANT ALL PRIVILEGES on database airflow to ubuntu;
ALTER ROLE ubuntu SUPERUSER;
ALTER ROLE ubuntu CREATEDB;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public to ubuntu;
```
4. Connect and get info
```
postgres-# \c airflow
airflow=# \conninfo
airflow=# \q
```
5. Update settings in Airflow configuration
```
sudo vi /etc/postgresql/*/main/pg_hba.conf
```
```
# IPv4 local connections:
host    all             all             0.0.0.0/0               md5
```
6. Restart the PostgreSQL service
```
sudo service postgresql restart
```

## Install Airflow
1. Set AIRFLOW_HOME environment variable to ~/airflow.
```
export AIRFLOW_HOME=~/airflow
```
2. Install Airflow dependencies
```
sudo apt-get install libmysqlclient-dev libssl-dev libkrb5-dev libsasl2-dev -y
```
3. Install Airflow using pip
```
sudo pip install apache-airflow
```
4. Initialize Airflow's database
```
airflow initdb
```

### Configure Airflow
1. Edit the Airflow config.
```
vi ~/airflow/airflow.cfg
```
2. Use CeleryExecutor instead of SequentialExecutor
```
executor = CeleryExecutor
```
3. Setup the DB connection
```
sql_alchemy_conn = postgresql+psycopg2://ubuntu@localhost:5432/airflow
```
4. Change broker_url & result_backend
```
broker_url = amqp://guest:guest@localhost:5672//
result_backend = amqp://guest:guest@localhost:5672//
```

### Install Rabbitmq
1. Install rabbitmq
```
sudo apt install rabbitmq-server
```
2. Edit configuration
```
sudo vi /etc/rabbitmq/rabbitmq-env.conf
```
```
NODE_IP_ADDRESS=0.0.0.0
```
3. Start Rabbitmq
```
sudo service rabbitmq-server start
```

### Install Celery
1. Install Celery
```
sudo pip install celery
```

### Reload Airflow Configuration
1. Load the new configuration
```
airflow initdb
```

## Starting Airflow
1. Create dags folder
```
mkdir -p ~/airflow/dags
```
2. Start airflow services
```
airflow webserver -D
airflow scheduler -D
airflow worker -D
```
