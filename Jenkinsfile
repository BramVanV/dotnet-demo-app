pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        DEPLOY_DIR = '/home/vagrant/dotnet-deploy'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log -1 --oneline'
            }
        }

        stage('Prepare') {
            steps {
                sh '''
                    set -eu
                    mkdir -p "$DEPLOY_DIR"
                    cp -R TodoApp "$DEPLOY_DIR/"
                    cp docker-compose.yml "$DEPLOY_DIR/"

                    docker compose \
                      -p dotnetlab \
                      -f "$DEPLOY_DIR/docker-compose.yml" \
                      config --quiet
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker compose \
                      -p dotnetlab \
                      -f "$DEPLOY_DIR/docker-compose.yml" \
                      build
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker compose \
                      -p dotnetlab \
                      -f "$DEPLOY_DIR/docker-compose.yml" \
                      up -d --wait --wait-timeout 180
                '''
            }
        }

        stage('HTTP smoke test') {
            steps {
                sh '''
                    set -eu
                    curl --fail --silent --show-error \
                      --connect-timeout 5 --max-time 15 \
                      --retry 15 --retry-connrefused --retry-delay 2 \
                      http://172.16.0.10:8081/ \
                      > todo-response.html

                    test -s todo-response.html
                    echo "PASS: Todo application returned a successful, nonempty response."
                '''
            }
        }
    }

    post {
        always {
            sh '''
                docker compose \
                  -p dotnetlab \
                  -f "$DEPLOY_DIR/docker-compose.yml" \
                  ps || true

                docker compose \
                  -p dotnetlab \
                  -f "$DEPLOY_DIR/docker-compose.yml" \
                  logs --no-color --tail=40 || true
            '''
        }
    }
}