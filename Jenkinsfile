pipeline {
    agent any

    stages {

        stage('Code') {
            steps {
                echo 'Cloning the Code'
                git branch: 'main', url: 'https://github.com/psanjayk04/DevopsProject1.git'
            }
        }

        stage('Maven Unit Test') {
            steps {
                dir('/var/lib/jenkins/workspace/pipeline') {
                    sh 'mvn test'
                }
            }
        }

        stage('Maven Build') {
            steps {
                dir('/var/lib/jenkins/workspace/pipeline') {
                    sh 'mvn clean install'
                }
            }
        }

        stage('Maven Integration Test') {
            steps {
                dir('/var/lib/jenkins/workspace/pipeline') {
                    sh 'mvn verify'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Build the image'
                sh 'docker build --no-cache -t psanjayk04/ci-pipeline:latest .'
            }
        }

        stage('Pushing the image to Dockerhub') {
            steps {
                echo 'Pushing image to Docker Hub'

                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                    // Use a proper image name + tag and push securely
                    sh "docker tag ci-pipeline:latest ${DOCKERHUB_USER}/ci-pipeline:latest"
                    // Safer login using password-stdin
                    sh "echo ${DOCKERHUB_PASS} | docker login --username ${DOCKERHUB_USER} --password-stdin"
                    sh "docker push ${DOCKERHUB_USER}/ci-pipeline:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('/var/lib/jenkins/workspace/pipeline') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                            sh 'kubectl delete -f all-pods || true'
                            sh 'kubectl apply -f deployment.yaml'
                            sh 'kubectl apply -f service.yaml'
                        }
                    }
                }
            }
        }
    }
}
