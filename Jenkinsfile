pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script{
                    echo 'docker build'
                    docker.withRegistry('http://192.168.31.31:5000')
                    def testImage = docker.build("test-image")
                    echo 'docker push'
                    testImage.push()
                    echo 'Building..'
                }
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
