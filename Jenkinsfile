pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/praveenselvakumarasamy/Multi_Tier_Bankapp.git'
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar_server') {
                    sh '''mvn sonar:sonar -Dsonar.projectName=bankapp \
                    -Dsonar.projectKey=bankapp  -Dsonar.java.binaries=target'''
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o report.html .'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Publish On Nexus') {
            steps { 
                withMaven(globalMavenSettingsConfig: 'Nexus_setting', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            environment{
                DOCKER_IMAGE = "praveenselvakumarasamy/bankapp:latest"
            }
            
            steps {
                script{
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    withDockerRegistry(credentialsId: 'docker_cred') {
                        sh 'docker push ${DOCKER_IMAGE}'
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o report1.html praveenselvakumarasamy/bankapp:latest'
            }
        }

        stage('Update Deployment File') {
            steps {
                git branch: 'main', credentialsId: 'git_cred', url: 'https://github.com/praveenselvakumarasamy/Multi_Tier_Bankapp.git'{
                    sh '''git config user.email "praveenpup7@gmail.com" \
                    git config user.name "praveenselvakumarasamy" \
                    sed -i "s/replaceimagetag/latest/g" Multi_Tier_Bankapp/bankapp_charts/templates/values.yaml \
                    git add Multi_Tier_Bankapp/bankapp_charts/templates/Deployment.yaml \
                    git commit -m "bankapp commit 1" \
                    git push -u origin main '''
                }
            }
        }
    }
}

