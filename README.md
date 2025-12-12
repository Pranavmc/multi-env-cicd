**Multi-Environment CI/CD Pipeline (Dev → QA → Prod) using Jenkins, GitHub & AWS EC2**
This project demonstrates a complete CI/CD pipeline that automatically builds, packages, and deploys a Java application across **three environments**:
- Dev  
- QA  
- Production  

The pipeline is implemented using **Jenkins Declarative Pipeline**, **EC2 application servers**, **GitHub**, and **Maven**.

**Project Features**
- Automatic build and packaging using **Maven**
- Deployment to **Dev**, followed by health checks
- **Manual approval gates** for QA and Prod
- Deployment to **QA and Production**
- Environment-specific configurations using `.env` files
- Log management per environment
- Reusable, scalable pipeline design

**Project Structure**
multi-env-cicd/
│
├── src/
├── target/
├── config/
│ ├── dev.env
│ ├── qa.env
│ └── prod.env
│
├── Jenkinsfile
├── pom.xml
└── README.md

**Technologies Used**
- Jenkins Declarative Pipeline  
- GitHub (SCM)  
- AWS EC2 (Dev, QA & Prod servers)  
- Java 17  
- Maven  
- SCP & SSH for deployment  
- cURL for health checks  

**Pipeline Architecture**
GitHub Repo → Jenkins Pipeline → Dev Server (Auto Deploy)
↓
QA Deployment (Manual Approval)
↓
Prod Deployment (Manual Approval)



**Jenkins Pipeline Stages**

**1. Checkout Source Code**
Pulls the application code from GitHub.

**2. Build Stage**
Builds the Java project and creates a JAR file using Maven.

**3. Package Artifact**
Copies the JAR file to an `artifacts/` directory with the version number.

**4. Deploy to Dev**
- Transfers JAR to Dev EC2 server
- Transfers `dev.env`
- Starts the application with a port (8080)
- Performs health check using cURL

**5. Manual Approval**
Jenkins pauses execution until user approval.

**6. Deploy to QA**
Similar to Dev but uses QA `.env` and port.

**7. Deploy to Prod**
Final deployment after approval.

**EC2 Server Setup (Required Before Running Pipeline)**

Run these commands on **all Dev, QA, and Prod EC2 instances**:

**1. Install Java**
sudo apt update
sudo apt install openjdk-17-jdk -y

**2. Create application directories**
sudo mkdir -p /opt/app/releases
sudo mkdir -p /opt/app/current

**3. Set permissions**
sudo chown -R ubuntu:ubuntu /opt/app

**4. Allow port in EC2 security group**
For example (Dev → 8080):
Type: Custom TCP
Port: 8080
Source: 0.0.0.0/0

Environment Configuration Files
**dev.env**
APP_ENV=dev
APP_PORT=8080
DB_URL=dev-db-url

**qa.env**
APP_ENV=qa
APP_PORT=8080
DB_URL=qa-db-url

**prod.env**
APP_ENV=prod
APP_PORT=8080
DB_URL=prod-db-url

**Trigger Pipeline**
Push any update to GitHub:
git add .
git commit -m "update"
git push origin main


**Jenkins automatically:**
- Pulls code  
- Builds JAR  
- Deploys to Dev  
- Waits for approval  
- Deploys to QA  
- Waits again  
- Deploys to Prod

**Final Result**
This project represents a **production-grade CI/CD workflow**, demonstrating:
- Automated builds  
- Environment-based deployments  
- Manual approval stages  
- Real EC2 server provisioning  
- Health monitoring  
- Reusable pipeline structure 
 
