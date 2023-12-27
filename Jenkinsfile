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
                    docker.withRegistry('https://registry.hub.docker.com', '4d46a31c-a4c7-454a-a4f2-62e664ce28d4') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html'
            }
        }

        stage('Deploy') {
            steps {
                script {
                   
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "new", 
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

                  
                    boolean isDeploymentSuccessful = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://172.31.16.189:80', returnStdout: true).trim() == '200'

                    if (!isDeploymentSuccessful) {
                       
                        def previousSuccessfulTag = readFile('previous_successful_tag.txt').trim()
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "new",
                                    transfers: [sshTransfer(
                                        execCommand: """
                                            docker pull mehmoodahmed313/resume:${previousSuccessfulTag}
                                            docker stop  mehmoodahmed313-cv-container || true
                                            docker rm mehmoodahmed313-cv-container || true
                                            docker run -d --name mehmoodahmed313-cv-container -p 80:80 junaid345/resume:${previousSuccessfulTag}
                                        """
                                    )]
                                )
                            ]
                        )
                    } else {
                       
                        writeFile file: 'previous_successful_tag.txt', text: "${env.BUILD_ID}"
                    }
                }
            }
        }
    }

    post {
        failure {
            mail(
                to: 'fa20-bse-072@cuiatk.edu.pk',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: """Something is wrong with the build ${env.BUILD_URL}
                Rolling back to the previous version

                Regards,
                Jenkins
                
                """
            )
        }
    }
}
