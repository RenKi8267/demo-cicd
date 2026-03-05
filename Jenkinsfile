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

        stage('Run') {
            when {
                expression {
                    // Non-multibranch jobs often have no BRANCH_NAME; allow local CI/CD testing.
                    return !env.BRANCH_NAME || env.BRANCH_NAME in ['main', 'master']
                }
            }
            steps {
                sh '''
                set -e

                APP_PORT=9097
                APP_PID=$(lsof -ti:$APP_PORT || true)
                if [ -n "$APP_PID" ]; then
                  kill -9 $APP_PID
                fi

                JAR_FILE=$(ls -1 target/*.jar | grep -v '\\.original$' | head -n 1 || true)
                if [ -z "$JAR_FILE" ]; then
                  echo "No runnable jar found in target/"
                  exit 1
                fi

                BUILD_ID=dontKillMe JENKINS_NODE_COOKIE=dontKillMe nohup java -jar "$JAR_FILE" --server.port=$APP_PORT > app.log 2>&1 &

                sleep 5
                NEW_PID=$(lsof -ti:$APP_PORT || true)
                if [ -z "$NEW_PID" ]; then
                  echo "App failed to start. Last 200 lines of app.log:"
                  tail -n 200 app.log || true
                  exit 1
                fi

                echo "Spring Boot started on port $APP_PORT with PID $NEW_PID"
                '''
            }
        }
    }
}
