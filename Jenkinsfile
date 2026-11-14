pipeline {
    agent any

    // Fires when GitHub sends a push webhook to http://<jenkins>/github-webhook/
    triggers {
        githubPush()
    }

    environment {
        IMAGE_NAME = 'portfolio-website'
        HOST_PORT  = '80'   // change if port 80 is busy (e.g. 8081)
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose up -d --build --remove-orphans'
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    for i in 1 2 3 4 5 6 7 8 9 10; do
                        if curl -fs http://localhost:${HOST_PORT}/ > /dev/null; then
                            echo "Site is up"
                            exit 0
                        fi
                        sleep 3
                    done
                    echo "Site did not come up"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            sh 'docker image prune -f'
            echo 'Deployed successfully'
        }
        failure {
            sh 'docker compose logs --tail=50 || true'
            echo 'Deployment failed'
        }
    }
}
