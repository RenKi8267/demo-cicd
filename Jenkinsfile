pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw -v'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Run (main only)') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                APP=$(lsof -ti:9097 || true)
                if [ ! -z "$APP" ]; then
                  kill -9 $APP
                fi
                setsid java -jar target/*.jar > app.log 2>&1 < /dev/null &
                '''
            }
        }
    }
}