
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
              git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/ashutoshdb/boardgame.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        
        stage('Build') {
            steps {
               sh "mvn package"
            }
        }
        
        stage('Publish To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"

                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t ashupop2/boardshack:latest ."
                    }
               }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html adijaiswal/boardshack:latest "
            }
        }
        
        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push ashupop2/boardshack:latest"
                             sh " docker rmi ashupop2/boardshack:latest"
                    }
               }
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
              withKubeConfig(caCertificate: '', clusterName: 'arn:aws:eks:us-east-1:590184030086:cluster/kube', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://BE035F0BF161F19E81A812F3E162E16D.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl apply -f deployment-service.yaml  --validate=false"
                }
            }
        }
        
        stage('Verify the Deployment') {
            steps {
              withKubeConfig(caCertificate: '', clusterName: 'arn:aws:eks:us-east-1:590184030086:cluster/kube', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://BE035F0BF161F19E81A812F3E162E16D.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl get pods -n webapps"
                        sh "kubectl get svc -n webapps"
                }
            }
        }
        
        
    }
//     post {
//     always {
//         // script {
//         //     // def jobName = env.JOB_NAME
//         //     // def buildNumber = env.BUILD_NUMBER
//         //     // def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
//         //     // def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

//         //     // def body = """
//         //     //     <html>
//         //     //     <body>
//         //     //     <div style="border: 4px solid ${bannerColor}; padding: 10px;">
//         //     //     <h2>${jobName} - Build ${buildNumber}</h2>
//         //     //     <div style="background-color: ${bannerColor}; padding: 10px;">
//         //     //     <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
//         //     //     </div>
//         //     //     <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
//         //     //     </div>
//         //     //     </body>
//         //     //     </html>
//         //     // """

//         //     // emailext (
//         //     //     subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
//         //     //     body: body,
//         //     //     to: 'jaiswaladi246@gmail.com',
//         //     //     from: 'jenkins@example.com',
//         //     //     replyTo: 'jenkins@example.com',
//         //     //     mimeType: 'text/html',
//         //     //     attachmentsPattern: 'trivy-image-report.html'
//         //     // )
//         // }
//     }
// }

}