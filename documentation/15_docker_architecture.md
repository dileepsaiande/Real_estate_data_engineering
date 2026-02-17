# Docker Architecture Documentation

## 1. Executive Summary

PropIntel's Docker architecture provides a consistent, portable runtime environment that encapsulates the complex dependency chain required for the application. By packaging Python 3.11, OpenJDK 17 (for Spark), and all necessary libraries into a single **Docker Image**, we ensure that the application functions identically across development (local), testing, and production (Kubernetes) environments, eliminating "it works on my machine" issues—especially critical for Windows users needing Hadoop binaries.

---

## 2. Container Design

### 2.1 The Dockerfile Explained

The `Dockerfile` follows a multi-step build process to optimize image size and security.

#### Base Image Selection
```dockerfile
FROM python:3.11-slim-bookworm
```
- **Why Slim?**: Reduces image footprint (~200MB vs ~1GB).
- **Why Bookworm?**: Stable Debian 12 base, ensuring long-term security updates.

#### System Dependencies (The "Heavy Lifting")
PySpark requires a Java Runtime Environment (JRE).
```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    openjdk-17-jre-headless \
    procps \
    curl \
    build-essential \
    && rm -rf /var/lib/apt/lists/*
```
- **openjdk-17**: Required by Spark 3.x.
- **procps**: Adds `ps`, which Spark's driver uses to monitor executors.
- **clean up**: `rm -rf` keeps the layer size small.

#### Environment Configuration
Sets critical paths for Spark to find Java and Python.
```dockerfile
ENV JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
ENV PYSPARK_PYTHON=python3
```

#### Application Setup
```dockerfile
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```
- **Layer Caching**: `requirements.txt` is copied *before* the source code. This ensures that changing a Python file doesn't trigger a full re-installation of dependencies.

---

## 3. Docker Compose Architecture

`docker-compose.yml` orchestrates the runtime configuration for local development.

### 3.1 Service Definition

```yaml
version: '3.8'
services:
  propintel-app:
    build: .
    ports:
      - "8501:8501"
    volumes:
      - ./data:/app/data
      - ./models:/app/models
    environment:
      - OLLAMA_HOST=http://host.docker.internal:11434
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

### 3.2 Key Configurations

#### Volume Mounting
- **Purpose**: Persistence and Live Updates.
- **Mapping**: `./data:/app/data` ensures that if a container crashes, the Admin's CSV edits are safe on the host.

#### Networking (Ollama Access)
- **Problem**: Docker containers cannot access localhost ports (where Ollama runs) by default.
- **Solution**: `extra_hosts` maps `host.docker.internal` to the host's specialized gateway IP, allowing the specific API calls to pass through.

---

## 4. Build & Run Workflow

### 4.1 Building the Image
```bash
docker build -t propintel:latest .
```
*Time taken*: ~2-3 mins (first run), < 10s (cached).

### 4.2 Running the Container
```bash
docker run -p 8501:8501 -v $(pwd)/data:/app/data propintel:latest
```

---

## 5. Security Best Practices

| Feature | Implementation | Status |
|---------|----------------|--------|
| **Non-Root User** | Running as `appuser` | ❌ Pending (Currently runs as root) |
| **Image Scanning** | Trivy/Snyk scans | ✅ Base image is regularly patched |
| **Secret Management** | `.env` files | ⚠️ Secrets (if any) currently in code |
| **Health Checks** | `HEALTHCHECK CMD` | ✅ Implemented via curl |

---

## 6. Optimization Techniques

### 6.1 .dockerignore
Excludes unnecessary files from the build context to speed up `COPY` operations.
```text
__pycache__
.git
.env
data/*.csv (Prevent overwriting production data with dev test data)
```

### 6.2 Multi-Stage Builds (Future)
Could split the build into a "builder" stage (compiling C extensions) and a "runtime" stage (pure copy), potentially shaving off another 100MB.

---

## 7. Troubleshooting

### Common Issue: "Java not found"
- **Cause**: Incorrect `JAVA_HOME` path.
- **Fix**: The Dockerfile hardcodes the Debian-specific path manually verified.

### Common Issue: "Exited with code 137"
- **Cause**: OOM (Out of Memory). Spark is memory hungry.
- **Fix**: Increase Docker Desktop RAM allocation to at least 4GB.
