## Prerequisites

Before you begin, make sure you have the following installed:

- **Docker**: Ensure Docker Desktop is installed and running on your machine.
- **Docker Compose**: Docker Desktop typically includes Docker Compose. Verify it by running:
  ```bash
  docker-compose --version
Cloning the Repository
Start by cloning the repository to your local machine. Open your terminal and run:

```bash
git clone https://github.com/your-username/petclinic-docker.git
cd petclinic-docker
```
This command downloads the project files to your local directory.

Creating Redis and Sentinel Configuration Files
To set up Redis with Sentinel for high availability, we need to create configuration files. Follow these steps:

Create a directory for Redis configurations:

```bash
mkdir -p redis/conf
```
Create Redis configuration files: Use a text editor to create three files named redis-0, redis-1, and redis-2 inside the redis/conf directory:

```bash
touch redis/conf/redis-0.conf redis/conf/redis-1.conf redis/conf/redis-2.conf
```
Contents for redis-0.conf:

```bash
protected-mode no
port 6379
masterauth a-very-complex-password-here
requirepass a-very-complex-password-here
Contents for redis-1.conf:
```

Contents for redis-1.conf:

```bash
protected-mode no
port 6379
slaveof redis-0 6379
masterauth a-very-complex-password-here
requirepass a-very-complex-password-here
Contents for redis-2.conf:
```

```bash
protected-mode no
port 6379
slaveof redis-0 6379
masterauth a-very-complex-password-here
requirepass a-very-complex-password-here
```

Create Sentinel configuration files: Similarly, create three files named sentinel-01.conf, sentinel-02.conf, and sentinel-03.conf inside the redis/conf directory:


```bash
touch redis/conf/sentinel-01.conf redis/conf/sentinel-02.conf redis/conf/sentinel-03.conf
```

Contents for each sentinel file:

```bash
port 5000
sentinel monitor mymaster redis-0 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster a-very-complex-password-here
```

Understanding Docker Compose
. In our project, we have multiple services (e.g., the Pet Clinic application, MySQL database, and Redis cluster) that work together seamlessly.

Explaining the docker-compose.yml File
Here’s the docker-compose.yml file used in this project:

```bash

services:
  petclinic:
    build:
      context: .
      dockerfile: Dockerfile.multi
    environment:
      SERVER_PORT: 8080
      MYSQL_URL: jdbc:mysql://mysqlserver/petclinic
    volumes:
      - /app
    ports:
      - "8100:8080"

  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: ""
      MYSQL_ALLOW_EMPTY_PASSWORD: true
      MYSQL_USER: petclinic
      MYSQL_PASSWORD: petclinic
      MYSQL_DATABASE: petclinic
    volumes:
      - mysql_config:/etc/mysql/conf.d

  redis-0:
    image: redis:4.0.2
    command: ["redis-server", "/etc/redis/redis.conf"]
    volumes:
      - ./redis/conf/redis-0:/etc/redis/redis.conf

  redis-1:
    image: redis:4.0.2
    command: ["redis-server", "/etc/redis/redis.conf"]
    volumes:
      - ./redis/conf/redis-1:/etc/redis/redis.conf

  redis-2:
    image: redis:4.0.2
    command: ["redis-server", "/etc/redis/redis.conf"]
    volumes:
      - ./redis/conf/redis-2:/etc/redis/redis.conf

  sentinel-01:
    image: redis:4.0.2
    command: ["redis-sentinel", "/etc/redis/sentinel.conf"]
    volumes:
      - ./redis/conf/sentinel-01:/etc/redis/sentinel.conf

  sentinel-02:
    image: redis:4.0.2
    command: ["redis-sentinel", "/etc/redis/sentinel.conf"]
    volumes:
      - ./redis/conf/sentinel-02:/etc/redis/sentinel.conf

  sentinel-03:
    image: redis:4.0.2
    command: ["redis-sentinel", "/etc/redis/sentinel.conf"]
    volumes:
      - ./redis/conf/sentinel-03:/etc/redis/sentinel.conf

volumes:
  mysql_config:
```

Explanation of Each Section
version: Specifies the version of Docker Compose being used (3.8 in this case).

services: Lists all the services (containers) that will be created.

petclinic:

build: Specifies the context and Dockerfile for building the image.
environment: Sets environment variables for the application (e.g., server port, MySQL connection).
volumes: Defines volumes for persistent storage.
ports: Maps port 8080 of the container to port 8100 on the host machine.
mysql: Configures the MySQL database.

image: Specifies the MySQL image version.
environment: Sets database credentials and configuration.
volumes: Mounts MySQL configuration.
redis-0, redis-1, redis-2: Configures three Redis instances.

image: Uses Redis version 4.0.2.
command: Starts Redis with the specified configuration file.
volumes: Maps Redis configuration files.
sentinel-01, sentinel-02, sentinel-03: Configures Redis Sentinel instances for monitoring.

command: Starts Redis Sentinel with the specified configuration file.
volumes: Maps Sentinel configuration files.
volumes: Defines named volumes for persistent storage, specifically for MySQL configuration.

Building and Running the Application
To start the application, run the following command in the terminal:

```bash
docker-compose up -d
```

What Happens When You Run This Command
- Builds Images: If not already built, Docker creates images for all defined services.
- Starts Containers: Each service runs in its own container, allowing them to work together, but in this case we'll have to run our containers manually because at the end of the docker-compose we'll only have the images.
  
Output of the Command
You will see logs indicating that each container is starting. This includes images like:

Redis Images: For the Redis instances.
Pet Clinic Image: The application image.
MySQL Image: The MySQL database image.
Port Mapping
The Pet Clinic application will be accessible on port 8100 of your host machine, which maps to port 8080 in the container.

Accessing the Application
Once the build is finished, you run the command 

```bash
docker ps
```
And you'll see a list of docker images such as; petclinic-docker-petclinic, mysql, redis

To be able to see the application in the Ui, we'll have to create a running instance of the image called a container and expose it in the container port 8080 and in the host port, any one of your choice

```bash
docker run -dp 8100:8080 --name petclinic-app petclinic-docker-petclinic
```

Pet Clinic UI: 
```bash
http://localhost:8100
```
This URL will display the Pet Clinic application interface.

<img width="1434" alt="Screen Shot 2024-10-01 at 08 56 33" src="https://github.com/user-attachments/assets/12bed183-4c3e-46cf-9bb5-5c5cd139666b">


Stopping the Application
To stop the application and remove the containers, run:

```bash
docker-compose down
```
This command stops all services and cleans up resources.

Troubleshooting
If you encounter any issues, consider the following:

Permission Denied Errors: Ensure that the mvnw file is executable. If you face issues, run:

```bash
chmod +x mvnw
```
Docker Daemon Issues: Make sure Docker Desktop is running and check for error messages in the Docker logs.

Network Issues: Ensure no other services are using the same ports as defined in the docker-compose.yml.

