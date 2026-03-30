pipeline {
    agent any

    environment {
        PROJECT_ID = 'project-2-490904'
        REGION = 'us-central1'
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

        stage('Wait for Sync') {
            steps {
                sh "sleep 15"
            }
        }

        stage('Deploy Backend (MIG Refresh)') {
            steps {
                echo "🚀 Starting Zero-Downtime Rolling Update..."
                // FIXED: Changed --max-unavailable to 0 for Regional MIG compliance
                sh """
                gcloud compute instance-groups managed rolling-action restart ${MIG_NAME} \
                --region=${REGION} \
                --max-unavailable=0 \
                --quiet
                """
            }
        }
    }

    post {
        success {
            echo "🎉 MISSION COMPLETE: Full-Stack CI/CD is Live and Green!"
        }
        failure {
            echo "❌ Still an issue. Check the logs."
        }
    }
}