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
                        docker rmi stratcastor/python-api || echo "Image does not exist"
                        docker rmi stratcastor/flask-nginx || echo "Image does not exist"
                        docker network create project || echo "network already exists"
                        '''
		        }
           }
        stage('Build') {
            steps {                        sh '''
                        docker build -t stratcastor/python-api -t stratcastor/python-api:v${BUILD_NUMBER} .   
                        docker build -t stratcastor/flask-nginx -t stratcastor/flask-nginx:v${BUILD_NUMBER} ./nginx             
                        '''
                    		        }
           }
        stage('Push') {
            steps {
                        sh '''
                        docker push stratcastor/python-api
                        docker push stratcastor/python-api:v${BUILD_NUMBER}
                        docker push stratcastor/flask-nginx
                        docker push stratcastor/flask-nginx:v${BUILD_NUMBER}
                        '''
		        }
           }
        stage('Deploy') {
            steps {
                         sh '''
                        ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                        docker run -d --name flask-app --network project stratcastor/python-api
                        docker run -d -p 80:80 --name nginx --network project stratcastor/flask-nginx
                        '''
                    }		        }
        stage('Cleanup') {
            steps {
                        sh '''
                        docker rmi stratcastor/python-api:v${BUILD_NUMBER}
                        docker rmi stratcastor/flask-nginx:v${BUILD_NUMBER}
                        '''
                    }
                }
    }
}