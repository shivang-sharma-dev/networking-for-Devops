# 🟡 Intermediate Project: Multi-Tier App with Nginx Load Balancer

## Project Overview

**Goal**: Deploy a 3-tier application with a proper network topology using Docker Compose — reverse proxy, load-balanced app servers, and a database on an isolated network.

**Skills practiced**: Docker networking, Nginx LB, environment isolation, health checks

**Time estimate**: 3–4 hours

---

## Architecture

```
Internet
    │
    ▼
[Nginx Reverse Proxy :80]
    │
    ├──► [App Server 1 :5000]
    ├──► [App Server 2 :5000]  ←── app-network
    └──► [App Server 3 :5000]
                │
                ▼
         [PostgreSQL DB]  ←── db-network (isolated)
```

---

## Setup Files

### `docker-compose.yml`

```yaml
version: "3.8"

networks:
  frontend:
  backend:
  db-net:

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app1
      - app2
      - app3
    networks:
      - frontend
      - backend

  app1:
    build: ./app
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - SERVER_ID=app1
    networks:
      - backend
      - db-net
    depends_on:
      - db

  app2:
    build: ./app
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - SERVER_ID=app2
    networks:
      - backend
      - db-net
    depends_on:
      - db

  app3:
    build: ./app
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - SERVER_ID=app3
    networks:
      - backend
      - db-net
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - db-net    # Only accessible from db-net

volumes:
  pgdata:
```

### `nginx/nginx.conf`

```nginx
events { worker_connections 1024; }

http {
    upstream app_servers {
        least_conn;
        server app1:5000;
        server app2:5000;
        server app3:5000;
    }

    server {
        listen 80;

        location /health {
            return 200 "proxy ok\n";
        }

        location / {
            proxy_pass http://app_servers;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
        }
    }
}
```

### `app/app.py` (Simple Flask app)

```python
from flask import Flask, jsonify
import os
import psycopg2

app = Flask(__name__)

@app.route("/")
def index():
    return jsonify({
        "server": os.getenv("SERVER_ID", "unknown"),
        "status": "ok"
    })

@app.route("/health")
def health():
    return jsonify({"status": "healthy"})

@app.route("/db")
def db_check():
    conn = psycopg2.connect(
        host=os.getenv("DB_HOST"),
        port=os.getenv("DB_PORT"),
        database="myapp",
        user="postgres",
        password="secret"
    )
    conn.close()
    return jsonify({"db": "connected"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

## Tasks

1. **Deploy** the stack: `docker compose up -d`
2. **Verify LB distribution**: `for i in {1..9}; do curl -s localhost/ | jq .server; done`
3. **Verify DB isolation**: Try `docker compose exec nginx curl http://db:5432` (should fail — nginx is not on db-net!)
4. **Kill one app server**: `docker compose stop app2` — verify LB still works
5. **Scale app servers**: Add `app4` and restart nginx
6. **Add health checks** to docker-compose.yml and test that unhealthy containers are removed from LB

---

## Deliverable

Document your setup with a `README.md` in this folder:
- Architecture diagram
- Network isolation explanation
- Tested scenarios and results
