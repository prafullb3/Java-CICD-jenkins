pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv("Sonarserver") {
                    sh 'chmod +x gradlew'
                    sh './gradlew sonarqube'
                    //    tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    //    sh "${tool("sonar")}/bin/sonar-scanner"
                    }
                }
                }  
            }
            stage("sonar quality gate analysis"){
                steps{
                    waitForQualityGate abortPipeline: true
                }
            }
        
        
            stage("docker build & docker push"){
                steps{
                    script{
                        withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                                sh '''
                                    docker build -t 13.126.150.193:8083/springapp:${VERSION} .
                                    docker login -u admin -p $docker_password 13.126.150.193:8083 
                                    docker push  13.126.150.193:8083/springapp:${VERSION}
                                    docker rmi 13.126.150.193:8083/springapp:${VERSION}
                                '''
                        }
                    }
                }
            }
    
            stage("pushing the helm charts to nexus"){
                steps{
                    script{
                        withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                            dir('kubernetes/') {
                                sh '''
                                    helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                    tar -czvf  myapp-${helmversion}.tgz myapp/
                                    curl -u admin:$docker_password http://13.126.150.193:8081/repository/helm-charts/ --upload-file myapp-${helmversion}.tgz -v
                                '''
                        }
                    }
                }
            }
        }
    }
}
