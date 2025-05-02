# 📘 DHIS2 with PostgreSQL – Dockerized Setup

This guide explains how to deploy **DHIS2** using **Docker** with a **PostgreSQL (PostGIS)** backend. It uses optimized Docker images for minimal footprint and best performance.

----------

## 📦 Requirements

-   Docker (v20+)
    
-   Docker Compose (v1.27+ or v2+)
    
-   1GB+ Free RAM
    
-   2GB+ Disk Space

To use the official `tomcat:9` base image instead of installing Tomcat manually, here's the updated **`dhis2/Dockerfile`** using `tomcat:9-jdk11` (Java 11 is required for DHIS2):

----------
## Dockerfile
## ✅ `dhis2/Dockerfile` (using `tomcat:9-jdk11`)

```Dockerfile
FROM tomcat:9-jdk11

ENV DHIS2_HOME=/opt/dhis/config

# Create config directory
RUN mkdir -p $DHIS2_HOME

# Copy dhis.conf into the container
COPY dhis.conf $DHIS2_HOME/dhis.conf

# Download and deploy DHIS2 WAR file
RUN apt-get update && apt-get install -y wget && \
    wget -O /usr/local/tomcat/webapps/ROOT.war https://s3-eu-west-1.amazonaws.com/releases.dhis2.org/2.40/dhis.war && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Set JVM memory and DHIS2 config path
ENV JAVA_OPTS="-Xms4000m -Xmx7000m -Ddhis2.home=$DHIS2_HOME"

EXPOSE 8080

CMD ["catalina.sh", "run"]

```

This Dockerfile:

-   Uses the official and stable `tomcat:9-jdk11` image
    
-   Copies your `dhis.conf` to the correct location
    
-   Downloads DHIS2 WAR into the `ROOT.war` for automatic deployment
    



----------

## ✅ Final `docker-compose.yml`

```yaml
version: '3.8'

services:
  postgres:
    image: postgis/postgis:13-3.1
    container_name: dhis2_postgres
    restart: always
    environment:
      POSTGRES_USER: dhis_tl
      POSTGRES_PASSWORD: 123456
      POSTGRES_DB: dhis2
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  dhis2:
    image: rthway/dhis2
    container_name: dhis2_app
    depends_on:
      - postgres
    ports:
      - "8080:8080"
    environment:
      - DHIS2_HOME=/opt/dhis/config
    volumes:
      - dhis2_config:/opt/dhis/config

volumes:
  pgdata:
  dhis2_config:

```

----------

## 🔁 Steps to Run

1.  Ensure you’ve built the DHIS2 image:
    
    ```bash
    docker build -t rthway/dhis2 ./dhis2
    
    ```
    
2.  Run the setup:
    
    ```bash
    docker-compose up -d
    
    ```
    
3.  Access DHIS2 in your browser:
    
    ```
    http://localhost:8080
    
    ```
    

----------
Now, you **can reduce the Docker image size** for your DHIS2 service. Here are practical strategies tailored to your current setup:

----------

### ✅ 1. **Use a Slimmer Base Image**

Instead of `tomcat:9-jdk11`, use `tomcat:9-jdk11-slim`, which is significantly smaller.

**Update your Dockerfile:**

```Dockerfile
FROM tomcat:9-jdk11-slim

```

----------

### ✅ 2. **Avoid Installing Unnecessary Packages**

In your current Dockerfile, you're installing `wget`. Instead of installing it, use `curl` (already present in many base images) or download the `.war` file _beforehand_ and copy it in.

```Dockerfile
# Instead of wget, download locally and copy:
COPY dhis.war /usr/local/tomcat/webapps/ROOT.war

```

----------

### ✅ 3. **Use `.dockerignore` File**

Avoid copying unnecessary files from your local machine into the image.

**Create `.dockerignore`:**

```
*.md
*.log
*.sh
*.zip
node_modules
.git
tmp

```

----------

### ✅ 4. **Combine and Clean Layers**

Reduce layer count and remove caches to keep image small:

```Dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

```

----------

### ✅ 5. **Use Multi-stage Builds** _(Optional but advanced)_

If you need to build or process anything, use a multi-stage build and copy only final artifacts.

----------

### ✅ Example of Optimized Dockerfile

```Dockerfile
FROM tomcat:9-jdk11-slim

ENV DHIS2_HOME=/opt/dhis/config

# Create config directory
RUN mkdir -p $DHIS2_HOME

# Copy minimal config
COPY dhis.conf $DHIS2_HOME/dhis.conf

# Copy WAR file instead of downloading
COPY dhis.war /usr/local/tomcat/webapps/ROOT.war

ENV JAVA_OPTS="-Xms4000m -Xmx7000m -Ddhis2.home=$DHIS2_HOME"

EXPOSE 8080

CMD ["catalina.sh", "run"]

```

----------
let’s break down the **estimated size savings** when optimizing your DHIS2 Docker image.

----------

## 🚀 Starting Point: Approximate Sizes

Image Layer

Approx. Size

`tomcat:9-jdk11`

~630 MB

DHIS2 WAR file (`dhis.war`)

~130 MB

Installed tools (e.g., wget)

~30–50 MB

Debian apt cache (left over)

~20 MB

Temporary build layers

~50 MB

🧱 **Typical total size**: **~850 MB to 900 MB**

----------

## ✅ After Optimization

Optimization Step

Saved Size (Approx.)

Use `tomcat:9-jdk11-slim` instead of full

-200 MB

Avoid `apt install` (e.g., wget, etc.)

-30 MB

Delete apt cache / temp files

-20 MB

Use `.dockerignore` to reduce context size

-10 MB

Use `COPY` instead of downloading in container

-5 MB (indirect)

🎯 **Optimized total size**: **~600–650 MB**

----------
