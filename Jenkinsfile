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
                sh 'mvn test'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Maven Integration Test') {
            steps {
                sh 'mvn verify'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build --no-cache -t psanjayk04/ci-pipeline:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_PASS'
                )]) {

                    sh '''
                    echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                    docker push $DOCKERHUB_USER/ci-pipeline:latest
                    '''
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
