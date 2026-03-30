pipeline {
    agent any

    environment {
        PROJECT_ID = 'project-2-490904'
        REGION = 'us-central1'
        IMAGE = "us-central1-docker.pkg.dev/${PROJECT_ID}/frontend-repo/frontend-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set GCP Project') {
            steps {
                // Robot logs into the correct project
                sh "gcloud config set project ${PROJECT_ID}"
            }
        }

        stage('Cloud Build') {
            steps {
                dir('frontend') {
                    // Triggers Google's remote builder
                    sh "gcloud builds submit --tag ${IMAGE} ."
                }
            }
        }

        stage('Deploy to Cloud Run') {
            steps {
                sh "gcloud run deploy capstone-frontend --image ${IMAGE} --platform managed --region ${REGION} --allow-unauthenticated"
            }
        }
    }
    
    post {
        success {
            echo "🎉 Pipeline Finished Successfully! Website is updated."
        }
        failure {
            echo "❌ Pipeline Failed. Check the Console Output logs."
        }
    }
}