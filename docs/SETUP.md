# Development Environment Setup

## Prerequisites

Before starting SunsetGuard development, you need to install:

### 1. Java Development Kit (JDK) 21

**Download**: [Eclipse Temurin JDK 21](https://adoptium.net/temurin/releases/?version=21)

- Choose: Windows x64, JDK, .msi installer
- Install to default location: `C:\Program Files\Eclipse Adoptium\jdk-21.x.x`
- **Important**: Check "Add to PATH" during installation

**Verify installation:**
```bash
java --version
# Should show: openjdk version "21.x.x"
```

### 2. Apache Maven

**Download**: [Maven 3.9.9+](https://maven.apache.org/download.cgi)

**Installation Steps:**
1. Download `apache-maven-3.9.9-bin.zip`
2. Extract to: `C:\Program Files\Apache\maven`
3. Add to PATH:
   - Windows Search → "Environment Variables"
   - System variables → Edit "Path"
   - Add: `C:\Program Files\Apache\maven\bin`
4. Create `MAVEN_HOME` environment variable:
   - New system variable: `MAVEN_HOME` = `C:\Program Files\Apache\maven`

**Verify installation:**
```bash
mvn --version
# Should show: Apache Maven 3.9.x
```

### 3. Docker Desktop

**Download**: [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)

**Requirements:**
- Windows 11 (you have this ✓)
- WSL 2 backend (Docker installer sets this up)
- At least 8GB RAM (you have this ✓)

**Installation:**
1. Download and run installer
2. Follow setup wizard (use WSL 2, not Hyper-V)
3. Restart computer
4. Start Docker Desktop
5. Wait for "Docker Desktop is running" in system tray

**Verify installation:**
```bash
docker --version
# Should show: Docker version 24.x.x or higher

docker compose version
# Should show: Docker Compose version v2.x.x
```

**Important Docker Settings** (to optimize for 8GB RAM):
1. Open Docker Desktop
2. Settings → Resources
3. Set:
   - **Memory**: 4 GB (leave 4GB for Windows)
   - **CPUs**: 2
   - **Swap**: 1 GB

### 4. Git (should already be installed)

```bash
git --version
# You have this working ✓
```

### 5. IDE (Choose one)

#### Option A: IntelliJ IDEA Community (Recommended)
**Download**: [IntelliJ IDEA Community Edition](https://www.jetbrains.com/idea/download/)

**Why**: Best Java IDE, free, excellent Spring Boot support

**Plugins to install:**
- Lombok (should prompt automatically)
- Docker
- Database Navigator

#### Option B: VS Code
**Download**: [Visual Studio Code](https://code.visualstudio.com/)

**Extensions to install:**
- Extension Pack for Java (Microsoft)
- Spring Boot Extension Pack
- Docker
- Lombok Annotations Support

---

## First-Time Setup

### 1. Clone the Repository (already done ✓)

```bash
cd C:\Users\rohan\OneDrive\Desktop\Rohan\Projects
git clone https://github.com/rohanmeshram/SunsetGuard.git
cd SunsetGuard
```

### 2. Start Docker Services

```bash
# Start all infrastructure services
docker compose up -d

# Wait ~30 seconds for services to start
# Check all services are healthy:
docker compose ps
```

**Expected output:**
```
NAME                      STATUS              PORTS
sunsetguard-postgres      Up (healthy)        0.0.0.0:5432->5432/tcp
sunsetguard-redis         Up (healthy)        0.0.0.0:6379->6379/tcp
sunsetguard-rabbitmq      Up (healthy)        0.0.0.0:5672->5672/tcp, 0.0.0.0:15672->15672/tcp
```

**Access services:**
- RabbitMQ Management UI: http://localhost:15672 (user: `sunsetguard`, password: `sunsetguard_dev_password`)
- PostgreSQL: `localhost:5432` (user: `sunsetguard`, password: `sunsetguard_dev_password`, database: `sunsetguard`)
- Redis: `localhost:6379` (no password in dev)

### 3. Build the Project

```bash
# Build all modules
mvn clean install

# If you get compilation errors, try:
mvn clean install -DskipTests
```

**First build will take 5-10 minutes** (downloading dependencies)

### 4. Run Tests

```bash
# Run all tests
mvn test

# Run specific module tests
cd sunsetguard-server
mvn test
```

### 5. Run the Server (Phase 1+)

```bash
cd sunsetguard-server
mvn spring-boot:run

# Server will start on: http://localhost:8080
```

---

## Daily Development Workflow

```bash
# 1. Start Docker services (if not running)
docker compose up -d

# 2. Pull latest changes
git pull

# 3. Build (if dependencies changed)
mvn clean install

# 4. Run server
cd sunsetguard-server
mvn spring-boot:run

# 5. Make changes, test, commit
git add .
git commit -m "Your message"
git push
```

---

## Troubleshooting

### Maven: "command not found"
- **Issue**: Maven not in PATH
- **Fix**: Restart terminal after adding Maven to PATH
- **Verify**: `echo $PATH` should include Maven bin directory

### Docker: "Cannot connect to Docker daemon"
- **Issue**: Docker Desktop not running
- **Fix**: Start Docker Desktop from Start menu, wait for it to say "running"

### Port already in use
```bash
# Check what's using port 5432 (PostgreSQL)
netstat -ano | findstr :5432

# Kill process if needed
taskkill /PID <process-id> /F

# Or use different ports in docker-compose.yml
```

### Out of memory during build
```bash
# Increase Maven memory
set MAVEN_OPTS=-Xmx1024m

# Or add to your environment variables permanently
```

### Docker containers keep restarting
```bash
# Check logs
docker compose logs postgres
docker compose logs redis
docker compose logs rabbitmq

# Common issue: Port conflict
# Solution: Change ports in docker-compose.yml
```

### Testcontainers fails
- **Issue**: Testcontainers can't access Docker
- **Fix**: Ensure Docker Desktop is running
- **Note**: First test run downloads images (takes time)

---

## IDE-Specific Setup

### IntelliJ IDEA

1. **Open Project**: File → Open → Select `SunsetGuard` folder
2. **Wait**: IntelliJ indexes files (5-10 minutes first time)
3. **Enable Lombok**:
   - File → Settings → Plugins → Install "Lombok"
   - File → Settings → Build → Compiler → Annotation Processors → Enable
4. **Run Configuration**:
   - Run → Edit Configurations → Add New → Spring Boot
   - Main class: `io.github.rohanmeshram.sunsetguard.SunsetGuardApplication` (Phase 1+)
   - Module: `sunsetguard-server`

### VS Code

1. **Open Project**: File → Open Folder → Select `SunsetGuard`
2. **Install Extensions** (should prompt automatically):
   - Extension Pack for Java
   - Spring Boot Extension Pack
3. **Lombok Setup**: Should work automatically with Extension Pack for Java

---

## Quick Health Check

Run this after setup to verify everything works:

```bash
# 1. Java
java --version

# 2. Maven
mvn --version

# 3. Docker
docker --version
docker compose ps

# 4. Git
git status

# 5. Test build (Phase 0 - will succeed even with no code)
mvn clean compile
```

**All should pass without errors!**

---

## Resources

- [Maven Getting Started](https://maven.apache.org/guides/getting-started/)
- [Docker Desktop Windows Setup](https://docs.docker.com/desktop/install/windows-install/)
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [IntelliJ IDEA Quickstart](https://www.jetbrains.com/help/idea/getting-started.html)

---

## Next Steps

After completing this setup:

1. ✅ All tools installed and verified
2. ✅ Docker services running
3. ✅ Project builds successfully
4. → **Ready for Phase 1!** See [START_HERE.md](../START_HERE.md)

---

**Having issues?** Open a GitHub issue or check the troubleshooting section above.
