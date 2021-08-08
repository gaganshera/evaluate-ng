pipeline {
    agent any
    environment{
        registry='nitingoyal/samplenodeapp'
        deployment_port="${env.BRANCH_NAME == "develop" ? 7300 : 7200}" 
    }
    options {
        timestamps()
    }
    stages {
        stage('Build') {
            steps {
                sh '''
                    npm install
                '''
            }
        }
        stage('Unit Testing') {
            steps {
                sh '''
                    npm run test
                '''
            }
        }
        stage('Sonar Analysis') {
            when {
                anyOf {
                    branch 'develop';
                }
            }
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }
            steps{
                script {
                    withSonarQubeEnv('Test_Sonar') {
						sh '''${scannerHome}/bin/sonar-scanner \
						        -Dsonar.projectKey=sonar-nitingoyal 
						
						'''
                    }
                }
            }
        }
        stage('Docker Image') {
            steps {
                sh '''
                    docker build . -t i-nitingoyal-master
                '''
            }
        }
        stage('Containers'){
            parallel {
                stage('Pre Container Check') {
                    steps {
                        sh '''
                            running_container=`docker ps -a | grep samplenodeapp-${BRANCH_NAME} | awk '{print $1}'`
                            if [[ $running_container != '' ]];
                                then  
                                    docker stop $running_container;
                                    docker rm $running_container 
                            fi
            			  '''
                    }
                }
                stage('Pushing DockerHub') {
                    steps {
                        sh 'docker tag i-nitingoyal-master ${registry}:${BUILD_NUMBER}'
                        sh 'docker tag i-nitingoyal-master ${registry}:latest'
                        withDockerRegistry([credentialsId: 'Dockerhub', url: '']){
                            sh 'docker push ${registry}:${BUILD_NUMBER}'
                            sh 'docker push ${registry}:latest'
                        }
                    }
                }
            }
        }
        stage('Docker Deployment') {
            steps {
              sh '''
				docker run --name samplenodeapp-${BRANCH_NAME} -p ${deployment_port}:7100 -d ${registry}:${BUILD_NUMBER}
			  '''
            }
        }
        stage('Kubernetes Deployment') {
            steps{
                sh 'kubectl apply -f deployments.yaml -l branch=${BRANCH_NAME}'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}