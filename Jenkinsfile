pipeline {
    agent any
    environment {
        GCR_CREDENTIALS_ID = '86ef8dfb-3d8a-47f4-82f2-cdbda26e3d00'
        IMAGE_NAME = 'steve-gcr-python-api'
        GCR_URL = 'gcr.io/lbg-mea-16'
        PROJECT_ID = 'lbg-mea-16'
        CLUSTER_NAME = 'steve-cluster'
        LOCATION = 'europe-west2-c'
        CREDENTIALS_ID = 'steve-jenkins-gcr'
    }
    stages {
        stage('Build and Push to GCR') {
    steps {
        script {
            // Authenticate with Google Cloud
            withCredentials([file(credentialsId: GCR_CREDENTIALS_ID, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
            }
            // Configure Docker to use gcloud as a credential helper
            sh 'gcloud auth configure-docker --quiet'
            // Build the Docker image
            sh "docker build -t ${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER} ."
            // Push the Docker image to GCR
            sh "docker push ${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER}"
        }
    }
        }
        stage('Deploy to GKE') {
            steps {
                script {
			        if (env.GIT_BRANCH == 'origin/main') {
                    sh '''
                    ssh -i ~/.ssh/id_rsa jenkins@10.154.0.43 << EOF
                    docker run -d --name ${IMAGE_NAME} --network jenkins-network ${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER}
                    #docker run -d -p 80:80 --name nginx --network jenkins-network steoconnor/flask-nginx
                    '''
                    } else if (env.GIT_BRANCH == 'origin/develop') {
                    sh '''
                    #new test ip address
                    ssh -i ~/.ssh/id_rsa jenkins@10.154.0.29 << EOF 
                    docker run -d --name ${IMAGE_NAME} --network jenkins-network ${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER}
                    #docker run -d -p 80:80 --name nginx --network jenkins-network steoconnor/flask-nginx
                    '''
                    } else {
                    sh '''
                    echo "Unrecognised branch"
                    '''
                    }

                }
            }    
        }
    }
}