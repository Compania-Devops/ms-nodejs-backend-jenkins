pipeline {
    agent {
        docker {
            image 'devops-agent:latest'
        }
    }

    environment {
        APELLIDO = "baraujo"  // lo mismo que en vars.APELLIDO
        SHORT_SHA = "${env.GIT_COMMIT[0..6]}" // equivalente al steps.short.outputs.short_sha
    }
    
    stages {
        stage('Azure Login') {
            steps {
                withCredentials([
                    string(credentialsId: 'azure-clientId',        variable: 'AZ_CLIENT_ID'),
                    string(credentialsId: 'azure-clientSecret',   variable: 'AZ_CLIENT_SECRET'),
                    string(credentialsId: 'azure-tenantId',       variable: 'AZ_TENANT_ID'),
                    string(credentialsId: 'azure-subscriptionId', variable: 'AZ_SUBSCRIPTION_ID')
                ]) {
                    sh '''
                      echo ">>> Iniciando sesión en Azure CLI..."
                      az login --service-principal \
                        --username $AZ_CLIENT_ID \
                        --password $AZ_CLIENT_SECRET \
                        --tenant   $AZ_TENANT_ID

                      echo ">>> Seleccionando subscripción..."
                      az account set --subscription $AZ_SUBSCRIPTION_ID

                      echo ">>> Sesión activa"
                      az account show
                    '''
                }
            }
        }

        stage('Hello World') {
            steps {
                sh 'ls -l'
                sh 'echo "Hello desde Alpine con Node.js 20 + Azure CLI + credenciales seguras en Jenkins!"'
                
            }
        }


        stage('Docker Login to ACR') {
            steps {
                sh '''
                  echo ">>> Haciendo login en ACR"
                  az acr login --name acr${APELLIDO}
                '''
            }
        }


        stage('Build and Push Docker Image') {
            steps {
                sh '''
                  IMAGE_NAME=acr${APELLIDO}.azurecr.io/my-nodejs-app
                  TAG=${SHORT_SHA}
                  echo ">>> Construyendo imagen $IMAGE_NAME:$TAG"
                  docker build -t $IMAGE_NAME:$TAG .
                  docker push $IMAGE_NAME:$TAG
                '''
            }
        }
    }
}
