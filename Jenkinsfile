pipeline {
    agent {
        // Необхідно для роботи в плейграунді
        label 'agent-node-label'
    }

    environment {
        // Поміняйте APP_NAME та DOCKER_IMAGE_NAME на ваше імʼя та прізвище, відповідно.
        APP_NAME = 'Taras'
        DOCKER_IMAGE_NAME = 'Muzychuk'
        // Необхідно для роботи в плейграунді
        GOCACHE="/home/jenkins/.cache/go-build/"
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Крок клонування репозиторію
                 git 'https://github.com/TarasMuzychuk/jenkins-ci-lab.git'
            }
        }

        stage('Compile') {
            agent {
                // Використання Docker образу з підтримкою Go версії 1.21.3. Обовʼязково необхідно використати параметр `reuseNode true` для Docker агента для роботи в плейграунді
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
                docker {
                    image 'golang:1.21.3'
                    reuseNode true
                }
            }
            steps {
                // Виконання юніт-тестів. Команду можна знайти в Google
               sh 'go test ./...'
            }
        }

        stage('Archive Artifact and Build Docker Image') {
            parallel {
                stage('Archive Artifact') {
                    steps {
                        // Створення TAR-архіву артефакту з використанням імені додатку APP_NAME та номеру сборки BUILD_NUMBER
                        script {
                    def artifactName = "${env.APP_NAME}-${env.BUILD_NUMBER}.tar"
                    sh "tar -cvf ${artifactName} ${env.APP_NAME}"
                    archiveArtifacts artifacts: "${artifactName}", onlyIfSuccessful: true
                }
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        // Створення Docker образу з імʼям DOCKER_IMAGE_NAME і тегом BUILD_NUMBER та передача аргументу APP_NAME за допомогою флагу `--build-arg`
                        def dockerImage = "${env.DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        def buildArgs = "--build-arg APP_NAME=${env.APP_NAME}"
                        sh "docker build -t ${dockerImage} ${buildArgs} ."
                    }
                }
            }
        }
    }

    post {
        success {
            // Архівація успішна, артефакт готовий для використання та збереження
             echo 'Pipeline has finished successful. The artifact is ready for use and preservation.'
        }
        always {
            // Завершення пайплайну, можна додати додаткові кроки (наприклад, розгортання) за потребою
            echo 'Pipeline finished'
        }
    }
}
