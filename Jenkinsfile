pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                sh '''
                ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                docker network create jenkins-network || true
                docker stop flask-app || echo "flask-app not running"
                docker stop jenkins-nginx || echo "nginx Not Running"
                docker rm flask-app || echo "flask-app not running"
                docker rm jenkins-nginx || echo "nginx Not Running"
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
                docker run -d -p 80:8080 --name flask-app --network jenkins-network steoconnor/python-api
                docker run -d -p --name jenkins-nginx --network jenkins-network steoconnor/jenkins-nginx
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