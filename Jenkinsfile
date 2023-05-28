    pipeline {
        agent any
        parameters {
            string(name: 'repoUrl', defaultValue: 'git@gitee.com:JornahLee/blog-jl.git', description: 'repo url')
            //repoBranch参数后续替换成git parameter不再依赖手工输入,JENKINS-46451【git parameters目前还不支持pipeline】
            string(name: 'repoBranch', defaultValue: 'master', description: 'git branch name')
            string(name: 'dockerRegistry', defaultValue: 'https://jornaharc.azurecr.io', description: 'docker registry')
            string(name: 'dockerRegistryPrefix', defaultValue: 'jornaharc.azurecr.io', description: 'docker registry')
            string(name: 'imageName', defaultValue: 'my-image', description: 'image name')
            string(name: 'dockerRegistryCredentialId', defaultValue: 'ACR-azure', description: 'docker Registry Credential Id')
            string(name: 'deployServerHost', defaultValue: 'az.499801.xyz', description: 'deployServerHost')
            string(name: 'deployServerCredentialId', defaultValue: 'az-jornah', description: 'deployServerCredentialId')

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
                        withCredentials([usernamePassword(credentialsId: params.dockerRegistryCredentialId, passwordVariable: 'dockerpwd', usernameVariable: 'dockeruser')]) {
                            withCredentials([usernamePassword(credentialsId: params.deployServerCredentialId, passwordVariable: 'password', usernameVariable: 'uname')]) {
                                def remote = [:]
                                remote.name = 'deploy Server'
                                remote.host = params.deployServerHost
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
                                sshCommand remote: remote, command: "docker login -u $dockeruser -p $dockerpwd ${params.dockerRegistry}"
                                startCmd = "docker run --name ${params.imageName} -d -p 8089:8089 ${params.dockerRegistryPrefix}/${params.imageName}:${imageTag}"
                                println "start cmd : ${startCmd}"
                                sshCommand remote: remote, command: startCmd
                            }
                        }
                    }
                }
            }
        }
    }
