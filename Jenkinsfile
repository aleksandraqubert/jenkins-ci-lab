pipeline {
    agent {
        // Необхідно для роботи в плейграунді
        label 'agent-node-label'
    }

    environment {
        // Поміняйте APP_NAME та DOCKER_IMAGE_NAME на ваше імʼя та прізвище, відповідно.
        APP_NAME = 'oleksandra'
        DOCKER_IMAGE_NAME = 'tkachenko'
        // Необхідно для роботи в плейграунді
        GOCACHE="/home/jenkins/.cache/go-build/"
    }

stages {
    stage('Clone Repository') {
        steps {
            // Крок клонування репозиторію
            checkout scm
        }
    }
    // Додаткові етапи ...
}


        stage('Compile') {
            agent {
                // Використання Docker образу з підтримкою Go версії 1.21.3. Обовʼязково необхідно використати параметр `reuseNode true` для Docker агента для роботи в плейграунді
                // TODO: ваш код
                   docker {
                    image 'golang:1.21.3'
                    reuseNode true
                }
            }
            steps {
                // Компіляція проекту на мові Go. Всі ці флаги необхідні для запуску на пустій файловій системі образу scratch :)
                sh "CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags '-w -s -extldflags \"-static\"' -o ${APP_NAME} ."
            }
        }

        stage('Unit Testing') {
            agent {
                // Використання Docker образу з підтримкою Go версії 1.21.3. Обовʼязково необхідно використати параметр `reuseNode true` для Docker агента для роботи в плейграунді
                // TODO: ваш код
                 docker {
                    image 'golang:1.21.3'
                    reuseNode true
                }
            }
            steps {
                // Виконання юніт-тестів. Команду можна знайти в Google
                // TODO: ваш код
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
                        // Створення TAR-архіву артефакту з використанням імені додатку APP_NAME та номеру сборки BUILD_NUMBER
                        // TODO: ваш код
                         script {
                            sh "tar -czf ${APP_NAME}-${BUILD_NUMBER}.tar.gz ${APP_NAME}"
                        }
                       
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        // Створення Docker образу з імʼям DOCKER_IMAGE_NAME і тегом BUILD_NUMBER та передача аргументу APP_NAME за допомогою флагу `--build-arg`
                        // TODO: ваш код
                          script {
                            def dockerImageName = "${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                            sh "docker build -t ${dockerImageName} --build-arg APP_NAME=${APP_NAME} ."
                        }
                    }
                }
            }
        }
    

    post {
        success {
            // Архівація успішна, артефакт готовий для використання та збереження
            // TODO: ваш код
            archiveArtifacts artifacts: "${APP_NAME}-${BUILD_NUMBER}.tar.gz", onlyIfSuccessful: true
            echo "Pipeline finished successfully"
        }
        always {
            // Завершення пайплайну, можна додати додаткові кроки (наприклад, розгортання) за потребою
            echo 'Pipeline finished'
        }
    }
}
