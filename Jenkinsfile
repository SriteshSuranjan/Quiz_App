pipeline {
    agent any

    environment {
        // Update with your Android SDK path in WSL
        ANDROID_HOME = '/home/ubuntu/Android/Sdk'
        PATH = "${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools:${PATH}"
        GRADLE_OPTS = "-Dorg.gradle.daemon=false"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git(
                    url: 'https://github.com/SriteshSuranjan/Quiz_App.git',
                    branch: 'main',
                    credentialsId: 'Quiz_App' // Create this in Jenkins credentials
                )
            }
        }

        stage('Build Debug APK') {
            steps {
                sh './gradlew assembleDebug --stacktrace'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'app/build/outputs/apk/debug/*.apk',
                    fingerprint: true
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh './gradlew test'
            }
            post {
                always {
                    junit 'app/build/test-results/**/*.xml'
                }
            }
        }

        stage('Code Quality Check') {
            steps {
                sh './gradlew lintDebug'
            }
            post {
                always {
                    androidLint pattern: 'app/build/reports/lint-results-*.xml'
                }
            }
        }

        // Optional: Only runs when merging to main
        stage('Deploy to Firebase') {
            when {
                branch 'main'
            }
            steps {
                script {
                    withCredentials([file(credentialsId: 'firebase-key', variable: 'FIREBASE_CONFIG']) {
                        sh '''
                        cp $FIREBASE_CONFIG app/firebase-service.json
                        ./gradlew appDistributionUploadDebug
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // Slack notification example (uncomment after setting up)
            // slackSend channel: '#android-ci', message: "Build ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            
            // Email notification
            emailext (
                subject: "Quiz App Build ${currentBuild.currentResult}",
                body: """
                Build: ${env.JOB_NAME} #${env.BUILD_NUMBER}
                Status: ${currentBuild.currentResult}
                URL: ${env.BUILD_URL}
                """,
                to: 'sritesh@example.com' // Update your email
            )
        }
    }
}
