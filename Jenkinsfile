pipeline {
    agent any

    environment {
        ANDROID_HOME = '/home/ubuntu/Android/Sdk'
        PATH = "${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/tools:${env.PATH}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Write local.properties') {
            steps {
                sh '''
                    echo "sdk.dir=${ANDROID_HOME}" > local.properties
                    cat local.properties
                '''
            }
        }

        stage('Inject google-services.json') {
            steps {
                withCredentials([file(credentialsId: 'Quiz_App', variable: 'SERVICE_ACCOUNT')]) {
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
                sh './gradlew assembleDebug'
            }
        }

        stage('Archive APK Artifact') {
            steps {
                archiveArtifacts artifacts: 'app/build/outputs//*.apk', fingerprint: true
            }
        }

        stage('Deploy to Firebase') {
            steps {
                withCredentials([file(credentialsId: 'Quiz_App', variable: 'FIREBASE_KEY')]) {
                    withCredentials([string(credentialsId: 'FIREBASE_TOKEN', variable: 'FIREBASE_TOKEN')]) {
                        sh '''
                            cp $FIREBASE_KEY firebase-service.json
                            firebase appdistribution:distribute app/build/outputs/apk/debug/app-debug.apk \
                                --app 1:823895722189:android:b8f7135c3959cb295f0184 \
                                --token $FIREBASE_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Pipeline is working!'
            }
        }
    }
}
