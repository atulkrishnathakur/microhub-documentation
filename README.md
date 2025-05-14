## Initial directory structure of `microhub` microservices project. It is the full directory structure
1. To get directory structure run command. Here we have ignored '.git' directory. command:  `tree -a -I ".git"`
```
atul@atul-Lenovo-G570:~/microhub$ tree -a -I ".git"
```
2. Directory structure
```
microhub
├── microhub-documentation
│   ├── .dockerignore
│   ├── .gitignore
│   └── README.md
├── microhub-gateway
│   ├── app
│   │   └── main.py
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── .env
│   ├── .env.example
│   ├── .gitignore
│   ├── README.md
│   └── requirements.txt
├── microhub-nginx
│   ├── docker-compose.yml
│   ├── .dockerignore
│   ├── .gitignore
│   ├── nginx.conf
│   └── README.md
├── microhub-order-management
│   └── app
│       └── main.py
├── microhub-pgadmin
│   ├── docker-compose.yml
│   ├── .dockerignore
│   ├── .env
│   ├── .env.example
│   ├── .gitignore
│   └── README.md
├── microhub-postgresql
│   ├── docker-compose.yml
│   ├── .dockerignore
│   ├── .env
│   ├── .env.example
│   ├── .gitignore
│   └── README.md
└── microhub-product-management
    └── app
        └── main.py

```

## The `microhub-gateway` gateway service
1. Create the `microhub\microhub-gateway` directory
2. In this directory create separate git repository to push and pull in github. If you create separate repository then you can easly manage repository for every service.
3. Create the Dockerfile like `microhub\microhub-gateway\Dockerfile`
```
# Use the official Python image
FROM python:3.12.8

# Set the working directory
WORKDIR /microhub-gateway

# This command put requirements.txt in container in specific directory
COPY ./requirements.txt /microhub-gateway/requirements.txt

# This command will install dependancies in docker container
RUN pip install --no-cache-dir --upgrade -r /microhub-gateway/requirements.txt

# copy source code files and directory in docker container
COPY ./app /microhub-gateway/app

# this is default command. It will run after container start
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

```

4. Create the `microhub\microhub-gateway\docker-compose.yml` file. In this file the network has been defined because gateway is the entry point of all services. All other services will be use this network because same network is important for microservices communication.

```
# version: '3.9' # Defines the version of Docker Compose being used. No need to write in newer version in docker compose file
services:
  microhub-gateway: # Service for your FastAPI application
    build:
      context: . # Directory containing the Dockerfile
      dockerfile: Dockerfile # Path to the Dockerfile for building the image
    image: microhub-gateway:1.0 # Name and tag for the Docker image
    container_name: microhubgatewaycontainer # Custom name for the container
    ports:
      - "8000:8000" # Maps port 8000 on the host to port 8000 in the container. Here port map as <hostport>:<containerport>
    networks:
      - microhubnetwork # Connects to your custom network
    restart: always # Automatically restarts the container if it stops or after a host machine reboot

networks:
  microhubnetwork:  # Ensure this name matches all other references
    driver: bridge
    name: microhub_network  # Explicitly name it
    
```
5. You can check in browser
```
http://localhost:8000/docs
```

## The `microhub-nginx` nginx service
1. Create the `microhub\microhub-nginx` directory
2. In this directory create separate git repository to push and pull in github. If you create separate repository then you can easly manage repository for every service.
3. Create the `microhub\microhub-nginx\nginx.conf` file
```
events {}

http {
    upstream fastapi_server {
        server microhub-gateway:8000; # It is the service name of microhub-gateway. It is defined in microhub-gateway/docker-compose.yml file
    }

    server {
        listen 80;

        location / {
            proxy_pass http://fastapi_server;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

4. Create the `microhub\microhub-nginx\docker-compose.yml` file. In this file the network used that is defined in `microhub-gateway` service.

```
# version: '3.9' # Defines the version of Docker Compose being used. No need to write in newer version in docker compose file

services:
  microhub-nginx: # Service for the Nginx web server
    image: nginx:stable # Uses the stable version of the official Nginx image
    container_name: microhubnginxcontainer # Custom name for the container
    ports:
      - "80:80" # Maps port 80 on the host to port 80 in the container. Here port map as <hostport>:<containerport>
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro # Mounts the custom Nginx configuration file in read-only mode. It is bind mount. It is not persistent valume
    networks:
      - microhub_network # Connects to your custom network
    restart: always # Automatically restarts the container if it stops or after a host machine reboot

networks:
  microhub_network: # this network defined in gateway service
    external: true  # Ensures it uses the pre-created network.
```
5. After nginx cofiguration you can check in browser
```
http://localhost/docs

```

## The `microhub-pgadmin` service for pgadmin4
1. Create the `microhub\microhub-pgadmin` directory
2. In this directory create separate git repository to push and pull in github. If you create separate repository then you can easly manage repository for every service.
3. Create the `microhub\microhub-pgadmin\docker-compose.yml` file. In this file the network used that is defined in `microhub-gateway` service.
```
services:
  microhub-pgadmin: # Service for pgAdmin4
    image: dpage/pgadmin4:9.1.0 # Uses pgAdmin4 version 9.1.0
    container_name: microhubpgadmin4container # Custom name for the pgAdmin4 container
    ports:
      - "5050:80" # Maps port 5050 on the host to port 80 in the container
    env_file: 
      - .env # Load all environment variables from the .env file
    environment:
      - PGADMIN_DEFAULT_EMAIL=$PGADMIN_DEFAULT_EMAIL # Admin email for logging into pgAdmin4
      - PGADMIN_DEFAULT_PASSWORD=$PGADMIN_DEFAULT_PASSWORD # Admin password for logging into pgAdmin4
    volumes:
      - pgadmin4data:/var/lib/pgadmin # Persistent storage for database data
    networks:
      - microhub_network # Connects to your custom network
    restart: always # Automatically restarts the container if it stops or after a host machine reboot

volumes:
  pgadmin4data: # Named volume for PostgreSQL data
    driver: local # used to create valume in host machine
    name: microhub_pgadmin4data # Explicitly set the volume name

networks:
  microhub_network: # this network defined in gateway service
    external: true  # Ensures it uses the pre-created network.

```

4. After pgadmin4 cofiguration you can check in browser
```
http://localhost:5050

```


## The `microhub-postgresql` service for postgresql
1. Create the `microhub\microhub-postgresql` directory
2. In this directory create separate git repository to push and pull in github. If you create separate repository then you can easly manage repository for every service.
3. Create the `microhub\microhub-postgresql\docker-compose.yml` file. In this file the network used that is defined in `microhub-gateway` service. Create a new user and password for postgresql databases for securities.
```
# version: '3.9' # Defines the version of Docker Compose being used. No need to write in newer version in docker compose file

services:
  microhubpostgresql: # Service for the PostgreSQL database
    image: postgres:17 # Uses the official PostgreSQL image for version 17
    container_name: microhubpostgresqlcontainer # Custom name for the container
    env_file: 
      - .env # Load all environment variables from the .env file
    ports:
      - "5432:5432"  # Define PostgreSQL default port
    environment:
      - POSTGRES_USER=$POSTGRES_USER # explicitly define postgresql super user
      - POSTGRES_PASSWORD=$POSTGRES_PASSWORD # Explicitly defines postgresql super user password
    volumes:
      - postgresqlvolume:/var/lib/postgresql/data # Persistent storage for database data
    networks:
      - microhub_network # Connects to your custom network
    restart: always # Automatically restarts the container if it stops or after a host machine reboot

volumes:  
  postgresqlvolume: # Named volume for PostgreSQL data
    driver: local # used to create valume in host machine
    name: microhub_postgresqlvolume # Explicitly set the volume name

networks:
  microhub_network: # this network defined in gateway service
    external: true  # Ensures it uses the pre-created network.

```

4. After configuration you can check postgresql databases using `psql`


## What is service discovery
1. Service discovery is a way for microservices to automatically find and connect to each other without needing to know exact addresses.
2. Since Docker containers can start, stop, and move around, their IP addresses might change. Instead of manually updating these addresses, service discovery helps containers find each other dynamically.
3. Service discovery is the process by which services in a network automatically locate and communicate with each other without requiring hardcoded addresses or configurations. It ensures that services can dynamically register, find, and connect with other services as needed. 

## How to create sorvice discovery using docker compose
1. When all services are in the same Docker network, Docker automatically provides DNS resolution, and the DNS name for each service matches its service name as defined in docker-compose.yml. This is why other services can communicate using http://service-name:port instead of needing to know container IDs or IPs.
2. In Docker Compose, services are automatically registered in service discovery when they share the same network.
3. Each service name defined in docker-compose.yml becomes a DNS hostname.
4. Docker assigns each container an internal IP address within the network.
5. Services communicate using service names instead of IPs.
