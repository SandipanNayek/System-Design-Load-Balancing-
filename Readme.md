## 1. Load Balancing with Nginx (Round Robin Algorithm)

Implemented a load balancing architecture using **Nginx as a reverse proxy and load balancer** to distribute incoming HTTP requests across three Node.js server instances using the **Round Robin algorithm**.

### Architecture

- Nginx acts as the entry point for incoming client requests.
- Three independent Node.js server containers run the same application.
- Nginx distributes incoming requests sequentially across the available backend servers using Round Robin.
- Docker Compose manages the multi-container environment.
- Each backend instance uses a different `SERVER_NAME` environment variable to identify which server handles the request.

### Request Flow

```text
                   Client
                     |
                     |
              localhost:8000
                     |
                     v
              +-------------+
              |    Nginx    |
              |Load Balancer|
              +-------------+
                     |
          Round Robin Algorithm
                     |
          +----------+----------+
          |          |          |
          v          v          v
     +---------+ +---------+ +---------+
     | Server1 | | Server2 | | Server3 |
     |  :7000  | |  :7000  | |  :7000  |
     +---------+ +---------+ +---------+
          |          |          |
          v          v          v
      pehla       dusra      teesra
      server      server     server
```

### Nginx Configuration (`nginx/nginx.conf`)

```nginx
events {
}

http {
    upstream backend {
        server server1:7000;
        server server2:7000;
        server server3:7000;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

**Key Concepts:**
- `upstream backend`: Defines a group of backend servers.
- `server server1:7000`: Uses Docker Compose service names for internal communication.
- Round Robin: Nginx distributes requests sequentially among the backend servers by default.
- `proxy_pass`: Forwards client requests to the upstream backend group.
- `listen 80`: Nginx listens for incoming HTTP requests on port 80.

### Docker Compose Configuration (`docker-compose.yml`)

```yaml
services:
  nginx:
    image: nginx
    ports:
      - "8000:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf

  server1:
    build: ./server
    env_file:
      - ./server/.env
    environment:
      - SERVER_NAME=pehla server
    ports:
      - "5001:7000"

  server2:
    build: ./server
    env_file:
      - ./server/.env
    environment:
      - SERVER_NAME=dusra server
    ports:
      - "5002:7000"

  server3:
    build: ./server
    env_file:
      - ./server/.env
    environment:
      - SERVER_NAME=teesra server
    ports:
      - "5003:7000"
```

**Key Concepts:**
- Docker Compose creates a shared network for the containers.
- Nginx communicates with backend services using their service names.
- Each server runs internally on port `7000`.
- Host ports `5001`, `5002`, and `5003` allow direct access to the individual backend instances.
- Nginx exposes port `8000` to the host machine.

### Node.js Dockerfile (`server/Dockerfile`)

```dockerfile
FROM node

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["node", "index.js"]
```

**Key Concepts:**
- `FROM node`: Uses the Node.js base image.
- `WORKDIR /app`: Sets the working directory inside the container.
- `COPY package*.json ./`: Copies package manifests.
- `RUN npm install`: Installs application dependencies.
- `COPY . .`: Copies the application source code.
- `CMD`: Starts the Node.js server.

### Running the Application

Start all containers:

```bash
docker compose up --build
```

Access the load balancer:

```text
http://localhost:8000
```

Direct access to individual backend instances:

```text
http://localhost:5001
http://localhost:5002
http://localhost:5003
```

### Round Robin Demonstration

Repeated requests to `http://localhost:8000` are distributed across the backend instances.

Example:

| Request | Backend Server | Response |
|---|---|---|
| Request 1 | Server 1 | pehla server |
| Request 2 | Server 2 | dusra server |
| Request 3 | Server 3 | teesra server |
| Request 4 | Server 1 | pehla server |
| Request 5 | Server 2 | dusra server |
| Request 6 | Server 3 | teesra server |

*Note: This sequence illustrates the expected Round Robin distribution under normal conditions. Actual requests may vary depending on Nginx worker behavior, connection handling, and backend availability.*

### Learning Outcomes

- Implemented Nginx as a reverse proxy and load balancer.
- Applied the Round Robin load balancing algorithm.
- Distributed incoming HTTP traffic across multiple Node.js instances.
- Used Docker Compose for multi-container orchestration.
- Understood internal Docker networking and service discovery.
- Explored horizontal scaling by running multiple backend instances.
