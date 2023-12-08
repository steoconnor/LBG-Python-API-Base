pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                sh '''
                ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                docker network create jenkins-network || true
                docker stop python-api || echo "flask-app not running"
                docker stop jenkins-nginx || echo "nginx Not Running"
                docker rm python-api || echo "flask-app not running"
                docker rm jenkins-nginx || echo "nginx Not Running"
                docker rmi steoconnor/python-api || echo "no python-api image to remove"
                docker rmi steoconnor/jenkins-nginx || echo "no jenkins-nginx image to remove"
                '''
           }
        }
        stage('Build') {
            steps {
                sh '''
                docker build -t steoconnor/python-api -t steoconnor/python-api:v${BUILD_NUMBER} .   
                docker build -t steoconnor/jenkins-nginx -t steoconnor/jenkins-nginx:v${BUILD_NUMBER} ./nginx               
                '''
           }
        }
        stage('Push') {
            steps {
                sh '''
                docker push steoconnor/python-api
                docker push steoconnor/python-api:v${BUILD_NUMBER}
                docker push steoconnor/jenkins-nginx
                docker push steoconnor/jenkins-nginx:v${BUILD_NUMBER}
                '''
           }
        }
        stage('Deploy') {
            steps {
                sh '''
                ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                docker run -d --name python-api --network jenkins-network steoconnor/python-api
                docker run -d -p 80:80 --name jenkins-nginx --network jenkins-network steoconnor/jenkins-nginx
                '''
            }
        }
        stage('Cleanup') {
            steps {
                sh '''
                docker system prune -f 
                '''
           }
        }
    }
}