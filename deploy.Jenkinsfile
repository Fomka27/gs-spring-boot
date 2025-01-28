pipeline {
    agent {
        label 'app-server'  // Використовуємо агента з лейблом app-server
    }

    environment {
        // Змінні для SSH-підключення
        EC2_USER = 'appuser'  // Користувач на EC2
        EC2_IP = '10.0.1.107'  // IP-адреса EC2-інстансу
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
                sshagent(['app-server']) {
                    sh '''
                       
                        // Копіюємо JAR-файл на сервер
                        scp -o StrictHostKeyChecking=no complete/target/${JAR_FILE} ${EC2_USER}@${EC2_IP}:/home/${EC2_USER}/
                        

                        // Запускаємо JAR-файл на сервері
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} "cd /home/${EC2_USER}/app && java -jar *.jar"
                    '''
                }
            }
        }
        stage('Run Script') {
            steps {
                sshagent(['app-server']) {
                    sh '''
                        # Змінюємо права доступу до файлу та запускаємо його
                        chmod +x script.sh.copy
                        ./script.sh.copy
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment successful!'  // Повідомлення про успішне завершення
        }
        failure {
            echo 'Build or deployment failed!'  // Повідомлення про помилку
        }
    }
}
