pipeline {
    agent any

    environment {
        App_Version = "v1.0.${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Azizaman/datastore-project-devops.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh '''
                echo "-------- Building Application --------"
                mvn clean package
                echo "-------- Build Completed --------"
                '''
            }
        }

        stage('Maven Test') {
            steps {
                sh '''
                echo "-------- Running Tests --------"
                mvn test
                echo "-------- Tests Completed --------"
                '''
            }
        }

        stage('Upload Artifact to S3') {
            steps {
                sh '''
                echo "-------- Uploading Artifact to S3 --------"
                aws s3 cp target/*.jar s3://datastore-artefact-store-apps-jenkins/
                echo "-------- Upload Successful --------"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "-------- Building Docker Image --------"
                docker build -t datastore:${App_Version} .
                echo "-------- Docker Image Built --------"
                '''
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh '''
                echo "-------- Scanning Image --------"
                trivy image datastore:${App_Version}
                echo "-------- Scan Completed --------"
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                echo "-------- Tagging Image --------"
                docker tag datastore:${App_Version} 8072388539/datastore:${App_Version}
                echo "-------- Tagging Done --------"
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo "-------- Docker Login --------"
                    docker login -u $DOCKER_USER -p $DOCKER_PASS

                    echo "-------- Pushing Image --------"
                    docker push 8072388539/datastore:${App_Version}
                    echo "-------- Push Completed --------"
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                echo "-------- Cleaning Docker --------"
                docker image prune -a -f
                '''
            }
        }

        stage('Deployment Approval') {
            steps {
                input message: "Do you want to deploy?"
            }
        }

        stage('Trigger Deployment Job') {
            steps {
                build job: "KubernetesDeployment",
                parameters: [
                    string(name: "App_Name", value: "datastore-deploy"),
                    string(name: "App_Version", value: "${App_Version}")
                ]
            }
        }
    }
}
