pipeline {
    agent { label 'app' }

    parameters {
        string(name: 'REMOTE_USER', defaultValue: 'ubuntu', description: 'Remote user for SSH')
        string(name: 'REMOTE_HOST', defaultValue: '35.157.11.66', description: 'Remote host for SSH')
        string(name: 'DST_FOLDER', defaultValue: '/home/ubuntu/app', description: 'Destination folder on remote server')
    }


    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'git@github.com:Fomka27/gs-spring-boot.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Prepare Remote Directory') {
            steps {
                sshagent(credentials: ['ifomenko']) {
                    sh "ssh -o StrictHostKeyChecking=no ${params.REMOTE_USER}@${params.REMOTE_HOST} \"mkdir -p ${params.DST_FOLDER}\""
                }
            }
        }

        stage('Transfer Files') {
            steps {
                sshagent(credentials: ['ifomenko']) {
                    sh "scp -o StrictHostKeyChecking=no target/demo-0.0.1-SNAPSHOT.jar ${params.REMOTE_USER}@${params.REMOTE_HOST}:${params.DST_FOLDER}"
                }
            }
        }

        stage('Run on Remote Server') {
            steps {
                sshagent(credentials: ['ifomenko']) {
                    sh "ssh -o StrictHostKeyChecking=no ${params.REMOTE_USER}@${params.REMOTE_HOST} \"cd ${params.DST_FOLDER} && nohup java -jar demo-0.0.1-SNAPSHOT.jar > app.log 2>&1 &\""
                }
            }
        }
    }
}
