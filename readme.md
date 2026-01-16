# Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project!
<div align="center">


  <br>
  <a href="http://netflix-clone-with-tmdb-using-react-mui.vercel.app/">
    <img src="./public/assets/netflix-logo.png" alt="Logo" width="100" height="32">
  </a>
</div>

<br />

<div align="center">
  <img src="./public/assets/home-page.png" alt="Logo" width="100%" height="100%">
  <p align="center">Home Page</p>
</div>


<img width="1236" height="672" alt="image" src="https://github.com/user-attachments/assets/ca295c68-7b40-4ac7-aa3d-ba7e8da1cb5f" />



**Get the API Key:**

- Open a web browser and navigate to TMDB (The Movie Database) website.
- Click on "Login" and create an account.
- Once logged in, go to your profile and select "Settings."
- Click on "API" from the left-side panel.
- Create a new API key by clicking "Create" and accepting the terms and conditions.
- Provide the required basic details and click "Submit."
- You will receive your TMDB API key.

![image](https://github.com/user-attachments/assets/7deedb6b-6c33-483e-a9a7-5208752a3b44)

TMDB API-KEY: 
````
eb3428e4a33e89ba1946e24fcb6cfede
````

## Step1: Launch EC2 instance
 - AMI: Ubuntu
 - Instance_Type: t2.medium
 - Volume: 30 GB

## Step2: Connect To Instance
 - Install Jenkins
 - Install Docker
 - Install SonarQube    # used for code quality testing
 - Install Trivy        # used to scan docker images
      
**Jenkins**
````
sudo apt update
sudo apt install fontconfig openjdk-21-jre  -y
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
````
**Docker**
````

sudo apt-get update
sudo apt-get install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
````
**SonarQube**
````
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
````

**Trivy**
````
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
````
## Step3: Connect to Jenkins 

## Step4: Connect to SonarQube
   - Admin->my account->security->generate token
![image](https://github.com/user-attachments/assets/26cb309d-aa3c-4a74-873f-9e87b2fcce00)

Step5: In Jenkins
     - Manage Jenkins: Credentials
       - Sonar-Token
       - Git-Cred
       - Docker-Cred
## Step6: Install Required Plugins:
   **Install below plugins**

````
Eclipse Temurin Installer 
````
````
SonarQube Scanner
````
````
NodeJs Plugin
````
````
docker
````
````
stage view
````

## Step7: Install  Tools: Manage Jenkins->Tools
   - add jdk: "jdk17" ->install from adoptium.net->version- 17
   - add SonarQube Scanner: "sonar-scanner"
   - add NodeJs: "node16" -> version 16.15.1
   - docker: "docker"

### **Configure Java and Nodejs in Global Tool Configuration**
Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save
#### Jdk
![image](https://github.com/user-attachments/assets/fe876745-d024-403c-806b-4a7d8c1dba11)
#### SonarQube-Scanner
![image](https://github.com/user-attachments/assets/24589963-9a7e-4d6a-9598-66580c195e30)

#### Node-js
![image](https://github.com/user-attachments/assets/51617874-be4d-438c-a93e-5a5d9e5781fa)
#### Docker
![image](https://github.com/user-attachments/assets/289c2e2a-df33-476b-a195-d584db3ef03e)



## Step8: Log in to Sonarqube and generate token
 - username: admin
 - password: admin
<img width="1902" height="957" alt="image" src="https://github.com/user-attachments/assets/36620768-5f81-440c-b31b-ecf29c609f64" />   

## Step9: Add DockerHub & Sonarqube Credentials:
   **Docker**
  - Go to  "Manage Jenkins" → Credentials."
  - Click on "Global."
  - Click on "Add Credentials" 
  - Choose "username with password" as the kind of credentials.
  - Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker-cred").
  - Click "OK" to save your DockerHub credentials.
    
     **SonarQube**
  - Go to  "Manage Jenkins" → Credentials."
  - Click on "Global."
  - Click on "Add Credentials" 
  - Choose "secret text" as the kind of credentials.
  - Enter your sonarqube token and give the credentials an ID (e.g., "sonar-token").
  - Click "create" to save yourcredentials

<img width="1907" height="846" alt="image" src="https://github.com/user-attachments/assets/bcd447f5-4a49-478d-99d6-1379202f4334" />

## Step10: Configure Sonar Server: Manage Jenkins->System
   - name: "sonar-server"
   - url:
   - token:
![image](https://github.com/user-attachments/assets/c5d05628-1502-4a92-b722-7ad3eed5d587)

## Step 11: Restart Jenkins

## Step12: Create Pipeline
````
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {

        stage('Code-Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/abhipraydhoble/netflix.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
       stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=020581a34f3ab93b1360a55bea864bd9 -t abhipraydh96/moviesite ."
                       sh "docker push abhipraydh96/moviesite "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image abhipraydh96/moviesite > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 abhipraydh96/moviesite'
            }
        }
        
    }
}
````
Note: 
- ensure jenkins user has permission to create container
   ````
   sudo usermod -aG docker jenkins
   newgrp docker
   sudo chmod 777 /var/run/docker.sock
   ````
- Verify all the names before running pipeline
  
## Output:
1. trivy output
<img width="1657" height="917" alt="image" src="https://github.com/user-attachments/assets/bdae8864-ac9e-4d8a-8aa2-4558deb114ed" />
2. sonarube ouput
<img width="1916" height="915" alt="image" src="https://github.com/user-attachments/assets/df9e8a1e-f903-4be8-8d5a-97af04f16ca4" />
3. netflix app
<img width="1888" height="872" alt="image" src="https://github.com/user-attachments/assets/fe5fbe19-e62f-42c7-b3e7-9454bc1cec5b" />





 
