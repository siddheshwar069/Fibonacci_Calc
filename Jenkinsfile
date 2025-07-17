pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage("Tests") {
            steps {
                sh "echo building client image"
                sh "docker build -t siddheshwar069/client:latest -f ./client/Dockerfile.dev ./client"
                sh "docker run siddheshwar069/client:latest -e CI=true npm test"
                sh "echo build is successful"
            }
        }

        stage("Build_Images") {
            steps {
                sh "echo building production images"
                sh "docker build -t siddheshwar069/client:latest ./client"
                sh "docker build -t siddheshwar069/nginx:latest ./nginx"
                sh "docker build -t siddheshwar069/server:latest ./server"
                sh "docker build -t siddheshwar069/worker:latest ./worker"
                sh "echo production images successfully built"
            }
        }

        stage("docker_login") {
            steps {
                echo "Logging in to Dockerhub..."
                withCredentials([usernamePassword(
                    credentialsId: '8dac37ca-8423-40c1-bc6c-70c741f2fc90',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "echo successfully logged in to Docker..."
                }
            }
        }
        stage("push_docker"){
            steps{
            sh '''
                echo pushing images to dockerhub
                docker push siddheshwar069/client:latest
                docker push siddheshwar069/worker:latest
                docker push siddheshwar069/server:latest
                docker push siddheshwar069/nginx:latest
                echo images successfully pushed to docker
            '''
            }
        }

        stage('compose_up'){
           steps{
            sh '''
                echo "building docker compose file and starting it"
                docker compose up --build -d
                echo compose is up and running
            '''
           }
        }

        stage("compose_down"){
            sh '''
                sleep 300
                docker compose down
            ''' 
        }
    }

    post {
        always {
            echo 'Logging out from Docker...'
            sh 'docker logout || true'
        }
    }
}
