pipeline {
    agent any
    environment{
        registry='nitingoyal/samplenodeapp'
    }
    options {
        timestamps()
    }
    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Nitin-GitUser/nagp_jenkins_assignment.git'
            }
        }
        stage('Run Testcases') {
            steps {
                sh '''
                    npm install
                    npm run test
                '''
            }
        }
        stage('SonarQube Analysis') {
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
        stage('Build Image') {
            steps {
                sh '''
                    docker build . -t i-nitingoyal-master
                '''
            }
        }
        stage('Pushing Image to repository') {
            steps {
                sh 'docker tag i-nitingoyal-master ${registry}:${BUILD_NUMBER}'
                withDockerRegistry([credentialsId: 'Dockerhub', url: '']){
                    sh 'docker push ${registry}:${BUILD_NUMBER}'
                }
            }
        }
        stage('Deploy container') {
            steps {
              sh '''
                running_container=`docker ps -a | grep c-nitingoyal-master | awk '{print $1}'`
                if [[ $running_container != '' ]];
                    then  
                        docker stop $running_container;
                        docker rm $running_container 
                fi
				docker run --name c-nitingoyal-master -p 7100:7100 -d ${registry}:${BUILD_NUMBER}
			  '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}