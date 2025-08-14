[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

# Patient Management System

A production‑ready **Spring Boot REST API** for managing patients, doctors, and appointments. Ships with **PostgreSQL**, **Docker/Docker Compose**, database migrations via **Flyway**, and interactive docs via **Swagger/OpenAPI**. Secure by default (JWT optional), testable, and easy to deploy.

---

## ✨ Features

* CRUD for **Patients**, **Doctors**, **Appointments**
* Search & pagination
* Soft delete & audit timestamps
* Validation & error handling (RFC‑7807 style problem details)
* OpenAPI (Swagger UI)
* Flyway database migrations
* Spring Security (JWT bearer auth – optional)
* Test suite (JUnit 5, Testcontainers)
* Docker & Docker Compose for local dev and prod

---

## 🧱 Tech Stack

* **Java 17+**, **Spring Boot 3** (Web, Data JPA, Validation, Security)
* **PostgreSQL 15**
* **Flyway** for schema versioning
* **Lombok**, **MapStruct** (optional)
* **Testcontainers**, **JUnit 5**, **Mockito**

---

## 📁 Project Structure

```
patient-management/
├─ src/
│  ├─ main/java/com/example/pms/
│  │  ├─ config/           # Security, OpenAPI, CORS, Jackson
│  │  ├─ controller/       # REST controllers
│  │  ├─ dto/              # Request/response models
│  │  ├─ entity/           # JPA entities
│  │  ├─ exception/        # GlobalExceptionHandler, errors
│  │  ├─ mapper/           # MapStruct mappers (optional)
│  │  ├─ repository/       # Spring Data repositories
│  │  ├─ service/          # Business logic
│  │  └─ PatientManagementApplication.java
│  ├─ main/resources/
│  │  ├─ application.yml
│  │  └─ db/migration/     # V1__init.sql, ... (Flyway)
│  └─ test/java/...        # Unit + integration tests
├─ Dockerfile
├─ docker-compose.yml
├─ Makefile (optional)
├─ pom.xml
└─ README.md
```

---

## 🗄️ Data Model (ERD)

```
Doctor (id, first_name, last_name, email, specialization)
Patient (id, first_name, last_name, email, phone, dob, gender)
Appointment (id, patient_id, doctor_id, start_time, end_time, notes, status)

Relationships:
- Doctor 1..* Appointment *..1 Patient
```

---

## 🚀 Getting Started

### Prerequisites

* Java 17+
* Maven 3.9+
* Docker & Docker Compose

### Environment Variables

Configure in `application.yml` or via env vars:

```
SPRING_PROFILES_ACTIVE=dev
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/pms
SPRING_DATASOURCE_USERNAME=pms
SPRING_DATASOURCE_PASSWORD=pms
JWT_SECRET=change-me-please
```

### Run Locally (no Docker)

```bash
# start postgres (if you have it locally)
psql -c 'CREATE DATABASE pms;'
psql -c "CREATE USER pms WITH PASSWORD 'pms';"
psql -c 'GRANT ALL PRIVILEGES ON DATABASE pms TO pms;'

# run the app
./mvnw spring-boot:run
```

App runs at: `http://localhost:8080`
Swagger UI: `http://localhost:8080/swagger-ui/index.html`

### Run with Docker (recommended)

```bash
# build jar
./mvnw -DskipTests package

# build and run via compose
docker compose up --build
```

This starts:

* `api` at `http://localhost:8080`
* `postgres` at `localhost:5432`

### Quick Makefile (optional)

```Makefile
run:       ## Run locally (requires local Postgres)
	./mvnw spring-boot:run

docker:    ## Build and run with Docker Compose
	docker compose up --build

test:      ## Run tests
	./mvnw test

clean:     ## Clean build
	./mvnw clean
```

---

## 🧪 Testing

* **Unit tests**: services and mappers
* **Integration tests**: repositories & controllers with **Testcontainers**

```bash
./mvnw test
```

---

## 🔐 Security (Optional JWT)

* By default, endpoints can be configured as public for demo, or protected with JWT.
* To enable JWT, set `security.enabled=true` and provide `JWT_SECRET`.
* Use `/api/auth/login` to obtain a token.

Authorization header:

```
Authorization: Bearer <your-jwt>
```

---

## 📜 API Reference

Base URL: `http://localhost:8080/api/v1`

### Patients

* `GET /patients` – list (supports `page`, `size`, `q`)
* `GET /patients/{id}` – get by id
* `POST /patients` – create
* `PUT /patients/{id}` – replace
* `PATCH /patients/{id}` – partial update
* `DELETE /patients/{id}` – soft delete

**Create Patient (example)**

```bash
curl -X POST http://localhost:8080/api/v1/patients \
  -H 'Content-Type: application/json' \
  -d '{
    "firstName": "Aarav",
    "lastName": "Sharma",
    "email": "aarav@example.com",
    "phone": "+91-9999999999",
    "dob": "2001-08-09",
    "gender": "MALE"
  }'
```

### Doctors

* `GET /doctors`
* `GET /doctors/{id}`
* `POST /doctors`
* `PUT /doctors/{id}`
* `DELETE /doctors/{id}`

### Appointments

* `GET /appointments?patientId=&doctorId=&from=&to=`
* `GET /appointments/{id}`
* `POST /appointments`
* `PUT /appointments/{id}`
* `DELETE /appointments/{id}`

**Create Appointment (example)**

```bash
curl -X POST http://localhost:8080/api/v1/appointments \
  -H 'Content-Type: application/json' \
  -d '{
    "patientId": 1,
    "doctorId": 1,
    "startTime": "2025-08-14T10:00:00",
    "endTime":   "2025-08-14T10:30:00",
    "notes": "Initial consultation"
  }'
```

---

## 🧰 Error Handling

* Uses Problem Details (HTTP RFC 7807) shape:

```json
{
  "type": "https://example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 400,
  "detail": "email must be a valid address",
  "instance": "/api/v1/patients"
}
```

---

## 🧾 OpenAPI & Swagger

* OpenAPI JSON: `/v3/api-docs`
* Swagger UI: `/swagger-ui/index.html`

Minimal config example (YAML):

```yaml
springdoc:
  swagger-ui:
    path: /swagger-ui
  api-docs:
    path: /v3/api-docs
```

---

## 🗃️ Database & Migrations

Example `V1__init.sql`:

```sql
CREATE TABLE doctors (
  id BIGSERIAL PRIMARY KEY,
  first_name VARCHAR(80) NOT NULL,
  last_name  VARCHAR(80) NOT NULL,
  email      VARCHAR(120) UNIQUE NOT NULL,
  specialization VARCHAR(120),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE patients (
  id BIGSERIAL PRIMARY KEY,
  first_name VARCHAR(80) NOT NULL,
  last_name  VARCHAR(80) NOT NULL,
  email      VARCHAR(120) UNIQUE,
  phone      VARCHAR(30),
  dob        DATE,
  gender     VARCHAR(10),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE appointments (
  id BIGSERIAL PRIMARY KEY,
  patient_id BIGINT NOT NULL REFERENCES patients(id),
  doctor_id  BIGINT NOT NULL REFERENCES doctors(id),
  start_time TIMESTAMP NOT NULL,
  end_time   TIMESTAMP NOT NULL,
  notes      TEXT,
  status     VARCHAR(20) DEFAULT 'SCHEDULED',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_appt_patient ON appointments(patient_id);
CREATE INDEX idx_appt_doctor  ON appointments(doctor_id);
CREATE INDEX idx_appt_start   ON appointments(start_time);
```

---

## 🐳 Docker

**Dockerfile** (JDK 17, minimized):

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn -q -e -DskipTests dependency:go-offline
COPY src ./src
RUN mvn -q -DskipTests package

# Runtime stage
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/target/patient-management-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

**docker-compose.yml**:

```yaml
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: pms
      POSTGRES_USER: pms
      POSTGRES_PASSWORD: pms
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U pms -d pms"]
      interval: 5s
      timeout: 5s
      retries: 10

  api:
    build: .
    depends_on:
      db:
        condition: service_healthy
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/pms
      SPRING_DATASOURCE_USERNAME: pms
      SPRING_DATASOURCE_PASSWORD: pms
      JWT_SECRET: change-me-please
    ports:
      - "8080:8080"

volumes:
  db_data:
```

---

## 🔄 Seeding (optional)

Add `V2__seed.sql` with demo data for quick testing.

---

## 🧰 Useful cURL Snippets

```bash
# List patients (page 0, size 20)
curl 'http://localhost:8080/api/v1/patients?page=0&size=20'

# Search patients by query (name/email)
curl 'http://localhost:8080/api/v1/patients?q=aarav'

# Delete patient (soft)
curl -X DELETE 'http://localhost:8080/api/v1/patients/1'
```

---

## 🧩 Configuration

```yaml
server:
  port: 8080

spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5432/pms}
    username: ${SPRING_DATASOURCE_USERNAME:pms}
    password: ${SPRING_DATASOURCE_PASSWORD:pms}
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        format_sql: true
        jdbc.lob.non_contextual_creation: true
  flyway:
    enabled: true

security:
  enabled: false  # set true to enable JWT

logging:
  level:
    root: INFO
    org.springframework.web: INFO
```

---

## 🧱 Build & Deployment

* Local Docker image: `docker build -t pms-api:latest .`
* Run: `docker run -p 8080:8080 --env-file .env pms-api:latest`
* Push to registry: tag and push (`docker tag`, `docker push`)
* Deploy behind Nginx/Traefik with HTTPS

---

## 🤝 Contributing

1. Fork the repo & create a feature branch
2. Write tests for changes
3. Ensure `./mvnw test` passes
4. Open a PR with a clear description

---

## 🐞 Troubleshooting

* **Port 8080 in use** → set `server.port=8081` or free the port
* **Flyway errors** → check your `db/migration` scripts & DB credentials
* **Cannot connect to DB** → verify `SPRING_DATASOURCE_URL` and Compose healthcheck
* **Swagger not loading** → ensure springdoc starter is on classpath

---

## 📄 License

MIT (or your preferred license).

---

## 📌 Badges (optional)

```
![Build](https://img.shields.io/badge/build-passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
```

> Tip: Replace placeholder values (e.g., `change-me-please`) before deploying to production.
