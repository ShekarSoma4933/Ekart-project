pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git CheckOut') {
            steps {
               git branch: 'main', url: 'https://github.com/ShekarSoma4933/Ekart-project.git'
            }
        }
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Tests') {
            steps {
               sh "mvn test -DskipTests=true "
            }
        }
       /* stage("SonarQube Analysis") {
            steps {
               withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART -Dsonar.java.binaries=. ''' 
                }
            }
        }*/
        
        stage("OWASP dependecy Check"){
            steps{
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Maven build"){
            steps{
                sh "mvn package -DskipTests=true"
            }
        }
        stage("Docker Image Build"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh 'docker build -t shekarsoma493/ekart:$BUILD_NUMBER -f docker/Dockerfile .'
                     }
                }
            }
        }
        stage("Trivy Image scan"){
           steps{
             sh 'trivy image shekarsoma493/ekart:$BUILD_NUMBER > trivy-report.txt'  
           } 
        }
        stage("Docker Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh 'docker push shekarsoma493/ekart:$BUILD_NUMBER'
                     }
                }
            }
        }
// CD Pipeline

      stage("Cleanup Workspace"){
        steps{
          cleanWs()
        }
      }
      stage('Git CheckOut Ekart-argocd') {
            steps {
               git branch: 'main', url: 'https://github.com/ShekarSoma4933/Ekart-argocd.git'
            }
        }
      stage("Update Manifest file"){
        steps{
          script{
                sh "cat deployment-ekart.yaml"
                sh "sed -i 's/ekart.*/ekart:${BUILD_NUMBER}/g' deployment-ekart.yaml"
                }
            }
        }
      stage("Push the manifest changes to EkartArgocd repo"){
        steps{
          script{
              withCredentials([usernamePassword(credentialsId: 'git-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                  sh '''
                      git config --global user.name "shekarsoma4933"
                      git config --global user.mail "shekarsoma4933@gmail.com"
                      git remote set-url origin https://${USER}:${PASS}@github.com/ShekarSoma4933/Ekart-argocd.git
                      git add .
                      git commit -m "Update deployment manifest file"
                      git push origin main
                    '''
              }
          }
        }
      }
    }
}

