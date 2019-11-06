pipeline {
    agent any
        
    options {
        timestamps()
    }

    triggers { pollSCM('H */5 * * *') }
    
    stages {
        stage('test') {
            environment { HOME="." }
            steps {
                script {
                    def myTestContainer = docker.image('node:13')
                    myTestContainer.pull()
                    myTestContainer.inside {
                        sh 'npm install --only=dev'
                        sh 'npm test'
                    }
                }
            }
        }
        stage('test with a db') {
            environment { HOME="." }
            steps {
                script {
                    def mysql = docker.image('mysql').run("-e MYSQL_ALLOW_EMPTY_PASSWORD=yes") 
                    def myTestContainer = docker.image('node:13')
                    myTestContainer.pull()
                    myTestContainer.inside("--link ${mysql.id}:mysql") { // using linking, mysql will be available at host: mysql, port: 3306
                        sh 'npm install --only=dev' 
                        sh 'npm test'                     
                    }                                   
                    mysql.stop()
                }
            }
        }
        stage('docker build/push') {     
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        def app = docker.build("massimilianovallascas/docker-nodejs-demo:${commit_id}", '.').push()
                    }   
                }
            }                 
        }     
    }

    post {
        failure {
            // notify users when the Pipeline fails
            mail to: 'massimiliano.vallascas@gmail.com',
            subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
            body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}
