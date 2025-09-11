# SonarQube 
- is a platform used for continuous inspection of code quality and security.
- It acts as a code quality tool, analyzing code to identify bugs, vulnerabilities, code smells, and code duplications.
- Essentially, it helps developers maintain and improve the quality and security of their codebase.

  
**Code Quality Analysis:**
- SonarQube performs static code analysis, examining code without running it to identify potential issues.

**Bug Detection:**
- It helps find bugs, such as those caused by programming errors or logic flaws.
  
**Vulnerability Identification:**
- SonarQube identifies security vulnerabilities in the code, helping developers prevent potential attacks.
  
**Code Smell Detection:**
- It flags code smells, which are patterns in the code that indicate potential problems or inefficiencies.
  
**Code Duplication Detection:**
- SonarQube can detect and highlight duplicated code, which can be a sign of poor code organization.

**Security Assessment:**
- SonarQube helps assess the security of the code and identifies potential vulnerabilities.

**Code Coverage Analysis:**
- It analyzes the amount of code covered by unit tests, helping developers ensure adequate testing.
  
**Maintainability Improvement:**
- By identifying and addressing issues, SonarQube contributes to creating more maintainable and robust code.
  
**Technical Debt Reduction:**
- Early identification and resolution of issues help reduce the long-term cost of fixing problems.


# pipeline 1
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
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
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
        stage('Deploy'){
            steps{
                 sh "docker build --build-arg TMDB_V3_API_KEY=020581a34f3ab93b1360a55bea864bd9 -t netflix ."
                 sh "docker run -itd --name netflix -p 8081:80 netflix"
            }
        }
        
    }
}
````

# pipeline 2
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
# TMDB API KEY
````
f6ce6c8a5aa58b38fd70f6d5c6b06fda
````
