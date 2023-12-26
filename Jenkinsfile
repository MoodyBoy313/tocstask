pipeline {
    agent any

    stages {
      stage('Build') {
        steps {
          script {
            dockerImage = docker.build("mehmoodahmed313/resume:${env.BUILD_ID}")
        }
    }
}
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockeerhub') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html' // Simple check for index.html
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy the new version
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "thiird", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull mehmoodahmed313/resume:${env.BUILD_ID}
                                        docker stop mehmoodahmed313-cv-container || true
                                        docker rm mehmoodahmed313-cv-container || true
                                        docker run -d --name mehmoodahmed313-cv-container -p 80:80 mehmoodahmed313/resume:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                    // Check if deployment is successful
                    boolean isDeploymentSuccessful = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://3.25.95.24:80', returnStdout: true).trim() == '200'

                    if (!isDeploymentSuccessful) {
                        // Rollback to the previous version
                        def previousSuccessfulTag = readFile('previous_successful_tag.txt').trim()
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "thiird",
                                    transfers: [sshTransfer(
                                        execCommand: """
                                            docker pull mehmoodahmed313/resume:${previousSuccessfulTag}
                                            docker stop mehmoodahmed313-cv-container || true
                                            docker rm mehmoodahmed313-cv-container || true
                                            docker run -d --name mehmoodahmed313-cv-container -p 80:80 junaid345/resume:${previousSuccessfulTag}
                                        """
                                    )]
                                )
                            ]
                        )
                    } else {
                        // Update the last successful tag
                        writeFile file: 'previous_successful_tag.txt', text: "${env.BUILD_ID}"
                    }
                }
            }
        }
    }

    post {
        success {
            mail(
                to: 'modhipaji786@gmail.com',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
            
        failure {
            mail(
                to: 'modhipaji786@gmail.com',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
    }
}
