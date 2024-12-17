# Rahmatullah DevOps Task
This repository contains the scripts, configurations, and documentation for setting up and managing various aspects of a DevOps environment, including server management, DNS configuration, CI/CD pipeline creation, load balancing, resource monitoring, database management, and application deployment.

## Task 1

### Server Management

**Objective**: Set up an Ubuntu server, create a user named `devops` with sudo privileges, and install and configure Nginx to serve a static HTML page on port 80.

![alt text](https://github.com/sahunishant191/Rahmatullah-Nowruz/blob/main/Task%201/Ubuntu-server-24.png)

**Deliverables**:
- Commands used for user creation.
- Nginx configuration file.

**Commands for User Creation and providing sudo access**:
```bash
sudo adduser devops
sudo usermod -aG sudo devops
```

**Nginx Configuration File**: 
```bash
nano /etc/nginx/sites-available/default
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}

systemctl restart nginx
```
## Task 2

### DNS Management
**Objective**: Set up a subdomain devops.example.com to point to your server’s IP address and configure a CNAME record for www.devops.example.com to point to devops.example.com.

**Deliverables**: I have shared the steps in the document in task 2 folder how to do the setup , since i do not have any domian as of now so i haved shared the steps to do it.

## Task 3

### CI/CD Pipeline Creation
**Objective**: Create a CI/CD pipeline using any tool (Jenkins, GitHub Actions, or GitLab CI) to pull code from a repository, run a simple test script, and deploy the code to the server with an .env file.

**Deliverables**: This Task is completed and is mentioned in Task 3 Folder with screenshot and documents.
```bash
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install Dependencies
      run: npm install

    - name: Run Tests
      run: npm test

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Deploy to Server
      env:
        SERVER_IP: ${{ secrets.SERVER_IP }}
        USERNAME: ${{ secrets.USERNAME }}
        PRIVATE_KEY: ${{ secrets.PRIVATE_KEY }}
        ENV_FILE: ${{ secrets.ENV_FILE }}
      run: |
        echo "$PRIVATE_KEY" > private_key
        chmod 600 private_key
        scp -i private_key .env $USERNAME@$SERVER_IP:/home/ubuntu/app/.env
        ssh -i private_key $USERNAME@$SERVER_IP << 'EOF'
          cd /home/ubuntu/app
          npm install
          pm2 restart all
        EOF
```
## Task 4

### GitHub Actions Setup
**Objective**: Create a GitHub Actions workflow to lint a NestJS app, run build, and deploy the application to the server.

**Deliverables**: GitHub Actions YAML file.

```bash
name: Simple CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Run Simple Test
      run: echo "Running simple test" && exit 0

  deploy:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Deploy to Server
      env:
        SERVER_IP: ${{ secrets.SERVER_IP }}
        USERNAME: ${{ secrets.USERNAME }}
        PRIVATE_KEY: ${{ secrets.PRIVATE_KEY }}
      run: |
        echo "$PRIVATE_KEY" > private_key
        chmod 600 private_key
        scp -i private_key -r * $USERNAME@$SERVER_IP:/home/ubuntu/app2
        ssh -i private_key $USERNAME@$SERVER_IP "echo 'Deployment complete'"
```
## Task 5

### Nginix Load Balancing Setup
**Objective**: Set up a load balancer using Nginx to distribute traffic between two backend servers using a round-robin algorithm.

**Deliverables**: Nginx configuration file.
Create or edit the Nginx configuration file (typically located at /etc/nginx/nginx.conf or in a separate file within /etc/nginx/conf.d/)
```bash
http {
    upstream backend_servers {
        server backend1.example.com;
        server backend2.example.com;
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

**Explanation of the setup**
1. **Define the Upstream Servers**:
•	The upstream directive creates a group of backend servers that Nginx will distribute requests to.In this example, the backend_servers group includes two servers: backend1.example.com and backend2.example.com.
2. **Configure the Load Balancer**:
•	The server block listens on port 80 for incoming HTTP requests.
The location / block handles all requests and proxies them to the backend_servers group.
3. **Proxy Pass Configuration**:
•	proxy_pass http://backend_servers; forwards requests to the backend server group.
Various proxy_set_header directives set appropriate headers for the proxied requests, such as the original host, client IP address, and protocol.

**Load Balancing Algo**:
By default, Nginx uses the round-robin algorithm to distribute traffic among the servers listed in the upstream block. This means that each incoming request will be forwarded to the next server in the list in a circular manner, ensuring an even distribution of load

## Task 6

### Resource Management
**Objective**: Monitor CPU, memory, and disk usage on the server by installing and configuring a monitoring tool (e.g., Prometheus + Grafana, htop, or any equivalent).

**Deliverables**: Screenshots of the monitoring dashboard

**Prometheus Dashboard**

![alt text](https://github.com/sahunishant191/Rahmatullah-Nowruz/blob/main/Task%206/Prometheus.png)

**Grafana Dashboard**

![alt text](https://github.com/sahunishant191/Rahmatullah-Nowruz/blob/main/Task%206/grafana%20monitoring.png)


## Task 7
### Database Management
**Objective**: Install PostgreSQL and MariaDB on the server, create a database named test_db and a user named db_user with full privileges for both databases, and write a SQL query to create a table named employees.

**Deliverables**:

SQL Query for Table Creation on _**PostgreSQL**_
```bash
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL,
    position VARCHAR,
    salary INTEGER
);
```

SQL Queryt for Table Creation on _**MariaDB**_
```bash
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    position VARCHAR(255),
    salary INT
);
```


## Task 8

### Backup and Restoration
**Objective**: Create a script to back up the test_db database (both PostgreSQL and MariaDB) to a file and demonstrate restoring the database from the backup.

**Deliverables**:

_**PostgreSQL Backup Script**_:
```bash
#!/bin/bash

DB_NAME="test_db"
DB_USER="postgres"
BACKUP_DIR="/path/to/backup"
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_$(date +%Y%m%d%H%M%S).sql"

mkdir -p $BACKUP_DIR
pg_dump -U $DB_USER -F c -f $BACKUP_FILE $DB_NAME

echo "PostgreSQL database backup complete: $BACKUP_FILE"
```

_**MariaDB Backup Script**_:
```bash
#!/bin/bash

DB_NAME="test_db"
DB_USER="root"
BACKUP_DIR="/path/to/backup"
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_$(date +%Y%m%d%H%M%S).sql"

mkdir -p $BACKUP_DIR
mysqldump -u $DB_USER -p --databases $DB_NAME > $BACKUP_FILE

echo "MariaDB database backup complete: $BACKUP_FILE"
```
**Restoration Commands**:

_**PostgreSQL**_:
```bash
pg_restore -U postgres -d test_db /path/to/backup/test_db_YYYYMMDDHHMMSS.sql
```
_**MariaDB**_:
```bash
 mysql -u root -p test_db < /path/to/backup/test_db_YYYYMMDDHHMMSS.sql
```
## Task 9

### Node.jsNestJS Application Deployment
**Objective**: Set up a Node.jsenvironment on the server, install and configure a NestJS application, deploy it using a CI/CD pipeline, connect it to the PostgreSQL database, ensure secure management of environment variables, lint the application before deployment, and containerize the application using Docker and Docker Compose.

**Deliverables**:

_**Steps for setting up the Node.js environment**_
**Step 1: Set Up Node.js Environment on the Server**
```bash
#Update and Upgrade Packages:
sudo apt update
sudo apt upgrade -y

#Install Node.js and npm:
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs

#Verify Installation:
node -v
npm -v
```

**Step 2: Install and Configure a NestJS Application**
```lua
#Install NestJS CLI:
npm install -g @nestjs/cli

#Create a New NestJS Application:
nest new my-nest-app
cd my-nest-app

#Install PostgreSQL Client Library:
npm install @nestjs/typeorm typeorm pg

#Create the .env File in the root directory:
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=db_user
DB_PASSWORD=password
DB_NAME=test_db

#Install dotenv:
npm install dotenv

#Configure TypeORM in app.module.ts:
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import * as dotenv from 'dotenv';

dotenv.config(); // Load environment variables

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT, 10),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_NAME,
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**CI/CD pipeline YAML file or Deploying Nestjs Application**

Create a GitHub Actions Workflow YAML: Create a file named _.github/workflows/ci-cd-pipeline.yml_:
```lua
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'

    - name: Install Dependencies
      run: npm install

    - name: Run Linter
      run: npm run lint

  build:
    runs-on: ubuntu-latest
    needs: lint

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'

    - name: Install Dependencies
      run: npm install

    - name: Run Build
      run: npm run build

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Deploy to Server
      env:
        SERVER_IP: ${{ secrets.SERVER_IP }}
        USERNAME: ${{ secrets.USERNAME }}
        PRIVATE_KEY: ${{ secrets.PRIVATE_KEY }}
        DEPLOY_PATH: ${{ secrets.DEPLOY_PATH }}
      run: |
        echo "$PRIVATE_KEY" > private_key
        chmod 600 private_key
        scp -i private_key -r * $USERNAME@$SERVER_IP:$DEPLOY_PATH
        ssh -i private_key $USERNAME@$SERVER_IP "cd $DEPLOY_PATH && npm install --production && npm run start:prod"
```

**Sample .env file or secrets management documentation**
```bash
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=db_user
DB_PASSWORD
```
**Dockerfile** 
```lua
FROM node:16-alpine
WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000
CMD ["npm", "run", "start:prod"]
```
**docker-compose.yml**
```lua
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: db_user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: test_db
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```



