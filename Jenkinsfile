pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        IMAGE_NAME = 'springboot'
        IMAGE_TAG = 'latest'
        ACR_NAME = 'bootcampacr1234'  // without .azurecr.io
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        FULL_IMAGE_NAME = "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        TENANT_ID = 'bf744b93-bf76-404b-b4f7-2d85d553323d'
        RESOURCE_GROUP = 'bootcamp-rg'
        CLUSTER_NAME = 'bootcamp-aks-cluster'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Kareemujahid/spring-petclinic.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --format table --exit-code 0 --severity HIGH,CRITICAL .'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build("$IMAGE_NAME:$IMAGE_TAG")
                }
            }
        }

        stage('Login to ACR') {
            steps {
                withCredentials([
    usernamePassword(credentialsId: 'azurespn', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD'),
    string(credentialsId: 'tenant-id', variable: 'TENANT_ID'),
    string(credentialsId: 'acr-name', variable: 'ACR_NAME')
]) {
                    sh '''
                    az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                    az acr login --name $ACR_NAME
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    sh "docker tag $IMAGE_NAME:$IMAGE_TAG $FULL_IMAGE_NAME"
                    sh "docker push $FULL_IMAGE_NAME"
                }
            }
        }
    }
}
