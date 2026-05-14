#  Spring PetClinic – DevOps CI/CD Pipeline with Maven & SonarQube

> **Author:** Kishore · **Date:** May 2026 · **Version:** 4.0.0-SNAPSHOT

A complete end-to-end DevOps setup for the [Spring PetClinic](https://github.com/spring-projects/spring-petclinic) application — built with Maven, analysed with SonarQube, and deployed on AWS EC2 using Spring Boot's embedded server.

---

## 📌 Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [AWS EC2 Infrastructure](#aws-ec2-infrastructure)
3. [Tech Stack](#tech-stack)
4. [Setup & Deployment](#setup--deployment)
   - [Step 1 – Update the System](#step-1--update-the-system)
   - [Step 2 – Install Java](#step-2--install-java)
   - [Step 3 – Install Maven](#step-3--install-maven)
   - [Step 4 – Clone the Repository](#step-4--clone-the-repository)
   - [Step 5 – Build the Application](#step-5--build-the-application)
   - [Step 6 – SonarQube Analysis](#step-6--sonarqube-analysis)
   - [Step 7 – Install Tomcat 10](#step-7--install-tomcat-10)
   - [Step 8 – Stop Tomcat (Free Port 8080)](#step-8--stop-tomcat-free-port-8080)
   - [Step 9 – Run the Application](#step-9--run-the-application)
   - [Step 10 – Access the Application](#step-10--access-the-application)
5. [Useful Commands](#useful-commands)
6. [Key Concepts](#key-concepts)

---

## 🏗️ Architecture Overview

```
Developer
    │
    ▼
GitHub Repository
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│                   AWS EC2 – ap-south-1 (Mumbai)                 │
│                                                                 │
│  ┌───────────────┐   ┌──────────────────┐   ┌───────────────┐  │
│  │     Nexus     │   │    SonarQube     │   │  Maven Build  │  │
│  │   Artifact    │   │  Code Quality    │   │  + Spring     │  │
│  │  Repository   │   │   Port: 9000     │   │  Boot Runner  │  │
│  │  Port: 8081   │   │                  │   │  Port: 8080   │  │
│  └───────────────┘   └──────────────────┘   └───────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## ☁️ AWS EC2 Infrastructure

> **3 EC2 instances** running in **Asia Pacific (Mumbai) – ap-south-1**

| Instance Name   | Public IP        | Purpose                        | Port | Instance Type    |
|-----------------|------------------|--------------------------------|------|------------------|
| `nexus`         | —                | Artifact Repository (Nexus)    | 8081 | c7i.flex.large   |
| `sonarqube`     | 3.7.55.140       | Code Quality Analysis          | 9000 | c7i.flex.large   |
| `maven&tomcat`  | 43.204.114.151   | Build & Deploy                 | 8080 | c7i.flex.large   |
-----

<img width="3839" height="2159" alt="Screenshot 2026-05-14 113103" src="https://github.com/user-attachments/assets/9bb34823-0a60-4de3-a7f9-1d4596efc185" />

-----
**OS:** Ubuntu 26.04 LTS (GNU/Linux 7.0.0-1004-aws x86_64)

---

## 🛠️ Tech Stack

| Tool              | Version              | Purpose              |
|-------------------|----------------------|----------------------|
| Ubuntu            | 26.04 LTS            | Operating System     |
| Java (OpenJDK)    | 21                   | Runtime              |
| Apache Maven      | 3.x                  | Build Tool           |
| Spring PetClinic  | 4.0.0-SNAPSHOT       | Application          |
| Spring Boot       | Embedded             | App Framework        |
| Apache Tomcat     | 10.1.40              | Web Server           |
| SonarQube         | 26.5.0 (Community)   | Code Quality         |
| AWS EC2           | c7i.flex.large       | Cloud Infrastructure |

----

## CONFIGURATION OF INSTANCE:

----
<img width="3839" height="2032" alt="Screenshot 2026-05-14 113130" src="https://github.com/user-attachments/assets/062e7116-9fa9-4450-afe8-5ec9e61fafe3" />

----

<img width="3839" height="2026" alt="Screenshot 2026-05-14 113143" src="https://github.com/user-attachments/assets/b85a0651-9552-48a5-856b-fb643523fd57" />

----
-----
## 🚀 Setup & Deployment

### Step 1 – Update the System

Always start by updating the package list to get the latest software:

```bash
sudo apt update && sudo apt upgrade -y
```

---

### Step 2 – Install Java

Tomcat 10 and Maven both require Java. Install OpenJDK:

```bash
sudo apt install default-jdk -y
```

Verify the installation:

```bash
java -version
# Expected: openjdk version "21.x.x"
```

---

### Step 3 – Install Maven

Maven is used to build the Spring PetClinic project:

```bash
sudo apt install maven -y
```

Verify the installation:

```bash
mvn -version
```

---

### Step 4 – Clone the Repository

```bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
```

Project structure after cloning:

```
spring-petclinic/
├── src/                   # Java source code
├── target/                # Build output (generated after build)
├── pom.xml                # Maven project config
├── build.gradle           # Gradle config
├── docker-compose.yml     # Docker setup
├── k8s/                   # Kubernetes manifests
├── mvnw                   # Maven wrapper script
└── README.md
```

---

### Step 5 – Build the Application

Run the Maven build to compile the code and package it as a JAR:

```bash
mvn clean package -DskipTests
```

| Flag            | Meaning                                    |
|-----------------|--------------------------------------------|
| `clean`         | Delete previous build output               |
| `package`       | Compile + test + create JAR file           |
| `-DskipTests`   | Skip unit tests to speed up the build      |

After a successful build, the JAR is created at:

```
target/spring-petclinic-4.0.0-SNAPSHOT.jar
```
<img width="3839" height="2023" alt="Screenshot 2026-05-14 113223" src="https://github.com/user-attachments/assets/22921091-8afb-4248-8d30-24f796ae81d8" />

Confirm the output:

```bash
cd spring-petclinic/target/
ls
# spring-petclinic-4.0.0-SNAPSHOT.jar
# spring-petclinic-4.0.0-SNAPSHOT.jar.original
# classes/
# ...
```

---

### Step 6 – SonarQube Analysis

SonarQube performs static code analysis to detect bugs, vulnerabilities, and code smells.

#### 6a – Start SonarQube (on the SonarQube EC2 instance)

```bash
cd sonarqube-26.5.0.122743/bin/linux-x86-64/
./sonar.sh start
```

#### 6b – Run the analysis (from the maven&tomcat instance)

```bash
mvn sonar:sonar \
  -Dsonar.projectKey=spring_petclinic \
  -Dsonar.host.url=http://3.7.55.140:9000 \
  -Dsonar.login=<your-sonar-token>
```

#### 6c – View the analysis report
<img width="3839" height="2036" alt="Screenshot 2026-05-14 113200" src="https://github.com/user-attachments/assets/b4a9fd7b-d532-4025-b371-cdc1a9adb0e0" />

```bash
cat target/sonar/report-task.txt
```

Expected output:

```
projectKey=spring_petclinic
serverUrl=http://3.7.55.140:9000
dashboardUrl=http://3.7.55.140:9000/dashboard?id=spring_petclinic
```

#### SonarQube Dashboard Results
<img width="3839" height="2037" alt="Screenshot 2026-05-14 113235" src="https://github.com/user-attachments/assets/bf38ad06-06cf-4d9c-9ffe-7fdc1735ae2c" />

| Metric           | Result               |
|------------------|----------------------|
| Project          | spring_petclinic     |
| Lines of Code    | 1.2k                 |
| Version          | 4.0.0-SNAPSHOT       |
| Quality Gate     | ✅ **Passed**        |

---

### Step 7 – Install Tomcat 10

On the **maven&tomcat** EC2 instance, install Tomcat 10 via APT:

```bash
sudo apt install tomcat10 -y
```

APT automatically installs all dependencies (`libapr1t64`, `libeclipse-jdt-core-java`, `libtomcat10-java`, `tomcat10-common`, `libtcnative-1`) and:

- Creates the `tomcat` system user (UID 982)
- Creates all config files under `/etc/tomcat10/`
- Enables and starts the `tomcat10` systemd service on **port 8080**

Verify Tomcat is running:

```bash
systemctl status tomcat10.service
```

#### Tomcat 10 Directory Reference (APT Install)

| Purpose      | Path                        |
|--------------|-----------------------------|
| Home         | `/usr/share/tomcat10`       |
| Webapps      | `/var/lib/tomcat10/webapps` |
| Config files | `/etc/tomcat10/`            |
| Logs         | `/var/log/tomcat10/`        |

---

### Step 8 – Stop Tomcat (Free Port 8080)

> ⚠️ **Important!** Tomcat 10 starts automatically on port **8080**.
> Spring Boot also uses port **8080** by default.
> Both **cannot run simultaneously on the same port** — stop Tomcat first.

```bash
sudo systemctl stop tomcat10
```

Verify port 8080 is free:

```bash
sudo ss -tlnp | grep 8080
# Empty output = port is free ✅
```

---

### Step 9 – Run the Application

Navigate to the target directory:

```bash
cd ~/spring-petclinic/target/
```
<img width="3838" height="2028" alt="Screenshot 2026-05-14 113247" src="https://github.com/user-attachments/assets/193663cf-7736-49ff-b64e-98b096126a0e" />

#### Option A – Foreground (testing/debugging)

```bash
java -jar spring-petclinic-4.0.0-SNAPSHOT.jar
```

Press `Ctrl+C` to stop.

#### Option B – Background with nohup (recommended)

```bash
nohup java -jar spring-petclinic-4.0.0-SNAPSHOT.jar > /tmp/petclinic.log 2>&1 &
```

| Part                                    | Meaning                                           |
|-----------------------------------------|---------------------------------------------------|
| `nohup`                                 | Keep the process alive when the terminal closes   |
| `java -jar`                             | Run the JAR using the Java runtime                |
| `spring-petclinic-4.0.0-SNAPSHOT.jar`  | The built application                             |
| `> /tmp/petclinic.log`                  | Redirect stdout to a log file                     |
| `2>&1`                                  | Redirect stderr to the same log file              |
| `&`                                     | Run in the background                             |

Monitor the logs:

```bash
tail -f /tmp/petclinic.log
```

---

### Step 10 – Access the Application

Open your browser and navigate to:

```
http://43.204.114.151:8080
```

You should see the **Spring PetClinic Welcome Page** with Home, Find Owners, Veterinarians, and Error menus. ✅

---

## ⚙️ Useful Commands

```bash
# Monitor application logs
tail -f /tmp/petclinic.log

# Find the running Java process
ps aux | grep java

# Kill the application by PID
kill <PID>

# Check what is using port 8080
sudo ss -tlnp | grep 8080

# Tomcat service management
sudo systemctl start   tomcat10
sudo systemctl stop    tomcat10
sudo systemctl restart tomcat10
sudo systemctl status  tomcat10

# Prevent Tomcat from auto-starting on boot
sudo systemctl disable tomcat10
```

---

## 💡 Key Concepts

### JAR vs WAR — When to use Tomcat

| Type                   | External Tomcat Needed?                    | How to Run                              |
|------------------------|--------------------------------------------|-----------------------------------------|
| `.jar` (Spring Boot)   | ❌ No – has **embedded Tomcat** inside     | `java -jar app.jar`                     |
| `.war` (Traditional)   | ✅ Yes – needs an external Tomcat server   | Copy to `/var/lib/tomcat10/webapps/`    |

> This project is packaged as a **JAR** with an embedded Tomcat server.
> The externally installed Tomcat 10 is **not required** to run the application — it only needs to be stopped to free up port 8080.

---

### Why the Port 8080 Conflict Happens

```
Port 8080
    │
    ├── tomcat10 installed → starts automatically on 8080 ✅
    │
    └── Spring Boot JAR tries to start → port already taken ❌
              │
              └── Fix: sudo systemctl stop tomcat10
                        java -jar spring-petclinic-4.0.0-SNAPSHOT.jar
```

---

### What SNAPSHOT Means

```
spring-petclinic-4.0.0-SNAPSHOT.jar
                  └─────┬──────┘
                      4.0.0 = version number
                    SNAPSHOT = in-progress / development build
                               (not a final stable release)
```

---

## 👤 Author

**Kishore** — AWS DevOps Engineer  
AWS · Maven · SonarQube · Tomcat · Spring Boot

---

*Deployed on AWS EC2 – Asia Pacific (Mumbai) | May 2026*
