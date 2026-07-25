---
description: Setup CI/CD pipeline with GitHub Actions to build Spring Boot app, push Docker image to GHCR, and deploy to VPS via SSH.
---

# CI/CD VPS Deploy Workflow

## Purpose

Setup a complete CI/CD pipeline for a Spring Boot microservice:
- Build JAR with Gradle
- Build Docker Image (multi-stage)
- Push to GitHub Container Registry (GHCR)
- Deploy to VPS via SSH + Docker Compose

---

## Input

- Service name (e.g., `media-service`, `auth-service`)
- VPS deploy directory (e.g., `/dts/media`)
- Host port for the service (e.g., `8080`, `8081`)

---

## Preconditions

- Project is a Spring Boot application with Gradle.
- VPS has Docker, Docker Compose installed.
- VPS has `docker login ghcr.io` completed.
- VPS has shared Docker network `dts-network` created (`docker network create dts-network`).
- GitHub repository has 3 secrets configured: `VPS_HOST`, `VPS_USER`, `VPS_PASSWORD`.

If any precondition is missing,

STOP.

Inform the user what is missing and how to set it up.

---

## VPS Architecture

```
/dts/
├── infra/                    ← Shared services (setup once)
│   ├── docker-compose.yml    ← Postgres, Kafka, MinIO, Redis...
│   └── .env
│
├── <service-1>/              ← App 1
│   └── docker-compose.yml    ← Only this service's container
│
├── <service-2>/              ← App 2
│   └── docker-compose.yml    ← Only this service's container
```

All services communicate via shared Docker network `dts-network`.
Each app uses a different host port (8080, 8081, 8082...).
Infrastructure services are referenced by container name (e.g., `dts-postgres`, `dts-kafka`).

---

## Steps

### Step 0 — Gather Information

Determine from user or existing project:

1. **Service name**: from `settings.gradle` → `rootProject.name`
2. **GitHub owner**: from `git remote -v`
3. **Java version**: from `build.gradle` → `toolchain.languageVersion`
4. **Deploy directory**: `/dts/<service-name>`
5. **Host port**: ask user or assign next available (8080, 8081, 8082...)
6. **Dependencies**: which shared services does this app need? (Postgres, Kafka, MinIO, Redis...)

---

### Step 1 — Create Dockerfile

Create `Dockerfile` at project root with multi-stage build.

**Template:**

```dockerfile
# Stage 1: Build
FROM eclipse-temurin:<java-version>-jdk AS builder
WORKDIR /app
COPY gradle/wrapper/ gradle/wrapper/
COPY gradlew .
COPY build.gradle settings.gradle ./
RUN chmod +x gradlew && ./gradlew dependencies --no-daemon
COPY src/ src/
RUN ./gradlew bootJar --no-daemon -x test

# Stage 2: Runtime
FROM eclipse-temurin:<java-version>-jre AS runtime
WORKDIR /app
RUN groupadd --system appgroup && useradd --system --gid appgroup appuser
COPY --from=builder /app/build/libs/*.jar app.jar
RUN chown appuser:appgroup app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Rules:
- Java version in Dockerfile MUST match `build.gradle` and workflow.
- Always use non-root user for security.
- Use multi-stage to reduce image size.

---

### Step 2 — Create `.dockerignore`

Create `.dockerignore` at project root.

**Template:**

```
build/
.gradle/
.git/
.idea/
.vscode/
*.md
docs/
plan/
Dockerfile
docker-compose*.yml
.dockerignore
.agents/
.antigravity-ide/
```

---

### Step 3 — Create `docker-compose.prod.yml`

Create `docker-compose.prod.yml` at project root.
This file is for VPS production — only contains this service's container.

**Template:**

```yaml
services:
  <service-name>:
    image: ghcr.io/<github-owner>/<service-name>:latest
    container_name: <service-name>
    restart: unless-stopped
    ports:
      - "<host-port>:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://dts-postgres:5432/<db-name>
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
      SPRING_KAFKA_BOOTSTRAP-SERVERS: dts-kafka:9092
      # Add other environment variables as needed
    networks:
      - dts-network

networks:
  dts-network:
    external: true
```

Rules:
- Only include environment variables for dependencies this service actually uses.
- Use `dts-network` external network.
- Reference shared services by container name (e.g., `dts-postgres`, `dts-kafka`, `dts-minio`, `dts-redis`).
- Use `${DB_PASSWORD}` from `.env` file on VPS — never hardcode passwords.

---

### Step 4 — Create GitHub Actions Workflow

Create `.github/workflows/deploy.yml`.

**Template:**

```yaml
name: Build & Deploy

on:
  push:
    branches:
      - main

permissions:
  contents: read
  packages: write

env:
  IMAGE: ghcr.io/${{ github.repository_owner }}/<service-name>
  DEPLOY_DIR: /dts/<service-name>

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Setup JDK <java-version>
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: <java-version>

      - name: Cache Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Build Jar
        run: |
          chmod +x gradlew
          ./gradlew clean bootJar

      - name: Login GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build & Push Docker Image
        run: |
          docker build \
            -t $IMAGE:latest \
            -t $IMAGE:${{ github.sha }} .
          docker push $IMAGE:latest
          docker push $IMAGE:${{ github.sha }}

      - name: Copy docker-compose to VPS
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          password: ${{ secrets.VPS_PASSWORD }}
          source: "docker-compose.prod.yml"
          target: ${{ env.DEPLOY_DIR }}

      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1.2.2
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          password: ${{ secrets.VPS_PASSWORD }}
          script: |
            cd ${{ env.DEPLOY_DIR }}
            mv -f docker-compose.prod.yml docker-compose.yml
            docker compose pull
            docker compose up -d
```

Rules:
- `chmod +x gradlew` is REQUIRED (Windows dev loses execute permission).
- Java version MUST match `build.gradle` and `Dockerfile`.
- Only trigger on `push` to `main` (merge PR into main already triggers push).
- Do NOT add `pull_request` trigger — it causes duplicate runs and permission issues with GHCR.

---

### Step 5 — Verify GitHub Secrets

Check that the repository has these 3 secrets configured:

| Secret Name    | Description              |
|---------------|--------------------------|
| `VPS_HOST`    | VPS IP address           |
| `VPS_USER`    | SSH username             |
| `VPS_PASSWORD`| SSH password             |

If secrets are missing, instruct user to go to:
`https://github.com/<owner>/<repo>/settings/secrets/actions`

---

### Step 6 — Verify VPS Setup

Instruct user to run on VPS:

```bash
# Create deploy directory
mkdir -p /dts/<service-name>

# Verify GHCR login
docker login ghcr.io -u <github-username>

# Verify shared network exists
docker network ls | grep dts-network

# Create database if needed
docker exec -it dts-postgres psql -U postgres -c "CREATE DATABASE <db_name>;"
```

---

## Output

- `Dockerfile` (multi-stage build)
- `.dockerignore`
- `docker-compose.prod.yml` (production, VPS)
- `.github/workflows/deploy.yml` (GitHub Actions)

---

## Validation Checklist

- [ ] Java version is consistent across `build.gradle`, `Dockerfile`, and workflow
- [ ] `chmod +x gradlew` is present in workflow Build step
- [ ] Workflow only triggers on `push` to `main` (no `pull_request`)
- [ ] `docker-compose.prod.yml` uses `dts-network` external network
- [ ] `docker-compose.prod.yml` references shared services by container name
- [ ] No hardcoded passwords in any file
- [ ] `.dockerignore` excludes build artifacts and IDE files
- [ ] GitHub Secrets are configured (VPS_HOST, VPS_USER, VPS_PASSWORD)
- [ ] VPS has `docker login ghcr.io` completed
- [ ] VPS has deploy directory created
- [ ] Host port does not conflict with other services

---

## Port Allocation Convention

| Service           | Host Port |
|-------------------|-----------|
| media-service     | 8080      |
| auth-service      | 8081      |
| notification-service | 8082   |
| gateway           | 80 / 443  |

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `./gradlew: Permission denied` | Windows dev loses file permissions | Add `chmod +x gradlew` before build |
| `denied: permission_denied` on GHCR push | Token lacks `write:packages` scope | Check `permissions: packages: write` in workflow |
| Duplicate workflow runs | `pull_request: closed` trigger | Remove `pull_request` trigger, keep only `push` |
| Container can't reach Postgres/Kafka | Not on same Docker network | Add `networks: dts-network` to docker-compose |
| `unauthorized` on VPS docker pull | GHCR login expired | Re-run `docker login ghcr.io` on VPS |
