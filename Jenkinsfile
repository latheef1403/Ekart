pipeline {
    agent any
    
    /* Particular Version Recommended to Use in Project */
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }

    stages {
        
        /* Clone the code from https://github.com/chinnayyachintha/Ekart/tree/main*/
        stage('SCM') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/chinnayyachintha/Ekart.git']])
            }
        }
        
        /* Compile source code and skip Test cases why? because in this project we will get error */
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        /* OWASP Dependency Check is a software composition analysis (SCA) tool that identifies project dependencies with known vulnerabilities */
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'Dependency-Check'
            }
        }
        
        /* Publish report in the form xml/html */
        stage('Publish OWASP Dependency Check Report') {
            steps {
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        /* Scanning files to verify wheter is error free or not */
        stage("TRIVY FS  SCAN"){
            steps{
                sh "trivy fs ."
            }
        }
        
        /* To Check Vulunerabilities and Code Smells through Sonarqube Scanner */
        stage('Sonarqube Analysis'){
            steps{
                sh 'mvn clean package -DskipTests=true'
                sh '''mvn sonar:sonar \
                         -Dsonar.projectKey=Ekart-Project \
                         -Dsonar.projectName='Ekart-Project' \
                         -Dsonar.host.url=http://20.81.235.176:9000 \
                         -Dsonar.token=sqp_704f6699d679e0dca52b573d09e51e1f762a919b
                    '''
            }
        }
     
        /* Deploy war/jar/ear file through maven */
        stage('Deploy to Nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: 'New-Global-Config-File', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true){
                    sh "mvn deploy -DskipTests=true"  
                }
            }
        }
        
        /* Build the image and tag  */
        stage('Docker Build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        sh "docker build -t shopping-cart-1 -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart chinnayya339/shopping-cart-1:v1.1.0"
                
                    }
                }
            }
        }
        
        /* To Scan image to Find vulnerabilities in image */
        stage("TRIVY IMAGE SCAN"){
            steps{
                sh " trivy image chinnayya339/shopping-cart-1:v1.1.0"
            }
        }
        
        /* Push tagged image into Docker-Hub and Deploy to Docker Container */
        stage('Docker Push & Deploy to Docker Container') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        sh "docker push chinnayya339/shopping-cart:v1.0.0"
                        sh "docker run --name Ekart -d -p 8070:8070 chinnayya339/shopping-cart-1:v1.1.0"
                        
                    }
                }
            }
        }
    }
}
