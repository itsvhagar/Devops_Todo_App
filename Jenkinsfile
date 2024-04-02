#!/usr/bin/env groovy

import groovy.transform.Field

@Field
String DOCKER_USER_REF = 'anhviet_docker'
@Field
String SSH_ID_REF = 'ssh-credentials-id'

pipeline {
    agent any

    tools {
        dockerTool 'docker'
    }

    stages {
        stage("build and test") {
            steps {
                sh "ls -la"
                sh "docker build -t itsanhviet/mgm_devops_1:latest ."
            }
        }
        stage("Docker login and push docker image") {
            steps {
                withBuildConfiguration {
                    echo repository_password
                    sh "docker login --username ${repository_username} --password ${repository_password}"
                    sh "docker push itsanhviet/mgm_devops_1:latest"
                }
            }
        }
        stage("deploy") {
            steps {
                withBuildConfiguration {
                    sshagent(credentials: [SSH_ID_REF]) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no root@ec2-18-143-167-76.ap-southeast-1.compute.amazonaws.com "docker run --detach --name av_mgm_devops_1 -p 4d008:8000 itsanhviet/mgm_devops_1:latest"
                        '''
                    }
                }
            }
        }
    }
}

void withBuildConfiguration(Closure body) {
    withCredentials([usernamePassword(credentialsId: DOCKER_USER_REF, usernameVariable: 'repository_username', passwordVariable: 'repository_password')]) {
        body()
    }
}