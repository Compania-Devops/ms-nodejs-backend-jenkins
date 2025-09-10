pipeline {
    agent {
        docker { image 'devops-agent:latest' }
    }

    environment {
        APELLIDO = "baraujo"
        SHORT_SHA = "${env.GIT_COMMIT[0..6]}"
        IMAGE_NAME = "acr${APELLIDO}.azurecr.io/my-nodejs-app"
        TAG = "${SHORT_SHA}"
        IMAGE = "${IMAGE_NAME}:${TAG}"
        RESOURCE_GROUP = "rg-cicd-terraform-app-${APELLIDO}"
        ACR_NAME = "acr${APELLIDO}"
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
                      echo ">>> Azure login..."
                      az login --service-principal \
                        --username $AZ_CLIENT_ID \
                        --password $AZ_CLIENT_SECRET \
                        --tenant $AZ_TENANT_ID
                      az account set --subscription $AZ_SUBSCRIPTION_ID
                      az account show
                    '''
                }
            }
        }
        
        stage('Hello world') {
            steps {
                script { // Declarar más variables de entorno
                    env.VARIABLE = "demo123"
                }

                sh '''
                  echo ">>> Impresión Hello world"
                  echo "Hello world"
                  echo "Variable declarada en script: $VARIABLE"
                  echo "Variable declarada en environment: $APELLIDO"
                '''
            }
        }
    }
}
