pipeline {

    agent any

    environment {

        SONAR_HOST_URL = 'http://localhost:9000'

        SONAR_PROJECT_KEY = 'qjs'
        SONAR_PROJECT_NAME = 'qjs'

        SONAR_TOKEN = credentials('SONAR_TOKEN')
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('SonarQube Analysis') {
            steps {

                withSonarQubeEnv('sonarqube-server') {

                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                        -Dsonar.sources=src \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.token=${SONAR_TOKEN} \
                        -Dsonar.exclusions=node_modules/**,dist/** \
                        -Dsonar.typescript.tsconfigPath=tsconfig.json
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }

    post {

        always {
            echo 'Pipeline completed!'
        }

        success {
            echo 'Build successful! Code quality passed SonarQube checks.'
        }

        failure {
            echo 'Build failed. Check SonarQube for issues.'
        }
    }
}
