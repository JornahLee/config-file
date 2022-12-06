pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'docker build'
                def testImage = docker.build("test-image")
                echo 'docker push'
                testImage.push()
                echo 'Building..'
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
