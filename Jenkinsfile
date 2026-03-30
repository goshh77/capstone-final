pipeline {
    agent any

    environment {
        PROJECT_ID = 'project-2-490904'
        REGION = 'us-central1'
        ZONE = 'us-central1-c' 
        MIG_NAME = 'capstone-backend-mig'
        IMAGE = "us-central1-docker.pkg.dev/${PROJECT_ID}/frontend-repo/frontend-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Set GCP Project') {
            steps {
                sh "gcloud config set project ${PROJECT_ID}"
            }
        }

        stage('Build & Deploy Frontend') {
            steps {
                dir('frontend') {
                    sh "gcloud builds submit --tag ${IMAGE} . --quiet"
                }
                sh "gcloud run deploy capstone-frontend --image ${IMAGE} --platform managed --region ${REGION} --allow-unauthenticated --quiet"
            }
        }

        stage('Wait Before Backend Restart') {
            steps {
                echo "Waiting before backend restart..."
                sh "sleep 15"
            }
        }

        stage('Deploy Backend (MIG Refresh)') {
            steps {
                echo "🚀 Starting Rolling Update for Backend..."
                sh """
                gcloud compute instance-groups managed rolling-action restart ${MIG_NAME} \
                --zone=${ZONE} \
                --max-unavailable=1 \
                --quiet
                """
            }
        }
    }

    post {
        success {
            echo "🎉 SUCCESS: Full-Stack Deployment (Frontend + Backend) is Complete!"
        }
        failure {
            echo "❌ Deployment Failed. Check Console Output."
        }
    }
}