pipeline {
    agent any

    environment {
        DB_URL = 'mysql+pymysql://usr:pwd@host:3306/db'
        DISABLE_AUTH = true
        GOOGLE_ACCESS_KEY_ID = credentials('google-access-key-id')
    }

    stages {
        stage('Сборка') {
            steps {
                echo 'Сборка приложения...'
                sh '''
                    echo "Многострочный шаг"
                    ls -lh
                '''
                sh '''
                    echo "URL базы данных: ${DB_URL}"
                    echo "DISABLE_AUTH: ${DISABLE_AUTH}"
                    env
                '''
                echo "Запуск сборки № ${env.BUILD_NUMBER} на ${env.JENKINS_URL}"
            }
        }

        stage('Тестирование') {
            steps {
                echo 'Тестирование приложения...'
            }
        }

        stage('Деплой на стейджинг') {
            steps {
                echo 'Проверка наличия команд'
                sh 'which chmod || echo "chmod not found"'
                sh 'which ./deploy || echo "./deploy not found"'
                sh 'which ./smoke-tests || echo "./smoke-tests not found"'

                echo 'Изменение прав на выполнение скриптов'
                sh 'chmod u+x deploy smoke-tests || { echo "chmod failed"; exit 1; }'

                echo 'Деплой на стейджинг'
                sh './deploy staging || { echo "deploy failed"; exit 1; }'

                echo 'Выполнение smoke-тестов'
                sh './smoke-tests || { echo "smoke-tests failed"; exit 1; }'
            }
        }

        stage('Проверка работоспособности') {
            steps {
                input 'Отправить на продакшн?'
            }
        }

        stage('Деплой на продакшн') {
            steps {
                sh './deploy prod || { echo "deploy to prod failed"; exit 1; }'
            }
        }
    }

    post {
        always {
            script {
                node {
                    echo 'Выполняется всегда'
                }
            }
        }
        cleanup {
            script {
                node {
                    echo 'Очистка рабочей области'
                    sh 'ls -l'
                    retry(3) {
                        cleanWs()
                    }
                }
            }
        }
        success {
            script {
                node {
                    echo 'Сборка прошла успешно'
                }
            }
        }
        failure {
            script {
                node {
                    echo 'Задача провалилась'
                    mail to: 'ваша почта@gmail.com',
                         subject: "${env.JOB_NAME} — Сборка № ${env.BUILD_NUMBER} провалилась",
                         body: "Подробности: ${env.BUILD_URL}"
                }
            }
        }
        unstable {
            script {
                node {
                    echo 'Статус: нестабильный (провал тестов)'
                }
            }
        }
        changed {
            script {
                node {
                    echo 'Состояние пайплайна изменилось'
                }
            }
        }
        fixed {
            script {
                node {
                    echo 'Предыдущий запуск был неудачным, текущий — успешный'
                }
            }
        }
    }
}
