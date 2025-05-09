pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Inject google-services.json') {
            steps {
                withCredentials([file(credentialsId: 'google-services-json-secret', variable: 'SERVICE_ACCOUNT')]) {
                    sh 'cp $SERVICE_ACCOUNT app/google-services.json'
                }
            }
        }

        stage('Set Gradle Executable') {
            steps {
                sh 'chmod +x ./gradlew'
            }
        }

        stage('Clean Project') {
            steps {
                sh './gradlew clean'
            }
        }

        stage('Build APK') {
            steps {
                sh './gradlew assembleDebug' // or assembleRelease if needed
            }
        }

        stage('Archive APK Artifact') {
            steps {
                archiveArtifacts artifacts: 'app/build/outputs/**/*.apk', fingerprint: true
            }
        }

        // Optional: Firebase App Distribution
        // stage('Deploy to Firebase') {
        //     steps {
        //         withCredentials([file(credentialsId: 'firebase-service-json', variable: 'FIREBASE_KEY')]) {
        //             sh '''
        //                 cp $FIREBASE_KEY firebase-service.json
        //                 firebase login:ci --token $FIREBASE_TOKEN
        //                 firebase appdistribution:distribute app/build/outputs/apk/debug/app-debug.apk \
        //                     --app <YOUR_APP_ID> \
        //                     --token $FIREBASE_TOKEN
        //             '''
        //         }
        //     }
        // }
    }
}

