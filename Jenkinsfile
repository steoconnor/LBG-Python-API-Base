pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                       sh '''
                       ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                       docker network create jenkins-network || echo "network already exists"
                       docker stop flask-app || echo "flask-app not running"
                       docker rm flask-app || echo "flask-app doesn't exist"
                       docker stop nginx || echo "nginx not running"
                       docker rm nginx || echo "nginx doesn't exist"
                       docker rmi steoconnor/flask-nginx || echo "no flask-nginx image to remove"
                       '''
                }
            }        
        }
        stage('Build') {
            steps {
                 script {
                    sh '''
                    docker build -t python-api -t steoconnor/python-api:v${BUILD_NUMBER} .   
                    docker build -t flask-nginx -t steoconnor/flask-nginx:v${BUILD_NUMBER} ./nginx               
                    '''
                }
            }    
        }
        stage('Push') {
            steps { 
                script {
                    sh '''
                    docker push steoconnor/python-api
                    docker push steoconnor/python-api:v${BUILD_NUMBER}
                    docker push steoconnor/flask-nginx
                    docker push steoconnor/flask-nginx:v${BUILD_NUMBER}
                    '''
                }    
           }
        }
        stage('Deploy') {
            steps {
                    sh '''
                    ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                    docker run -d --name flask-app --network jenkins-network steoconnor/python-api
                    docker run -d -p 80:80 --name nginx --network jenkins-network steoconnor/flask-nginx
                    '''
            }    
        }
        stage('Cleanup') {
            steps {
                sh '''
                docker system prune -f
                docker rmi steoconnor/python-api || echo "no python-api image to remove"
                docker rmi steoconnor/flash-nginx || echo "no flash-nginx image to remove"
                '''
           }
        }
    }
}