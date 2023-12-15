pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                script {
			        if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        kubectl create ns prod || echo "------- Prod Namespace Already Exists -------"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/develop') {
                        sh '''
                        kubectl create ns dev || echo "------- Dev Namespace Already Exists -------"
                        '''
                    } else {
                        sh '''
                        echo "Unrecognised branch"
                        '''
                    }
		        }
           }
        }
        stage('Build') {
            steps {
                script {
			        if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        docker build -t gcr.io/lbg-mea-16/steve-project-api -t gcr.io/lbg-mea-16/steve-project-api:prod-v${BUILD_NUMBER} . 
                        '''
                    } else if (env.GIT_BRANCH == 'origin/develop') {
                        sh '''
                        docker build -t gcr.io/lbg-mea-16/steve-project-api -t gcr.io/lbg-mea-16/steve-project-api:dev-v${BUILD_NUMBER} .         
                        '''
                    } else {
                        sh '''
                        echo "Unrecognised branch"
                        '''
                    }
		        }
           }
        }
        stage('Push') {
            steps {
                script {
			        if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        docker push gcr.io/lbg-mea-16/steve-project-api
                        docker push gcr.io/lbg-mea-16/steve-project-api:prod-v${BUILD_NUMBER}
                        '''
                    } else if (env.GIT_BRANCH == 'origin/develop') {
                        sh '''
                        docker push gcr.io/lbg-mea-16/steve-project-api
                        docker push gcr.io/lbg-mea-16/steve-project-api:dev-v${BUILD_NUMBER}
                        '''
                    } else {
                        sh '''
                        echo "Unrecognised branch"
                        '''
                    }
		        }
           }
        }
        stage('Deploy') {
            steps {
                script {
			        if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        kubectl apply -n prod -f ./kubernetes
                        kubectl set image deployment/flask-api-deployment flask-container=gcr.io/lbg-mea-16/steve-project-api:prod-v${BUILD_NUMBER} -n prod
                        '''
                    } else if (env.GIT_BRANCH == 'origin/develop') {
                        sh '''
                        kubectl apply -n dev -f ./kubernetes
                        kubectl set image deployment/flask-api-deployment flask-container=gcr.io/lbg-mea-16/steve-project-api:dev-v${BUILD_NUMBER} -n dev
                        '''
                    } else {
                        sh '''
                        echo "Unrecognised branch"
                        '''
                    }
		        }
            }
        }
        stage('Cleanup') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        docker rmi gcr.io/lbg-mea-16/steve-project-api:prod-v${BUILD_NUMBER}
                        '''
                    } else if (env.GIT_BRANCH == 'origin/develop') {
                        sh '''
                        docker rmi gcr.io/lbg-mea-16/steve-project-api:dev-v${BUILD_NUMBER}
                        '''
                    }
                }
                sh '''
                docker rmi gcr.io/lbg-mea-16/steve-project-api:latest || echo "gcr.io/lbg-mea-16/steve-project-api:latest doesn't exist"
                docker system prune -f 
                '''
           }
        }
    }
}