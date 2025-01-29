pipeline {
    agent {
        label 'app-server'  // Використовуємо агента з лейблом app-server
    }

    environment {
        // Змінні для SSH-підключення
        EC2_USER = 'ubuntu'  // Користувач на EC2
        EC2_IP = '10.0.1.57'  // IP-адреса EC2-інстансу
        JAR_FILE = 'spring-boot-complete-0.0.1-SNAPSHOT.jar'  // Назва JAR-файлу
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'git@github.com:Fomka27/gs-spring-boot.git', branch: 'main', credentialsId: 'app-server', changelog: false, poll: false
  // Клонуємо репозиторій
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -f complete/pom.xml'  // Збираємо проєкт з використанням complete/pom.xml
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['ifomenko']) {
                    sh '''
                        # Копіюємо JAR-файл на сервер
                        scp -o StrictHostKeyChecking=no complete/target/${JAR_FILE} ${EC2_USER}@${EC2_IP}:/home/${EC2_USER}/app
                        
                        # Запускаємо JAR-файл на сервері
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} "cd /home/${EC2_USER}/app && nohup java -jar *.jar > app.log 2>&1 & exit"
                    '''
                }
            }
        }
        stage('Run Script') {
            steps {
                sshagent(['ifomenko']) {
                    sh '''
                        # Надаємо права на виконання скрипту
                        chmod +x script.sh.copy
                        
                        # Виконуємо скрипт
                        ./script.sh.copy
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment successful!'
            telegramSend(
                message: "✅ Build and Deployment Successful for ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                chatId: -4724473433 // Вкажіть ваш Chat ID
            )
        }
        failure {
            echo 'Build or deployment failed!'
            telegramSend(
                message: "❌ Build or Deployment Failed for ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                chatId: -4724473433
            )
        }
    }
}
