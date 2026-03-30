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
                sh "sleep 10"
            }
        }

        stage('Deploy Backend (MIG Refresh)') {
            steps {
                echo "🚀 Starting Rolling Update for Regional Backend..."
                // FIXED: Set to 3 (matches the number of zones in us-central1)
                sh """
                gcloud compute instance-groups managed rolling-action restart ${MIG_NAME} \
                --region=${REGION} \
                --max-unavailable=3 \
                --quiet
                """
            }
        }
    }

    post {
        success {
            echo "🎉 MISSION ACCOMPLISHED: Full-Stack CI/CD is Successful!"
        }
        failure {
            echo "❌ Build failed. Check the console logs."
        }
    }
}