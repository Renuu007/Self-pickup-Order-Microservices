# Self-pickup-Order-Microservices Documentation

This document provides an overview of the Self-pickup-Order-Microservices project, details the technologies used, explains the rationale for these choices, and offers guidance on orchestrating multiple Docker Compose files. It serves as a reference for developers working on or maintaining the project.

## Project Overview

The Self-pickup-Order-Microservices project is a backend system for a self-pickup ordering service, enabling customers to place orders online and pick them up at a designated location (e.g., a restaurant or store). Built using a microservices architecture with Go (Golang), it ensures scalability, maintainability, and efficiency.

- **Purpose**: Manages order processing, user authentication, and related tasks for a self-pickup system.
- **Architecture**: Comprises independent microservices (e.g., Auth, Product, Order) communicating via gRPC, with an API Gateway handling external requests.
- **Repository**: Hosted at [https://github.com/Renuu007/Self-pickup-Order-Microservices](https://github.com/Renuu007/Self-pickup-Order-Microservices).

## Technologies Used

The project leverages a modern tech stack tailored for microservices. Below is a summary of the technologies and their roles:

| **Category**            | **Technology**               | **Details**                                                                 |
|-------------------------|------------------------------|-----------------------------------------------------------------------------|
| Programming Language    | Go (Golang)                  | Primary language for building scalable microservices.                       |
| Microservices Framework | gRPC                         | Handles communication between Auth, Product, and Order services.            |
| API Gateway             | Gin                          | Manages external HTTP/REST API requests from clients.                       |
| Database                | PostgreSQL                   | Relational database with instances on ports 5433 (Auth), 5434 (Product), 5435 (Order). |
| Containerization        | Docker, Docker Compose       | Docker for containerizing services, Docker Compose for orchestration.       |
| Build Tools             | Makefile                     | Automates build tasks (e.g., creating Docker images).                       |
| Version Control         | Git                          | Manages code versioning, hosted on GitHub.                                  |
| API Testing Tools       | Postman, Insomnia            | Used for testing and documenting APIs.                                     |
| Prerequisites           | Docker, Docker Compose       | Required for running the project locally.                                  |

### Key Technology Definitions

- **gRPC**: A high-performance RPC framework using HTTP/2 and Protocol Buffers for efficient service-to-service communication.
- **Gin**: A lightweight Go web framework for building fast REST APIs, used for the API Gateway.

## Why These Technologies?

The chosen technologies align with the project’s goals of performance, scalability, and maintainability. Below is the rationale for each, along with why alternatives were avoided:

1. **Go (Golang)**  
   - **Why Chosen**: Go’s concurrency model (goroutines), performance, and simplicity make it ideal for microservices. Its standard library supports HTTP servers and database interactions efficiently.
   - **Why Not Others**:
     - **Python**: Slower due to its interpreted nature and GIL, less suited for high-performance backends.
     - **Java**: Heavier memory usage and slower startup times compared to Go.
     - **Node.js**: Less efficient for CPU-bound tasks; Go’s compiled nature is better for performance.
     - **C#/.NET**: Tied to Microsoft ecosystems, less flexible than Go.

2. **gRPC (Inter-Service Communication)**  
   - **Why Chosen**: gRPC’s HTTP/2 and Protocol Buffers enable low-latency, high-throughput communication, critical for frequent service interactions (e.g., order validation). It supports streaming and auto-generates code, simplifying development.
   - **Performance**: 5-10x faster than REST (e.g., 50ms vs. 500ms for high-load calls) due to binary payloads and multiplexing.
   - **Scalability**: Efficient connections and load balancing support scaling under heavy traffic.
   - **Why Not Others**:
     - **REST**: Slower due to JSON and HTTP/1.1, with higher latency and bandwidth usage.
     - **GraphQL**: Designed for client-server, not service-to-service; adds unnecessary complexity.
     - **Message Queues (e.g., Kafka)**: Better for asynchronous events, not synchronous RPCs.

3. **Gin (API Gateway)**  
   - **Why Chosen**: Gin’s high performance (~100,000 requests/second), low memory footprint, and middleware support make it ideal for the API Gateway. It integrates seamlessly with Go.
   - **Performance**: Faster than alternatives like Express.js or Spring Boot, with minimal latency.
   - **Scalability**: Stateless design allows horizontal scaling with load balancers.
   - **Why Not Others**:
     - **Express.js**: Slower (~10,000 requests/second) and JavaScript-based, less efficient.
     - **Spring Boot**: Higher memory usage (~200MB vs. Gin’s ~10MB) and slower startup.
     - **GraphQL**: Adds query parsing overhead and complexity, unnecessary for fixed REST endpoints.

4. **PostgreSQL**  
   - **Why Chosen**: Robust relational database with support for complex queries and transactions. Separate instances ensure data isolation for microservices.
   - **Why Not Others**:
     - **MySQL**: Fewer advanced features (e.g., JSON support).
     - **MongoDB**: NoSQL, less suited for structured order/user data.
     - **SQLite**: Not designed for high concurrency or distributed systems.

5. **Docker and Docker Compose**  
   - **Why Chosen**: Docker ensures consistent environments, while Docker Compose simplifies orchestration for development and small-scale deployments.
   - **Why Not Others**:
     - **Kubernetes**: Too complex for development; better for production at scale.
     - **Podman**: Less mature than Docker.
     - **Bare Metal**: Prone to environment inconsistencies.

6. **Makefile, Git, Postman/Insomnia**  
   - **Why Chosen**: Makefile automates builds, Git is standard for version control, and Postman/Insomnia simplify API testing.
   - **Why Not Others**: Alternatives (e.g., Bash scripts, SVN, cURL) are less standardized or feature-rich.

## Orchestrating Multiple Docker Compose Files

The project likely uses Docker Compose to manage services (e.g., Auth, Product, Order, PostgreSQL). If multiple Compose files exist (e.g., for different services or environments), here’s how to orchestrate them:

### Why Multiple Compose Files?
- **Modularity**: Separate files for services (e.g., `docker-compose.auth.yml`) or environments (e.g., `docker-compose.dev.yml`).
- **Environment-Specific Settings**: Tailor configurations for development vs. production.
- **Reusability**: Base files can be extended for specific use cases.

### Methods for Orchestration

1. **Using the `-f` Flag**  
   Combine multiple files, with later files overriding earlier ones:
   ```bash
   docker-compose -f docker-compose.base.yml -f docker-compose.dev.yml up -d
   ```
   - Example:
     ```yaml
     # docker-compose.base.yml
     version: '3.8'
     services:
       auth:
         image: auth-service
         ports:
           - "8081:8081"
     ```
     ```yaml
     # docker-compose.dev.yml
     version: '3.8'
     services:
       auth:
         environment:
           - DEBUG=true
         volumes:
           - ./auth:/app
     ```

2. **Using Override Files**  
   Docker Compose merges `docker-compose.yml` with `docker-compose.override.yml` automatically:
   ```bash
   docker-compose up -d
   ```
   - Example:
     ```yaml
     # docker-compose.yml
     version: '3.8'
     services:
       order:
         image: order-service
         ports:
           - "8082:8082"
     ```
     ```yaml
     # docker-compose.override.yml
     version: '3.8'
     services:
       order:
         environment:
           - LOG_LEVEL=debug
     ```

3. **Using Profiles**  
   Group services with profiles for selective execution:
   ```yaml
   # docker-compose.yml
   version: '3.8'
   services:
     auth:
       image: auth-service
       profiles: ["auth", "all"]
     product:
       image: product-service
       profiles: ["product", "all"]
   ```
   Run specific profiles:
   ```bash
   docker-compose --profile auth up -d
   ```

4. **Environment Variables**  
   Use `.env` files to parameterize configurations:
   ```yaml
   # docker-compose.yml
   version: '3.8'
   services:
     postgres:
       image: postgres:latest
       ports:
         - "${DB_PORT}:5432"
   ```
   ```bash
   # .env
   DB_PORT=5433
   ```

5. **Advanced Tools**  
   For production, consider Kubernetes or Docker Swarm for large-scale orchestration.

### Best Practices
- Use descriptive file names (e.g., `docker-compose.prod.yml`).
- Minimize duplication with base files and overrides.
- Define shared networks:
  ```yaml
  networks:
    default:
      name: app-network
  ```
- Validate configurations:
  ```bash
  docker-compose -f docker-compose.base.yml -f docker-compose.dev.yml config
  ```
- Document commands in the README.

## API Gateway Access

The API Gateway (built with Gin) handles external HTTP requests. No public URL is specified in the repository, as it’s likely designed for local development.

- **Local Access**: Check `docker-compose.yml` for the `api-gateway` service’s port (e.g., `8080`). Access it at `http://localhost:8080` after running `docker-compose up`.
- **Verification**: Use `docker ps` to confirm the exposed port.
- **Production**: In production, the gateway would be hosted on a domain (e.g., `api.example.com`), but this is not configured in the repo.

## Getting Started

1. **Prerequisites**:
   - Install Docker: [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)
   - Install Docker Compose: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

2. **Clone the Repository**:
   ```bash
   git clone https://github.com/Renuu007/Self-pickup-Order-Microservices
   cd Self-pickup-Order-Microservices
   ```

3. **Run the Project**:
   If using a single `docker-compose.yml`:
   ```bash
   docker-compose up -d
   ```
   For multiple files:
   ```bash
   docker-compose -f docker-compose.base.yml -f docker-compose.dev.yml up -d
   ```

4. **Test APIs**:
   Use Postman ([https://www.postman.com/](https://www.postman.com/)) or Insomnia ([https://insomnia.rest/](https://insomnia.rest/)) to test API endpoints.

## Future Considerations

- **Production Deployment**: Explore Kubernetes for scaling in production.
- **Monitoring**: Add tools like Prometheus or Grafana for service monitoring.
- **CI/CD**: Integrate GitHub Actions or Jenkins for automated deployments.

## References

- [Project Repository](https://github.com/Renuu007/Self-pickup-Order-Microservices)
- [Docker Documentation](https://docs.docker.com/)
- [Postman](https://www.postman.com/)
- [Insomnia](https://insomnia.rest/)