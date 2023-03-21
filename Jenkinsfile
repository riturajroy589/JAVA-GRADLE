pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
        }
        stage("docker build and docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-pass', variable: 'nexus_password')]) {
                        sh '''
                    docker build -t 34.131.172.217:8083/springapp:${VERSION} .
                    docker login -u admin -p $nexus_password 34.131.172.217:8083
                    docker push 34.131.172.217:8083/springapp:${VERSION}
                    docker rmi 34.131.172.217:8083/springapp:${VERSION}
                    '''

                    }
                    
                }
            }
        }
        stage("datree misconfigs check"){
            steps{
                script{
                    dir('kubernetes/') {
                        sh 'helm datree test myapp/'
                    }

                }
            }
        }

    }
}

    

