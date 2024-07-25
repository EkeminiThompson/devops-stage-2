# Full-Stack FastAPI and React Template

Welcome to the Full-Stack FastAPI and React template repository. This repository serves as a demo application for interns, showcasing how to set up and run a full-stack application with a FastAPI backend and a ReactJS frontend using ChakraUI.

## Project Structure

The repository is organized into two main directories:

- **frontend**: Contains the ReactJS application.
- **backend**: Contains the FastAPI application and PostgreSQL database integration.

Each directory has its own README file with detailed instructions specific to that part of the application.

## Getting Started

To get started with this template, please follow the instructions in the respective directories:

- [Frontend README](./frontend/README.md)
- [Backend README](./backend/README.md)

### Step-by-Step Guide: Dockerized Full Stack Web Application Deployment

#### Prerequisites:
- AWS EC2 instance (Amazon Linux 2 or Ubuntu)
- Basic knowledge of Docker, Docker Compose, and Nginx
- Domain/subdomain (can be obtained from Afraid DNS for free)
- GitHub account

### Step 1: Fork the Repository
1. Fork the repository: [https://github.com/hngprojects/devops-stage-2](https://github.com/hngprojects/devops-stage-2)
2. Clone your fork to your local machine:
    ```sh
    git clone https://github.com/your-username/devops-stage-2.git
    cd devops-stage-2
    ```

### Step 2: Dockerization

#### Backend (FastAPI)
1. Create a `Dockerfile` in the `backend` directory:
    ```Dockerfile
    # backend/Dockerfile
    FROM python:3.10-slim

    WORKDIR /app

    COPY pyproject.toml poetry.lock /app/
    RUN pip install poetry && poetry config virtualenvs.create false && poetry install

    COPY . /app

    CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
    ```

2. Ensure your backend uses environment variables to configure the database connection. Update `main.py` or your config file accordingly.

#### Frontend (React with Vite)
1. Create a `Dockerfile` in the `frontend` directory:
    ```Dockerfile
    # frontend/Dockerfile
    FROM node:18

    WORKDIR /app

    COPY package.json package-lock.json ./
    RUN npm install

    COPY . .

    RUN npm run build

    CMD ["npx", "serve", "-s", "dist", "-l", "3000"]
    ```

### Step 3: Configure Nginx Proxy
1. Create an `nginx.conf` file in the root of your project:
    ```nginx
    # nginx.conf
    server {
        listen 80;
        server_name your-domain.com;

        location / {
            proxy_pass http://frontend:3000;
        }

        location /api {
            proxy_pass http://backend:8000/api;
        }

        location /docs {
            proxy_pass http://backend:8000/docs;
        }

        location /redoc {
            proxy_pass http://backend:8000/redoc;
        }
    }
    ```

2. Create a `docker-compose.yml` file in the root of your project:
    ```yaml
    version: '3.8'

    services:
      frontend:
        build: ./frontend
        ports:
          - "3000:3000"
        networks:
          - webnet

      backend:
        build: ./backend
        ports:
          - "8000:8000"
        networks:
          - webnet
        environment:
          - DATABASE_URL=postgresql://postgres:GRACEGRACE2020@e@db:5432/postgres

      db:
        image: postgres:13
        volumes:
          - postgres_data:/var/lib/postgresql/data
        environment:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: GRACEGRACE2020@e
          POSTGRES_DB: postgres

      adminer:
        image: adminer
        ports:
          - 8080:8080
        networks:
          - webnet

      nginx:
        image: nginx:alpine
        volumes:
          - ./nginx.conf:/etc/nginx/nginx.conf
        ports:
          - "80:80"
        networks:
          - webnet

    networks:
      webnet:

    volumes:
      postgres_data:
    ```

### Step 4: Database Configuration
1. Ensure your FastAPI application connects to the PostgreSQL database using environment variables.

### Step 5: Adminer Setup
1. Adminer is already configured in the `docker-compose.yml` to run on port 8080. You can access it via `http://your-domain.com:8080`.

### Step 6: Proxy Manager Setup
1. Set up Nginx Proxy Manager by adding the necessary Docker service in `docker-compose.yml`:
    ```yaml
    proxy:
        image: jc21/nginx-proxy-manager:latest
        ports:
          - "8090:81"
          - "80:80"
          - "443:443"
        volumes:
          - ./data:/data
          - ./letsencrypt:/etc/letsencrypt
        environment:
          DB_MYSQL_HOST: "db"
          DB_MYSQL_PORT: 3306
          DB_MYSQL_USER: "npm"
          DB_MYSQL_PASSWORD: "npm"
          DB_MYSQL_NAME: "npm"
        depends_on:
          - db
    ```

### Step 7: Cloud Deployment
1. **Set up your EC2 instance:**
   - Launch an EC2 instance (Amazon Linux 2 or Ubuntu).
   - Open ports 80, 443, 8080, and 8090 in your security group.

2. **Install Docker and Docker Compose on the EC2 instance:**
    ```sh
    sudo apt update
    sudo apt install docker.io docker-compose -y
    sudo usermod -aG docker $USER
    logout and login again
    ```

3. **Clone your repository on the EC2 instance:**
    ```sh
    git clone https://github.com/your-username/devops-stage-2.git
    cd devops-stage-2
    ```

4. **Run Docker Compose:**
    ```sh
    sudo docker-compose up -d
    ```

### Step 8: Domain and HTTPS Configuration
1. Use Afraid DNS to get a free subdomain and point it to your EC2 instance's public IP.
2. Configure Nginx Proxy Manager to handle HTTPS:
    - Access Nginx Proxy Manager at `http://your-domain.com:8090`.
    - Set up a proxy host for `your-domain.com` and enable HTTPS with Let's Encrypt.

3. **Redirect HTTP to HTTPS:**
    - Configure the redirection in Nginx Proxy Manager under SSL settings.

4. **Redirect www to non-www:**
    - Add a new proxy host for `www.your-domain.com` and set it to redirect to `your-domain.com`.

### Final Checks:
- Ensure the frontend is accessible at `http://your-domain.com`.
- Ensure the backend is accessible at `http://your-domain.com/api`, `http://your-domain.com/docs`, and `http://your-domain.com/redoc`.
- Adminer should be accessible at `http://db.your-domain.com`.
- Proxy Manager should be accessible at `http://proxy.your-domain.com`.

This guide should help you deploy your Dockerized full stack application on an AWS EC2 instance with proper proxy configurations. If you encounter any issues, feel free to ask for further assistance.
