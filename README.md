## initail directory structure of `microhub` microservices project
1. to get directory structure run command
```
atul@atul-Lenovo-G570:~/microhub$ tree -a -I ".git"
```
2. directory structure
```
mirohub
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