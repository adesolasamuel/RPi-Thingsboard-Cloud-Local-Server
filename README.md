# How to set up Raspberry Pi Thingsboard Cloud Local Server

Thingsboard is an hybrid IoT Cloud platform that can be used to monitor your devices data. One very good capability of Thingsboard is that it can be used locally on your machine (Windows, Linus, Mac or Raspberry Pi) without needing internet connection and it can also be used on the cloud like a normal cloud platform. To run a local server of Thingsboard on your Raspberry pi, you will need to install some runtime for the backend and also setting up the data base since the telemetry data will be hosted locally. 

The following steps will guide you on setting up Thingsboard cloud server on your Raspberry Pi. 

## Step 1. Install Java 11 (OpenJDK)
ThingsBoard service is running on Java 11. Follow this instructions to install OpenJDK 11:
```
sudo apt update
sudo apt install openjdk-11-jdk
```
You will need configure your operating system to use OpenJDK 11 by default. You can configure which version is the default using the following command:

```
sudo update-alternatives --config java
```
You can check the installation using the following command:
```
java -version
```
Expected command output is:
```
openjdk version "11.0.xx"
OpenJDK Runtime Environment (...)
OpenJDK 64-Bit Server VM (build ...)
```

## Step 2. ThingsBoard service installation
Download installation package.
```
wget https://github.com/thingsboard/thingsboard/releases/download/v3.4.1/thingsboard-3.4.1.deb
```

Install ThingsBoard as a service
```
sudo dpkg -i thingsboard-3.4.1.deb
```
## Step 3. Configure ThingsBoard database

The database we will be using with Thingsboard is PostgresSQL, it is the recommended database by Thingsboard. Many cloud vendors support managed PostgreSQL servers which is a cost-effective solution for most of ThingsBoard instances. Instructions listed below will help you to install PostgreSQL.

import the repository signing key:
```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

add repository contents to your system:
```
RELEASE=$(lsb_release -cs)
echo "deb http://apt.postgresql.org/pub/repos/apt/ ${RELEASE}"-pgdg main | sudo tee  /etc/apt/sources.list.d/pgdg.list
```

install and launch the postgresql service:
```
sudo apt update
sudo apt -y install postgresql
sudo service postgresql start
```

Once PostgreSQL is installed you may want to create a new user or set the password for the the main user. The instructions below will help to set the password for main postgresql user

```
sudo su - postgres
psql
\password
\q
```

Then, press “Ctrl+D” to return to main user console and connect to the database to create thingsboard DB:
```
psql -U postgres -d postgres -h 127.0.0.1 -W
CREATE DATABASE thingsboard;
\q
```

## Step 4: ThingsBoard Configuration
Edit ThingsBoard configuration file

```
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

Add the following lines to the configuration file. Don’t forget to replace “PUT_YOUR_POSTGRESQL_PASSWORD_HERE” with your real postgres user password ypu entered above:
```
# DB Configuration 
export DATABASE_TS_TYPE=sql
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=PUT_YOUR_POSTGRESQL_PASSWORD_HERE
# Specify partitioning size for timestamp key-value storage. Allowed values: DAYS, MONTHS, YEARS, INDEFINITE.
export SQL_POSTGRES_TS_KV_PARTITIONING=MONTHS
```

## Step 5. Choose ThingsBoard queue service
ThingsBoard uses queue services for API calls between micro-services and able to use next queue services, we will be using In-memory which is built in as the default in Thingsboard. In Memory queue is built-in and enabled by default. No additional configuration steps required.

## Step 6. Memory update for slow machines (1GB of RAM)
If your Raspberry pi is 1GB RAM edit ThingsBoard configuration file

```
sudo nano /etc/thingsboard/conf/thingsboard.conf
```
Add the following lines to the configuration file

```
# Update ThingsBoard memory usage and restrict it to 256MB in /etc/thingsboard/conf/thingsboard.conf
export JAVA_OPTS="$JAVA_OPTS -Xms256M -Xmx256M"
```
