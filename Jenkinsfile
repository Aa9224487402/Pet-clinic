pipeline {
    agent any 
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar scanner'
    }
    
    stages{
        
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/iam-arya/Pet-clinic.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar scanner') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }
        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
         stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
        
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: '7e584c64-2800-414c-a606-2de754fda992', toolName: 'docker') {
                        
                        sh "docker build -t winner ."
                        sh "docker tag winner aryashersingh2/pet-clinic123:latest "
                        sh "docker push aryashersingh2/pet-clinic123:latest "
                    }
                }
            }
        }
        
        stage("TRIVY"){
            steps{
                sh " trivy image aryashersingh2/pet-clinic123:latest"
            }
        }
        
        stage("Deploy To Tomcat"){
            steps{
                sh "cp  /var/lib/jenkins/workspace/winner/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
            }
        }
    }
}
