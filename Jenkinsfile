pipeline {
    agent {
        // Необхідно для роботи в плейграунді
        label 'agent-node-label'
    }

     environment {
        APP_NAME = 'oleksandra'
        DOCKER_IMAGE_NAME = 'tkachenko'
        // Необхідно для роботи в плейграунді
        GOCACHE="/home/jenkins/.cache/go-build/"
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Крок клонування репозиторію1
                checkout scm
            }
        }

        stage('Compile') {
            agent {
                // Використання Docker образу з підтримкою Go версії 1.21.3.
                docker {
                    image 'golang:1.21.3'
                    reuseNode true
                }
            }
            steps {
                // Компіляція проекту на мові Go
                sh "CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags '-w -s -extldflags \"-static\"' -o ${APP_NAME} ."
            }
        }

        stage('Unit Testing') {
            agent {
                // Використання Docker образу з підтримкою Go версії 1.21.3.
                docker {
                    image 'golang:1.21.3'
                    reuseNode true
                }
            }
            steps {
                script {
            // Переходимо в каталог робочого простору
            dir("${WORKSPACE}") {
                // Встановлює залежності та виконує юніт-тести
                sh 'go get -t -v ./...'
                sh 'go test -v ./...'
                 }
                }
            }
        }

        stage('Archive Artifact and Build Docker Image') {
            parallel {
                stage('Archive Artifact') {
                    steps {
                        // Створення TAR-архіву артефакту
                        sh "tar -czf ${APP_NAME}-${BUILD_NUMBER}.tar.gz ${APP_NAME}"
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        // Створення Docker образу
                        script {
                            def dockerImageName = "${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                            sh "docker build -t ${dockerImageName} --build-arg APP_NAME=${APP_NAME} ."
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            // Архівація успішна, артефакт готовий для використання та збереження
            archiveArtifacts artifacts: "${APP_NAME}-${BUILD_NUMBER}.tar.gz", onlyIfSuccessful: true
        }
        always {
            // Завершення пайплайну, можна додати додаткові кроки (наприклад, розгортання) за потребою
            echo 'Pipeline finished.'
        }
    }
}

