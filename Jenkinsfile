pipeline {
    agent any
    parameters {
        string(name: 'repoUrl', defaultValue: 'git@gitee.com:JornahLee/blog-jl.git', description: 'repo url')
        //repoBranch参数后续替换成git parameter不再依赖手工输入,JENKINS-46451【git parameters目前还不支持pipeline】
        string(name: 'repoBranch', defaultValue: 'refactor_v2', description: 'git branch name')
        string(name: 'dockerRegistry', defaultValue: 'http://192.168.31.31:5000', description: 'docker registry')
        string(name: 'imageName', defaultValue: 'my-image', description: 'image name')
    }
    tools {
        maven 'my maven'
        jdk 'jdk8'
    }
    stages {
        stage('fetch code') {
            steps {
                println("this is branch name : ${repoBranch}")
                git branch: params.repoBranch, credentialsId: 'jenkins', url: params.repoUrl
            }
        }
        stage('mvn build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('docker image build') {
            steps {
                script {
                    echo 'docker build'
                    docker.withRegistry(params.dockerRegistry) {
                        def image = docker.build(params.imageName)
                        image.push()
                    }

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
