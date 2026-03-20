# Nexus & Jenkins — Artifact Management Demo

![CI/CD](https://img.shields.io/badge/CI%2FCD-Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Nexus](https://img.shields.io/badge/Artifact_Repo-Nexus-1B6CA8?style=for-the-badge&logo=sonatype&logoColor=white)
![Docker](https://img.shields.io/badge/Container-Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Maven](https://img.shields.io/badge/Build-Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)
![Java](https://img.shields.io/badge/Java-17-007396?style=for-the-badge&logo=java&logoColor=white)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu_24.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

---

## 📖 About This Project

This project demonstrates a complete **CI/CD artifact management pipeline** using **Jenkins** and **Sonatype Nexus Repository Manager**. It covers provisioning a cloud VM, installing Jenkins and Docker, spinning up Nexus as a Docker container, and building a Jenkins pipeline that compiles, tests, packages, and deploys Java artifacts to Nexus.

---

## 🏗️ Architecture Overview

```
┌─────────────┐     Push Code      ┌──────────────────────────────────────────┐
│   GitHub    │ ─────────────────► │              Jenkins Pipeline            │
│  Repository │                    │                                          │
└─────────────┘                    │  Git Checkout → Compile → Test → Build  │
                                   │                    │                     │
                                   └────────────────────┼─────────────────────┘
                                                        │ mvn deploy
                                                        ▼
                                             ┌─────────────────────┐
                                             │   Sonatype Nexus    │
                                             │  Repository Manager │
                                             │   (Docker :8081)    │
                                             └─────────────────────┘
```

---

## ✨ Features

- ⚙️ Automated Jenkins pipeline with 5 stages
- 📦 Artifact publishing to Nexus Repository via Maven
- 🐳 Nexus running as a Docker container
- ☕ Java 17 + Maven build toolchain
- 🔁 Fully repeatable infrastructure setup

---

## 🛠️ Tech Stack

| Tool | Purpose | Default Port |
|---|---|---|
| **Jenkins** | CI/CD Automation Server | `8080` |
| **Sonatype Nexus 3** | Artifact Repository Manager | `8081` |
| **Docker** | Container runtime for Nexus | — |
| **Maven 3** | Build & dependency management | — |
| **Java 17** | Application runtime & JDK | — |
| **Ubuntu 24.04** | Host operating system | — |

---

## 📋 Prerequisites

Before getting started, ensure you have the following:

- ✅ An AWS (or any cloud) account to provision a VM
- ✅ **MobaXTerm** or any SSH client installed locally
- ✅ Basic familiarity with Linux terminal commands
- ✅ Port access to open `8080` and `8081` on the VM

---

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/kartikhegadi/Nexus-Demo.git
cd Nexus-Demo
```

### 2. Provision the VM

Launch an **Ubuntu 24.04** VM with the following specs:

| Setting | Value |
|---|---|
| Instance Type | `t2.large` |
| Storage | `28 GB` EBS |
| Open Ports | `22, 80, 443, 8080, 8081, 6443, 30000–32767` |

### 3. Install Jenkins

```bash
sudo apt update
sudo apt install openjdk-17-jre-headless -y

# Run Jenkins install script (see setup guide for full script)
sudo chmod +x jenkins.sh && ./jenkins.sh
```

Access Jenkins at: **`http://<your-public-ip>:8080`**

### 4. Install Docker & Run Nexus

```bash
# Run Docker install script (see setup guide for full script)
sudo chmod +x docker.sh && ./docker.sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start Nexus container
sudo docker run -d --name nexus3 -p 8081:8081 sonatype/nexus3
```

Access Nexus at: **`http://<your-public-ip>:8081`**

### 5. Retrieve Nexus Admin Password

```bash
docker exec -it <ContainerID> /bin/bash
cat sonatype-work/nexus3/admin.password
```

---

## 🔧 Jenkins Pipeline

The pipeline is defined in the `Jenkinsfile` and consists of **5 stages**:

```
┌──────────────┐   ┌──────────┐   ┌───────┐   ┌───────┐   ┌──────────────────┐
│ Git Checkout │──►│ Compile  │──►│ Tests │──►│ Build │──►│ Deploy Artifacts │
└──────────────┘   └──────────┘   └───────┘   └───────┘   └──────────────────┘
```

| Stage | Maven Command | Output |
|---|---|---|
| **Git Checkout** | `git clone` | Source code pulled |
| **Compile** | `mvn compile` | Compiled `.class` files |
| **Tests** | `mvn test` | Test results & reports |
| **Build** | `mvn package` | `.jar` / `.war` artifact |
| **Deploy Artifacts** | `mvn deploy` | Artifact pushed to Nexus |

---

## 📁 Project Structure

```
Nexus-Demo/
├── src/
│   ├── main/java/          # Application source code
│   └── test/java/          # Unit tests
├── pom.xml                 # Maven build configuration & Nexus deploy settings
├── Jenkinsfile             # Jenkins declarative pipeline definition
└── README.md               # Project documentation
```

---

## ⚙️ Configuration

### Maven `settings.xml`

The pipeline uses a **Global Maven Settings** file (`settings.xml`) configured in Jenkins to authenticate with Nexus. Ensure the following is set up in Jenkins:

- **Global Tool Configuration** → Maven → named `maven3`
- **Global Tool Configuration** → JDK → named `jdk17`
- **Managed Files** → Global Maven Settings → ID: `settings.xml` (with Nexus credentials)

### `pom.xml` — Distribution Management

Your `pom.xml` should include the Nexus repository URLs for artifact deployment:

```xml
<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <url>http://<your-public-ip>:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>http://<your-public-ip>:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

---

## 🐳 Nexus Docker Reference

```bash
# Start Nexus
docker run -d --name nexus3 -p 8081:8081 sonatype/nexus3

# Check container status
docker ps

# View Nexus logs
docker logs -f nexus3

# Stop Nexus
docker stop nexus3

# Remove container
docker rm nexus3
```

---

## 🔒 Security Notes

> ⚠️ The following are recommended for production environments:

- Change the default Nexus `admin` password immediately after first login
- Disable **Anonymous Access** in production; enable only for internal trusted networks
- Use **HTTPS** with a valid SSL certificate for both Jenkins and Nexus
- Store Nexus credentials in Jenkins **Credentials Manager**, never in plain text
- Restrict VM port access using security group rules / firewall policies

---

## 🐛 Troubleshooting

| Issue | Possible Cause | Fix |
|---|---|---|
| Nexus page not loading | Container still starting | Wait 1–2 minutes and refresh |
| `mvn deploy` fails | Missing Nexus credentials | Verify `settings.xml` in Jenkins Managed Files |
| Jenkins not accessible | Port 8080 not open | Add inbound rule for port `8080` in VM security group |
| Docker command not found | Docker not installed | Re-run `docker.sh` install script |
| Wrong admin password | Copied shell prompt text | Re-enter container and copy only the password string |

---

## 📚 References

- 🔗 [Jenkins Official Docs](https://www.jenkins.io/doc/)
- 🔗 [Sonatype Nexus 3 Docker Image](https://hub.docker.com/r/sonatype/nexus3/)
- 🔗 [Docker Install on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- 🔗 [Maven Deploy Plugin](https://maven.apache.org/plugins/maven-deploy-plugin/)
- 🔗 [Demo Source Code — GitHub](https://github.com/kartikhegadi/Nexus.git)

---

## 👨‍💻 Author

**Kartik Hegadi**
- GitHub: [@kartikhegadi](https://github.com/kartikhegadi)

---

> 📝 *For a detailed step-by-step walkthrough with screenshots, refer to the full setup guide or the accompanying video tutorial.*
