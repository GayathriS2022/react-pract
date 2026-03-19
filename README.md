# react-pract
<img width="1227" height="816" alt="Screenshot 2026-03-19 at 3 40 19 PM" src="https://github.com/user-attachments/assets/404f43b4-21f0-452a-b322-a5a5a53e2f42" />
# Server Control Dashboard — Full-Stack Spring Boot + React

A production-ready monorepo with a **Spring Boot** backend (Java 21, Maven) and a **React** frontend (Vite, TypeScript). The React app is built and bundled into the Spring Boot JAR so the final artifact is a **single executable JAR**.

**Rendering:** CSR (Client-Side Rendering) — Spring Boot serves a static `index.html` shell; React handles all rendering in the browser.

---

## Table of Contents

- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Creating This Project From Scratch](#creating-this-project-from-scratch)
- [Dependencies](#dependencies)
- [Development (hot-reload)](#development-hot-reload)
- [Production Build (single JAR)](#production-build-single-jar)
- [Running on a Custom Port](#running-on-a-custom-port)
- [API Endpoints](#api-endpoints)
- [Theme System](#theme-system)
- [Reusable Components](#reusable-components)
- [Integrating the Frontend Into Another Spring Boot (Maven) Project](#integrating-the-frontend-into-another-spring-boot-maven-project)
- [Environment](#environment)

---

## Project Structure

```
react-pract/
├── pom.xml                                # Maven build with frontend-maven-plugin
├── mvnw / mvnw.cmd                        # Maven wrapper (no global install needed)
├── .mvn/wrapper/
│   ├── maven-wrapper.jar
│   └── maven-wrapper.properties
├── src/
│   └── main/
│       ├── java/com/reactpract/
│       │   ├── Application.java                 # Spring Boot entry point
│       │   ├── config/WebConfig.java            # SPA fallback for React Router
│       │   └── controller/
│       │       ├── ApiController.java           # /api/hello, /api/health
│       │       └── ServerController.java        # /api/servers (mock CRUD)
│       └── resources/
│           ├── application.yml
│           └── static/                          # ← auto-generated from frontend build
├── frontend/
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── vite-env.d.ts
│   ├── index.html
│   └── src/
│       ├── main.tsx                             # React mount point
│       ├── App.tsx                              # Router + providers
│       ├── theme/                               # Centralized design tokens
│       │   ├── theme.ts
│       │   ├── ThemeContext.tsx
│       │   └── index.ts
│       ├── components/                          # Reusable UI components
│       │   ├── Button/Button.tsx
│       │   ├── Text/Text.tsx
│       │   ├── Tabs/Tabs.tsx
│       │   ├── Spinner/Spinner.tsx
│       │   ├── ProgressBar/ProgressBar.tsx
│       │   ├── Toast/ToastContext.tsx + ToastContainer.tsx
│       │   ├── StatusIcon/StatusIcon.tsx
│       │   ├── ServerItem/ServerItem.tsx
│       │   ├── ServerCard/ServerCard.tsx
│       │   └── index.ts
│       ├── hooks/
│       │   ├── useApi.ts
│       │   └── useServers.ts
│       ├── services/
│       │   ├── api.ts                           # Axios instance
│       │   └── serverService.ts                 # Server API calls + retry logic
│       ├── types/
│       │   └── server.ts
│       ├── pages/
│       │   ├── ServersPage.tsx                  # Modern card-grid server view
│       │   ├── ServersClassicPage.tsx           # Classic list server view
│       │   ├── LogsPage.tsx
│       │   └── SettingsPage.tsx
│       ├── layouts/
│       │   └── AppLayout.tsx
│       └── styles/
│           └── global.css
└── README.md
```

---

## Prerequisites

| Tool       | Version  | Install                                      |
|------------|----------|----------------------------------------------|
| Java       | 21+      | `sdk install java 21-open` or Homebrew       |
| Node.js    | 20+      | `nvm install 20` or https://nodejs.org       |
| npm        | 10+      | Comes with Node.js                           |
| Maven      | 3.9+     | **Not needed** — wrapper (`./mvnw`) included |

---

## Creating This Project From Scratch

### Step 1: Create the root project directory

```bash
mkdir react-pract && cd react-pract
```

### Step 2: Initialize Maven wrapper

```bash
# Download the Maven wrapper (no global Maven install required)
curl -sL "https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper-distribution/3.3.2/maven-wrapper-distribution-3.3.2-bin.zip" -o /tmp/mvn-wrapper.zip
unzip -o /tmp/mvn-wrapper.zip -d /tmp/mvn-wrapper
cp /tmp/mvn-wrapper/mvnw .
cp /tmp/mvn-wrapper/mvnw.cmd .
mkdir -p .mvn/wrapper
cp /tmp/mvn-wrapper/.mvn/wrapper/maven-wrapper.jar .mvn/wrapper/
chmod +x mvnw
```

Create `.mvn/wrapper/maven-wrapper.properties`:

```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.9/apache-maven-3.9.9-bin.zip
wrapperUrl=https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper/3.3.2/maven-wrapper-3.3.2.jar
```

### Step 3: Create `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.3</version>
        <relativePath/>
    </parent>

    <groupId>com.reactpract</groupId>
    <artifactId>react-pract</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
        <frontend-maven-plugin.version>1.15.1</frontend-maven-plugin.version>
        <node.version>v20.11.0</node.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <!-- Frontend: install Node, npm ci, npm run build -->
            <plugin>
                <groupId>com.github.eirslett</groupId>
                <artifactId>frontend-maven-plugin</artifactId>
                <version>${frontend-maven-plugin.version}</version>
                <configuration>
                    <workingDirectory>frontend</workingDirectory>
                </configuration>
                <executions>
                    <execution>
                        <id>install-node-and-npm</id>
                        <goals><goal>install-node-and-npm</goal></goals>
                        <configuration>
                            <nodeVersion>${node.version}</nodeVersion>
                        </configuration>
                    </execution>
                    <execution>
                        <id>npm-install</id>
                        <goals><goal>npm</goal></goals>
                        <configuration>
                            <arguments>ci</arguments>
                        </configuration>
                    </execution>
                    <execution>
                        <id>npm-build</id>
                        <goals><goal>npm</goal></goals>
                        <configuration>
                            <arguments>run build</arguments>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- Copy frontend/dist → target/classes/static -->
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-frontend-to-static</id>
                        <phase>generate-resources</phase>
                        <goals><goal>copy-resources</goal></goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/classes/static</outputDirectory>
                            <resources>
                                <resource>
                                    <directory>frontend/dist</directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- Clean generated frontend files -->
            <plugin>
                <artifactId>maven-clean-plugin</artifactId>
                <configuration>
                    <filesets>
                        <fileset><directory>frontend/dist</directory></fileset>
                        <fileset><directory>frontend/node</directory></fileset>
                        <fileset><directory>src/main/resources/static</directory></fileset>
                    </filesets>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Step 4: Create Spring Boot backend structure

```bash
mkdir -p src/main/java/com/reactpract/config
mkdir -p src/main/java/com/reactpract/controller
mkdir -p src/main/resources
```

Then create `Application.java`, `WebConfig.java`, `ApiController.java`, `ServerController.java`, and `application.yml` (see source files).

### Step 5: Create React frontend with Vite

```bash
npm create vite@latest frontend -- --template react-ts
cd frontend
```

### Step 6: Install frontend dependencies

```bash
npm install react react-dom react-router-dom axios
npm install -D typescript @types/react @types/react-dom @vitejs/plugin-react vite
```

### Step 7: Configure path aliases

In `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

In `vite.config.ts`:

```typescript
import path from "path";

export default defineConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src"),
    },
  },
  server: {
    port: 3000,
    proxy: {
      "/api": {
        target: "http://localhost:8080",
        changeOrigin: true,
      },
    },
  },
});
```

### Step 8: Build and verify

```bash
cd ..   # back to project root
./mvnw clean package
java -jar target/react-pract-1.0.0.jar
# Open http://localhost:8080
```

---

## Dependencies

### Backend (Spring Boot / Maven)

| Dependency                          | Version | Purpose                          |
|-------------------------------------|---------|----------------------------------|
| `spring-boot-starter-parent`        | 3.4.3   | Spring Boot parent POM           |
| `spring-boot-starter-web`           | 3.4.3   | Embedded Tomcat + REST support   |
| `spring-boot-starter-test`          | 3.4.3   | Testing (JUnit 5)                |
| `frontend-maven-plugin`             | 1.15.1  | Builds frontend during Maven lifecycle |
| Java                                | 21      | Language level                   |
| Maven                               | 3.9.9   | Build tool (wrapper included)    |

### Frontend (React / npm)

**Runtime dependencies:**

| Package            | Version  | Purpose                              |
|--------------------|----------|--------------------------------------|
| `react`            | ^19.0.0  | UI library                           |
| `react-dom`        | ^19.0.0  | DOM renderer                         |
| `react-router-dom` | ^7.2.0   | Client-side routing                  |
| `axios`            | ^1.7.9   | HTTP client for API calls            |

**Dev dependencies:**

| Package                | Version  | Purpose                          |
|------------------------|----------|----------------------------------|
| `typescript`           | ~5.7.3   | Type checking                    |
| `vite`                 | ^6.1.0   | Build tool + dev server          |
| `@vitejs/plugin-react` | ^4.3.4  | React Fast Refresh for Vite      |
| `@types/react`         | ^19.0.8  | React type definitions           |
| `@types/react-dom`     | ^19.0.3  | ReactDOM type definitions        |

**No other third-party UI or state libraries are used.** All components are built from scratch using the centralized theme system.

---

## Development (hot-reload)

Run the backend and frontend dev servers separately for the best DX:

```bash
# Terminal 1 — Spring Boot (port 8080)
./mvnw spring-boot:run

# Terminal 2 — Vite dev server (port 3000, proxies /api → 8080)
cd frontend && npm install && npm run dev
```

Open **http://localhost:3000** for the React app with hot-reload.
API calls are automatically proxied to the Spring Boot server.

---

## Production Build (single JAR)

```bash
# Build everything (installs Node, npm deps, builds React, bundles into JAR)
./mvnw clean package

# Run the fat JAR
java -jar target/react-pract-1.0.0.jar
```

Open **http://localhost:8080** — Spring Boot serves both the API and the React SPA.

### What happens during `./mvnw clean package`:

1. `frontend-maven-plugin` downloads Node.js v20.11.0 into `frontend/node/` (first time only)
2. Runs `npm ci` in `frontend/` to install dependencies
3. Runs `npm run build` → outputs to `frontend/dist/`
4. `maven-resources-plugin` copies `frontend/dist/` into `target/classes/static/`
5. `spring-boot-maven-plugin` creates the executable fat JAR

---

## Running on a Custom Port

The default port is `8080`. You can override it in multiple ways:

### Option 1: Command-line argument

```bash
java -jar target/react-pract-1.0.0.jar --server.port=9090
```

### Option 2: Environment variable (`SERVER_PORT`)

```bash
SERVER_PORT=9090 java -jar target/react-pract-1.0.0.jar
```

### Option 3: Environment variable (`PORT`)

```bash
PORT=9090 java -jar target/react-pract-1.0.0.jar
```

### Option 4: Edit `application.yml`

```yaml
server:
  port: 9090
```

### Option 5: JVM system property

```bash
java -Dserver.port=9090 -jar target/react-pract-1.0.0.jar
```

### Priority order (highest wins):

1. `--server.port=XXXX` (command-line argument)
2. `-Dserver.port=XXXX` (JVM system property)
3. `SERVER_PORT` environment variable
4. `PORT` environment variable
5. Value in `application.yml` (default `8080`)

### Examples

```bash
# Run on port 3000
java -jar target/react-pract-1.0.0.jar --server.port=3000

# Run on port 443 (HTTPS default, needs root/sudo on Linux)
sudo java -jar target/react-pract-1.0.0.jar --server.port=443

# Run on port 0 (OS picks a random available port)
java -jar target/react-pract-1.0.0.jar --server.port=0
```

### Development mode on custom port

```bash
# Start backend on port 9090
./mvnw spring-boot:run -Dspring-boot.run.arguments="--server.port=9090"

# Update vite.config.ts proxy target to match, or:
cd frontend
VITE_API_PORT=9090 npm run dev
```

> **Note:** When changing the backend port during development, update the proxy target in `frontend/vite.config.ts` accordingly.

---

## API Endpoints

| Method | Path                        | Description                        |
|--------|-----------------------------|------------------------------------|
| GET    | `/api/hello`                | Sample greeting with timestamp     |
| GET    | `/api/health`               | Health check                       |
| GET    | `/api/servers`              | List all servers (mock data)       |
| POST   | `/api/servers/{id}/start`   | Start a server (simulated delay)   |
| POST   | `/api/servers/{id}/stop`    | Stop a server (simulated delay)    |
| POST   | `/api/servers/start-all`    | Start all servers                  |
| POST   | `/api/servers/stop-all`     | Stop all servers                   |

---

## Theme System

All UI components consume values from `frontend/src/theme/theme.ts`. To change the look and feel globally, edit the colors, typography, spacing, radii, shadows, or transitions in that file.

```
theme.ts
├── colors        → primary, secondary, success, warning, danger, neutrals, surfaces
├── typography    → fontFamily, sizes (sm–4xl), weights, line heights
├── spacing       → xs (0.25rem) through 3xl (4rem)
├── radii         → sm, md, lg, xl, full
├── shadows       → sm, md, lg
└── transitions   → fast (150ms), normal (250ms), slow (350ms)
```

---

## Reusable Components

| Component      | Props                                                        |
|----------------|--------------------------------------------------------------|
| `Button`       | variant, size, color, fullWidth, loading, disabled, icons    |
| `Text`         | variant, color, align, weight, as, truncate                  |
| `Tabs`         | tabs, variant, size, activeKey, onChange                      |
| `Spinner`      | size, color, thickness, label                                |
| `ProgressBar`  | value, max, size, color, showLabel, striped, animated        |
| `Toast`        | type (success/error/info), message, auto-dismiss             |
| `StatusIcon`   | status (running/stopped/unknown), size                       |
| `ServerItem`   | server, loading, onStart, onStop (list layout)               |
| `ServerCard`   | server, loading, onStart, onStop (card grid layout)          |

---

## Integrating the Frontend Into Another Spring Boot (Maven) Project

Follow these steps to take the React frontend from this project and embed it into a different Spring Boot Maven project.

### Step 1: Copy the `frontend/` folder

Copy the entire `frontend/` directory into the root of your target Maven project:

```bash
cp -r frontend/ /path/to/your-maven-project/frontend/
```

Your project should now look like:

```
your-maven-project/
├── pom.xml
├── frontend/           ← copied from this project
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── src/
├── src/
│   └── main/
│       ├── java/...
│       └── resources/
│           └── application.yml
```

### Step 2: Add `frontend-maven-plugin` to your `pom.xml`

Add the following plugins inside your `<build><plugins>` section:

```xml
<!-- Frontend: install Node, npm ci, npm run build -->
<plugin>
    <groupId>com.github.eirslett</groupId>
    <artifactId>frontend-maven-plugin</artifactId>
    <version>1.15.1</version>
    <configuration>
        <workingDirectory>frontend</workingDirectory>
    </configuration>
    <executions>
        <execution>
            <id>install-node-and-npm</id>
            <goals><goal>install-node-and-npm</goal></goals>
            <configuration>
                <nodeVersion>v20.11.0</nodeVersion>
            </configuration>
        </execution>
        <execution>
            <id>npm-install</id>
            <goals><goal>npm</goal></goals>
            <configuration>
                <arguments>ci</arguments>
            </configuration>
        </execution>
        <execution>
            <id>npm-build</id>
            <goals><goal>npm</goal></goals>
            <configuration>
                <arguments>run build</arguments>
            </configuration>
        </execution>
    </executions>
</plugin>

<!-- Copy frontend/dist → target/classes/static -->
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <executions>
        <execution>
            <id>copy-frontend-to-static</id>
            <phase>generate-resources</phase>
            <goals><goal>copy-resources</goal></goals>
            <configuration>
                <outputDirectory>${project.build.directory}/classes/static</outputDirectory>
                <resources>
                    <resource>
                        <directory>frontend/dist</directory>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>

<!-- Clean generated frontend files -->
<plugin>
    <artifactId>maven-clean-plugin</artifactId>
    <configuration>
        <filesets>
            <fileset><directory>frontend/dist</directory></fileset>
            <fileset><directory>frontend/node</directory></fileset>
            <fileset><directory>src/main/resources/static</directory></fileset>
        </filesets>
    </configuration>
</plugin>
```

### Step 3: Add the SPA fallback controller

React Router uses client-side routes like `/servers`, `/servers-classic`, etc. Without a fallback, Spring Boot returns 404 for those routes. Add this class to your project:

```java
package com.yourpackage.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.resource.PathResourceResolver;

import java.io.IOException;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/static/")
                .resourceChain(true)
                .addResolver(new PathResourceResolver() {
                    @Override
                    protected Resource getResource(String resourcePath, Resource location)
                            throws IOException {
                        Resource requested = location.createRelative(resourcePath);
                        return requested.exists() && requested.isReadable()
                                ? requested
                                : new ClassPathResource("/static/index.html");
                    }
                });
    }
}
```

### Step 4: Copy the backend controllers (optional)

If you want the server management API, copy the controller:

```bash
cp src/main/java/com/reactpract/controller/ServerController.java \
   /path/to/your-project/src/main/java/com/yourpackage/controller/
```

Update the `package` declaration in the copied file.

### Step 5: Add `.gitignore` entries

```gitignore
frontend/node_modules/
frontend/dist/
frontend/node/
src/main/resources/static/
```

### Step 6: Build and run

```bash
./mvnw clean package
java -jar target/your-project-1.0.0.jar

# Or on a custom port:
java -jar target/your-project-1.0.0.jar --server.port=9090
```

### Step 7: Update Vite proxy (if your backend port differs)

If your backend runs on a different port, edit `frontend/vite.config.ts`:

```typescript
proxy: {
  "/api": {
    target: "http://localhost:YOUR_PORT",
    changeOrigin: true,
  },
},
```

### How it works inside the JAR

```
your-project.jar
└── BOOT-INF/
    └── classes/
        └── static/              ← React build lives here
            ├── index.html
            └── assets/
                ├── index-XXXXX.css
                └── index-XXXXX.js
```

When the JAR runs:
1. Browser requests any route → Spring Boot checks `static/` for a matching file
2. If found (CSS, JS, images) → serves the file directly
3. If not found (e.g. `/servers`, `/servers-classic`) → falls back to `index.html`
4. `index.html` loads the JS bundle → React Router handles client-side routing
5. React makes API calls to `/api/*` → Spring Boot handles those as REST endpoints

### Verify it works

```bash
# Check the JAR contains the static files
jar tf target/your-project-1.0.0.jar | grep static

# Should output:
# BOOT-INF/classes/static/
# BOOT-INF/classes/static/index.html
# BOOT-INF/classes/static/assets/index-XXXXX.css
# BOOT-INF/classes/static/assets/index-XXXXX.js
```

### Quick summary

| What to copy / add                        | Where                                            |
|-------------------------------------------|--------------------------------------------------|
| `frontend/` directory                     | Root of your Maven project                       |
| `frontend-maven-plugin` + resource plugin | `pom.xml` `<plugins>` section                    |
| `WebConfig.java`                          | Your config package (for SPA fallback)           |
| `ServerController.java` (optional)        | Your controller package                          |
| `.gitignore` entries                       | Project `.gitignore`                             |

---

## Environment

The server port defaults to `8080` and can be overridden using any of the methods described in [Running on a Custom Port](#running-on-a-custom-port).

```bash
# Quick start on default port
./mvnw clean package && java -jar target/react-pract-1.0.0.jar

# Quick start on custom port
./mvnw clean package && java -jar target/react-pract-1.0.0.jar --server.port=9090

# Using environment variable
SERVER_PORT=3000 java -jar target/react-pract-1.0.0.jar
```

### Domain access examples

If your domain is `servers.sample.com`:

| Port  | URL                                    | Notes                                       |
|-------|----------------------------------------|---------------------------------------------|
| 80    | `http://servers.sample.com`            | Default HTTP port (no port needed in URL)   |
| 443   | `https://servers.sample.com`           | Default HTTPS port (needs SSL config)       |
| 8080  | `http://servers.sample.com:8080`       | Default Spring Boot port                    |
| 9090  | `http://servers.sample.com:9090`       | Custom port                                 |
| Any   | `http://servers.sample.com:<PORT>`     | Replace `<PORT>` with your chosen port      |

