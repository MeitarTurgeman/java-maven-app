#!/usr/bin/env groovy

pipeline {
    agent any
    tools {
        maven 'maven 3.9'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app') {
            steps {
                script {
                    echo "building the application..."
                    sh 'mvn clean package'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-credential', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t meitarturgeman/demo-app:${IMAGE_NAME} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push meitarturgeman/demo-app:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh-private-key', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                            eval $(ssh-agent -s)
                            ssh-add $SSH_KEY
                            ssh-keyscan github.com >> ~/.ssh/known_hosts
                            git config --global user.email "jenkins@example.com"
                            git config --global user.name "jenkins"
                            git remote set-url origin git@github.com:MeitarTurgeman/java-maven-app.git
                            git add .
                            git commit -m "ci: version bump"
                            git push origin HEAD:jenkins-jobs
                        '''
                    }
                }
            }
        }
    }
}
