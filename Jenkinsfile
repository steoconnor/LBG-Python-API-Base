pipeline {
    agent any
    stages {
        stage('Init') {
            steps {     sh '''
                        ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                        docker stop flask-app || echo "flask-app not running"
                        docker rm flask-app || echo "flask-app not running"
                        docker stop nginx || echo "nginx not running"
                        docker rm nginx || echo "nginx not running"
                        docker rmi steoconnor/python-api || echo "Image does not exist"
                        docker rmi steoconnor/flask-nginx || echo "Image does not exist"
                        docker network create project || echo "network already exists"
                        '''
		        }
           }
        stage('Build') {
            steps {  
                        sh '''
                        docker build -t steoconnor/python-api -t steoconnor/python-api:v${BUILD_NUMBER} .   
                        docker build -t steoconnor/flask-nginx -t steoconnor/flask-nginx:v${BUILD_NUMBER} ./nginx             
                        '''
                  }
           }
        stage('Push') {
            steps {
                        sh '''
                        docker push steoconnor/python-api
                        docker push steoconnor/python-api:v${BUILD_NUMBER}
                        docker push steoconnor/flask-nginx
                        docker push steoconnor/flask-nginx:v${BUILD_NUMBER}
                        '''
		        }
           }
        stage('Deploy') {
            steps {
                         sh '''
                        ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                        docker run -d --name flask-app --network project steoconnor/python-api
                        docker run -d -p 80:80 --name nginx --network project steoconnor/flask-nginx
                        '''
                    }		        }
        stage('Cleanup') {
            steps {
                        sh '''
                        docker rmi steoconnor/python-api:v${BUILD_NUMBER} || echo "flask nginx image doesn't exist"
                        docker rmi steoconnor/flask-nginx:v${BUILD_NUMBER} || echo "flask nginx image doesn't exist"
                        '''
                    }
                }
    }
}