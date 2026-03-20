# 🚀 Nexus & Jenkins Setup Guide
> A step-by-step guide to setting up Jenkins and Nexus, and pushing build artifacts to a Nexus Repository.

---

## 📋 Table of Contents

1. [Launch Ubuntu VM](#step-1-launch-ubuntu-vm)
2. [Connect to the VM](#step-2-connect-to-the-vm)
3. [Install Jenkins](#step-3-install-jenkins)
4. [Install Docker](#step-4-install-docker)
5. [Setup Nexus via Docker](#step-5-setup-nexus-via-docker)
6. [Configure Nexus](#step-6-configure-nexus)
7. [Jenkins Pipeline Script](#final-jenkins-pipeline-script)

---

## Step 1: Launch Ubuntu VM

| Setting | Value |
|---|---|
| **OS** | Ubuntu 24.04 |
| **Instance Type** | `t2.large` |
| **EBS Storage** | 28 GB |

### 🔓 Open the following ports (optional):

```
22, 80, 443, 465, 587, 6443, 8080, 8081
25, 27017, 30000–32767, 30000–100000
```

---

## Step 2: Connect to the VM

Use **MobaXTerm** to connect to the VM via SSH.

---

## Step 3: Install Jenkins

### 3.1 — Update packages and install Java

```bash
sudo apt update
sudo apt install openjdk-17-jre-headless -y
```

### 3.2 — Create and run the Jenkins installation script

```bash
vi jenkins.sh
```

Paste the following into `jenkins.sh`:

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y
```

Save and exit: `Esc` → `:wq`

### 3.3 — Make the script executable and run it

```bash
sudo chmod +x jenkins.sh
./jenkins.sh
```

> 💡 **Note:** Jenkins runs on **port 8080** by default. Open port `8080` on your VM.

### 3.4 — Access Jenkins

```
http://<your-public-ip>:8080
```

Complete the initial setup wizard to configure your Jenkins dashboard.

---

## Step 4: Install Docker

> 📖 Reference: [Docker Official Install Docs](https://docs.docker.com/engine/install/ubuntu/)

### 4.1 — Create and run the Docker installation script

```bash
vi docker.sh
```

Paste the following into `docker.sh`:

```bash
# Add Docker's official GPG key
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the Docker repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

Save and exit: `Esc` → `:wq`

### 4.2 — Make the script executable and run it

```bash
sudo chmod +x docker.sh
./docker.sh
```

### 4.3 — Install Docker packages (latest version)

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 4.4 — Switch to root and verify Docker

```bash
sudo su
docker --version
```

---

## Step 5: Setup Nexus via Docker

> 📖 Reference: [Nexus Docker Image on Docker Hub](https://hub.docker.com/r/sonatype/nexus3/)

### 5.1 — Run the Nexus container

```bash
docker run -d --name nexus3 -p 8081:8081 sonatype/nexus3
```

| Parameter | Description |
|---|---|
| `-d` | Run in detached (background) mode |
| `--name nexus3` | Name the container `nexus3` |
| `-p 8081:8081` | Map Nexus default port `8081` |
| `sonatype/nexus3` | Official Nexus Docker image |

### 5.2 — Verify the container is running

```bash
docker ps
```

> 💡 **Note:** Open port `8081` on your VM.

### 5.3 — Access Nexus

```
http://<your-public-ip>:8081
```

> ⏳ Nexus may take a minute or two to start. If you see *"This site can't be reached"*, wait and refresh.

---

## Step 6: Configure Nexus

### 6.1 — Retrieve the Admin Password

Enter the running container to get the initial password:

```bash
docker exec -it <ContainerID> /bin/bash
```

Once inside the container:

```bash
ls
cd sonatype-work/
cd nexus3/
cat admin.password
```

> ⚠️ Copy the password carefully — **do not copy** the shell prompt text (e.g., `bash-4.4`).

### 6.2 — Log In and Complete Setup

1. Go to `http://<your-public-ip>:8081`
2. Click **Sign In** (top right)
3. Enter credentials:
   - **Username:** `admin`
   - **Password:** *(paste the password copied above)*
4. Set a new password
5. ✅ **Enable Anonymous Access**
6. Click **Next** → **Finish**

### 6.3 — Exit the Container

```bash
exit
```

---

## 📁 Git Repository

The source code used in this demo is available at:

```
https://github.com/kartikhegadi/Nexus-Demo.git
```

---

## Final Jenkins Pipeline Script

Copy and paste the following **Declarative Pipeline** into your Jenkins job configuration:

```groovy
pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kartikhegadi/Nexus-Demo.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Deploy Artifacts') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'settings.xml',
                    jdk: 'jdk17',
                    maven: 'maven3',
                    mavenSettingsConfig: '',
                    traceability: true
                ) {
                    sh 'mvn deploy'
                }
            }
        }

    }
}
```

### ▶️ Running the Pipeline

1. Click **Apply** → **Save**
2. Click **Build Now**
3. Monitor the stages in the **Stage View**

---

## 🗂️ Pipeline Stages Overview

```
Git Checkout  →  Compile  →  Tests  →  Build  →  Deploy Artifacts
```

| Stage | Command | Description |
|---|---|---|
| Git Checkout | `git` | Clone source code from GitHub |
| Compile | `mvn compile` | Compile Java source files |
| Tests | `mvn test` | Run unit tests |
| Build | `mvn package` | Package into a `.jar` / `.war` |
| Deploy Artifacts | `mvn deploy` | Push artifacts to Nexus Repo |

---

> 📝 *Follow the video tutorial for additional visual guidance on configuring Jenkins tools, Maven settings, and Nexus repository connections.*
