pipeline {
    agent any
    parameters {
        string(name: 'repoUrl', defaultValue: 'git@gitee.com:JornahLee/blog-jl.git', description: 'repo url')
        //repoBranch参数后续替换成git parameter不再依赖手工输入,JENKINS-46451【git parameters目前还不支持pipeline】
        string(name: 'repoBranch', defaultValue: 'refactor_v2', description: 'git branch name')
        string(name: 'dockerRegistry', defaultValue: 'http://132.232.81.70:5000', description: 'docker registry')
        string(name: 'imageName', defaultValue: 'my-image', description: 'image name')
        string(name: 'dockerRegistryCredentialId', defaultValue: 'tx-docker-registry', description: 'docker Registry Credential Id')
        string(name: 'dockerRegistryLanHost', defaultValue: '172.27.0.6:5000', description: 'docker Registry LAN host')
    }

    tools {
        maven 'my maven'
        jdk 'jdk8'
    }
    triggers {
        pollSCM('H/2 * * * *')
    }
    stages {
        stage('fetch code') {
            steps {
                println("this is branch name : ${params.repoBranch}")
                git branch: params.repoBranch, credentialsId: 'jenkins', url: params.repoUrl
                script {
                    env.imageTag = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                }
            }
        }
        stage('mvn build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('push docker image') {
            steps {
                script {
                    echo 'docker build'
                    docker.withRegistry(params.dockerRegistry, params.dockerRegistryCredentialId) {
                        def image = docker.build(params.imageName)
                        image.push(imageTag)
                    }

                }
            }
        }
        stage('start container') {
            steps {
                // 需要插件 SSH Pipeline Steps
                script {
                    withCredentials([usernamePassword(credentialsId: 'tx-docker-registry', passwordVariable: 'dockerpwd', usernameVariable: 'dockeruser')]) {
                        withCredentials([usernamePassword(credentialsId: 'tx-server-name-pwd', passwordVariable: 'password', usernameVariable: 'uname')]) {
                            def remote = [:]
                            remote.name = 'tx-server'
                            remote.host = '132.232.81.70'
                            remote.user = uname
                            remote.password = password
                            remote.allowAnyHosts = true
                            sshCommand remote: remote, command: """
containerId=`docker ps -a | grep -w ${params.imageName} | awk '{print \$1}'`
imageNameAndTag=`docker ps | grep -w ${params.imageName}  | awk '{print \\\$2}'`
if [ "\$containerId" !=  "" ] ; then
    #停掉容器
    echo 'stop container....'
    docker stop `docker ps -a | grep -w ${params.imageName}  | awk '{print \$1}'`

    #删除容器
    echo 'clean container....'
    docker rm `docker ps -a | grep -w ${params.imageName}  | awk '{print \$1}'`

    #删除镜像
    echo 'clean image ....'
    docker rmi --force "\$imageNameAndTag"
fi
"""
//                        cloud server LAN ip : 172.27.0.6
                            sshCommand remote: remote, command: "docker login -u $dockeruser -p $dockerpwd ${params.dockerRegistryLanHost}"
                            startCmd = "docker run --name ${params.imageName} -d -p 8089:8089 ${params.dockerRegistryLanHost}/${params.imageName}:${imageTag}"
                            println "start cmd : ${startCmd}"
                            sshCommand remote: remote, command: startCmd
                        }
                    }
                }
            }
        }
    }
}
