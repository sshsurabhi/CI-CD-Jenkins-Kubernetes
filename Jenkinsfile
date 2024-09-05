pipeline {
    agent any
    stages {
        stage('Clean WorkSpace') {
            steps {
                deleteDir()
            }
        }
        stage('Checkout') {
            steps {
                git 'https://github.com/Sri-Harsha-S/jenkins-k8s-conclusion.git'
            }
        }
        stage('Build and Push') {
            steps {
                script {
                    sh 'ls -l'
                    withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_ID_PWD', usernameVariable: 'DOCKERHUB_ID', passwordVariable: 'DOCKERHUB_PWD')]){
                        dir('movie-service'){
                            sh 'docker build -t surabhiharsha5/movie-service:$BUILD_NUMBER .'
                            sh 'docker login -u $DOCKERHUB_ID -p $DOCKERHUB_PWD'
                            sh 'docker push surabhiharsha5/movie-service:$BUILD_NUMBER'
                        }
                    }
                    
                    dir('cast-service'){
                        sh 'docker build -t surabhiharsha5/cast-service:$BUILD_NUMBER .'
                        sh 'docker push surabhiharsha5/cast-service:$BUILD_NUMBER'
                    }
                    
                }
            }
        }
        stage('Update Deployment') {
            steps {
                script {
                    sh "sed -i 's|image: surabhiharsha5/movie-service:.*|image: surabhiharsha5/movie-service:${env.BUILD_NUMBER}|' k8s-manifests/movie-service-deployment.yaml"
                    sh "sed -i 's|image: surabhiharsha5/cast-service:.*|image: surabhiharsha5/cast-service:${env.BUILD_NUMBER}|' k8s-manifests/cast-service-deployment.yaml"
                }
            }
        }
        stage('Commit Changes to GitHub') {
            steps {
                script {
                    sh "git config --global user.email 'srisurabhi5@example.com'"
                    sh "git config --global user.name 'Sri-Harsha-S'"
                    def status = sh(script: 'git status --porcelain', returnStdout: true).trim()
                    if (status) {
                        withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_PAT')]){
                        sh """
                           git add .
                           git commit -m "Update image tag to ${env.BUILD_NUMBER}"
                           git push https://Sri-Harsha-S:${GITHUB_PAT}@github.com/Sri-Harsha-S/jenkins-k8s-conclusion.git master
                        """
                        }
                    }else {
                        echo "No changes to commit."
                    }
                }
            }
        }
        stage('Deployment') {
            environment
            {
            KUBECONFIG = credentials("config")
            }
                steps {
                    script {
                        //withCredentials([file(credentialsId: 'config', variable: 'KUBE_CONFIG_FILE')]) {
                        dir('k8s-manifests') {
                            sh """
                            rm -Rf .kube
                            mkdir .kube
                            ls
                            cat $KUBECONFIG > .kube/config
                            """
                            env.KUBECONFIG = '${HOME}/.kube/config'
                            sh "kubectl get namespaces"
                            sh "kubectl create namespace prod || true"
                            sh "kubectl create namespace qa || true"
                            sh "kubectl create namespace devd || true"
                            sh "kubectl create namespace staging || true"
                            sh "kubectl apply -f . -n prod"
                        }
                        //}
                    }
                }
            }
        }
    }
