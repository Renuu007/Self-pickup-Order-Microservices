# 🛍️ Self-pickup Order Microservices

This project implements a backend system for a self-pickup e-commerce application using a microservices architecture built with Go, Docker, and PostgreSQL.

---

## ✨ Features

*   **Microservices Architecture:** Ensures scalability and separation of concerns.
*   **API Gateway:** Single entry point for all client requests (HTTP).
*   **Authentication Service:** Manages user registration, login, and authorization (JWT likely).
*   **Product Service:** Handles product catalog management.
*   **Order Service:** Manages order creation and processing.
*   **Dockerized:** Fully containerized using Docker and Docker Compose for easy setup and deployment.
*   **Dedicated Databases:** Each core service (Auth, Product, Order) has its own PostgreSQL database instance.

---

## 📂 Folder Structure

```
.
├── 📄 .gitignore
├── 🐳 docker-compose.yml      # Defines and configures all services and databases
├── 🛠️ Makefile                # Utility commands (like building images)
├── 🖼️ Microservice Diagram-3.png # Original diagram image
├── 📝 readme.md               # This file!
├── 🚪 api-gateaway/           # API Gateway Service (Gin) - Entry point for clients
│   ├── cmd/main.go          # Main application entry
│   ├── internal/            # Internal logic (routing, handlers, clients)
│   ├── Dockerfile           # Instructions to build the service image
│   └── go.mod / go.sum      # Go module dependencies
├── 🔑 auth/                   # Authentication Service (gRPC)
│   ├── cmd/main.go          # Main application entry
│   ├── internal/            # Internal logic (handlers, service, repo, pb)
│   ├── Dockerfile           # Instructions to build the service image
│   └── go.mod / go.sum      # Go module dependencies
├── 📦 product/               # Product Service (gRPC)
│   ├── cmd/main.go          # Main application entry
│   ├── internal/            # Internal logic (handlers, service, repo, pb)
│   ├── Dockerfile           # Instructions to build the service image
│   └── go.mod / go.sum      # Go module dependencies
└── 🛒 order/                 # Order Service (gRPC)
    ├── cmd/main.go          # Main application entry
    ├── internal/            # Internal logic (handlers, service, repo, pb, client)
    ├── Dockerfile           # Instructions to build the service image
    └── go.mod / go.sum      # Go module dependencies
```

---

## 🚀 Getting Started

Follow these steps to get the application running locally.

### Prerequisites

*   ✅ [Docker](https://docs.docker.com/get-docker/) installed and running.
*   ✅ [Docker Compose](https://docs.docker.com/compose/install/) installed.
*   ✅ Git (for cloning, if you haven't already).
*   ✅ A terminal or command prompt (like PowerShell, CMD, bash).
*   ✅ An API Client (like [Postman](https://www.postman.com/) or [Insomnia](https://insomnia.rest/)) for testing POST/PUT etc. endpoints.

### Installation & Running

1.  **Clone the Repository (if you haven't):**
    ```bash
    git clone <repository_url>
    cd <repository_directory>
    ```

2.  **Build Docker Images:**
    The required Docker images are not available publicly and need to be built locally from the source code. Run the following commands in your terminal from the project's **root directory**:

    *   **API Gateway:**
        ```bash
        # Windows (CMD/PowerShell):
        cd api-gateaway
        docker build -t magistra/ecom-api-gateaway .
        cd ..

        # Linux/macOS:
        # cd api-gateaway && docker build -t magistra/ecom-api-gateaway . && cd .. 
        ```
    *   **Auth Service:**
        ```bash
        # Windows (CMD/PowerShell):
        cd auth
        docker build -t magistra/ecom-auth-service .
        cd ..

        # Linux/macOS:
        # cd auth && docker build -t magistra/ecom-auth-service . && cd ..
        ```
    *   **Product Service:**
        ```bash
        # Windows (CMD/PowerShell):
        cd product
        docker build -t magistra/ecom-product-service .
        cd ..

        # Linux/macOS:
        # cd product && docker build -t magistra/ecom-product-service . && cd ..
        ```
    *   **Order Service:**
        ```bash
        # Windows (CMD/PowerShell):
        cd order
        docker build -t magistra/ecom-order-service .
        cd ..

        # Linux/macOS:
        # cd order && docker build -t magistra/ecom-order-service . && cd ..
        ```
    *(Alternatively, if you have `make` installed, you can run `make docker-build` from the root directory).*

3.  **Run the Application:**
    Once all images are built, start all services using Docker Compose:
    ```bash
    docker-compose up -d
    ```
    The `-d` flag runs the containers in detached mode (in the background).

4.  **Verify Services:**
    Check if all containers are running without restarting:
    ```bash
    docker ps
    ```
    You should see `api-gateaway`, `auth-service`, `product-service`, `order-service`, and the three database containers (`auth-db`, `product-db`, `order-db`) with status `Up`.

---

## ⚙️ API Interaction

*   The **API Gateway** is the single entry point, accessible at `http://localhost:8000`.
*   You **must** use an API client tool (like Postman or Insomnia) or a command-line tool (`curl`, `Invoke-RestMethod`) to interact with endpoints that require methods other than `GET` (e.g., `POST`, `PUT`, `DELETE`).
*   Trying to access `POST`-only endpoints (like `/auth/register` or `/auth/login`) directly in your **web browser will result in a "404 Not Found" error**, because your browser sends a `GET` request, and the server is correctly configured to only accept `POST` for those specific paths.

### Key Endpoints (Requires API Client):

*   `POST /auth/register`: Register a new user. Body: `{"email": "user@example.com", "password": "your_password"}`
*   `POST /auth/login`: Log in a user. Body: `{"email": "user@example.com", "password": "your_password"}`
*   `POST /admin/register`: Register a new admin. Body: `{"email": "admin@example.com", "password": "your_password"}`
*   `POST /admin/login`: Log in an admin. Body: `{"email": "admin@example.com", "password": "your_password"}`
*   *(Product and Order endpoints likely exist under `/products` and `/orders` but require authentication. Explore the respective `routes.go` files for details)*

---

## 🌊 Workflow Diagram (Example: User Registration)

```
                                              ┌────────────────────┐
                                              │ Client             │
                                              │ (Browser/Postman)  │
                                              └─────────┬──────────┘
                                                        │ HTTP Request (e.g., POST /orders)
                                                        ▼
              +-----------------------------------------------------------------------------------+
              |       Docker Container                                                            |
              |                               +--------------------+                              |
              |                               │ API Gateway (:8000)│                              |
              |                               │ (Gin)              │                              |
              |                               +---------┬----------+                              |
              |                                         │ Forward Request                         |
              |                                         │ (Checks Auth if needed*)                |
              |                                         │                                         |
              |         ┌───────────────────────────────*──────────────────────────────────┐      |
              +---------|-------------------------------│----------------------------------|------+
                        │                               │                                  │gRPC Requests
                        ▼                               ▼                                  ▼
       +-------------------------------+ +--------------------------------+ +-------------------------------+
       | Docker Container              | | Docker Container               | | Docker Container              |
       |  +-------------------------+  | |  +-------------------------+   | |  +-------------------------+  |
       |  │ Auth Service*           │  | |  │ Product Service         │   | |  │ Order Service           │  |
       |  │ (:50051) [gRPC]         │  | |  │ (:50053) [gRPC]         │   | |  │ (:50052) [gRPC]         │  |
       |  +-----------┬-------------+  | |  +-----------┬-------------+   | |  +-----------┬-------------+  |
       |              │ SQL            | |              │ SQL             | |              │ SQL            |
       |              ▼                | |              ▼                 | |              ▼                |
       |  +-------------------------+  | |  +-------------------------+   | |  +-------------------------+  |
       |  │ Auth Database           │  | |  │ Product Database        │   | |  │ Order Database          │  |
       |  │ (:5433) [Postgres]      │  | |  │ (:5434) [Postgres]      │   | |  │ (:5435) [Postgres]      │  |
       |  +-------------------------+  | |  +-------------------------+   | |  +-------------------------+  |
       +-------------------------------+ +--------------------------------+ +-------------------------------+

```
*Note: For protected endpoints, API Gateway calls Auth Service first to validate the token.
 The called service (e.g., Order Service) might then call other services (e.g., Product Service).*
