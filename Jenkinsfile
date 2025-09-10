pipeline {
    agent {
        docker {
            image 'devops-agent:latest'
        }
    }

    environment {
        ENV = "dev" 
        API_PROVIDER_URL = "http://api.com"
        APELLIDO = "baraujo"  // lo mismo que en APELLIDO
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
        
        stage('Establecer variables') {
            steps {
                sh '''
                  IMAGE_NAME=acr${APELLIDO}.azurecr.io/my-nodejs-app
                  TAG=${steps.short.outputs.short_sha}
                  APP_NAME=aca-ms-${APELLIDO}-${ENV}
                  RESOURCE_GROUP=rg-cicd-terraform-app-${APELLIDO}
                  ACR_NAME=acr${APELLIDO}

                  echo "IMAGE=$IMAGE_NAME:$TAG" >> $WORKSPACE/.env
                  echo "APP_NAME=$APP_NAME" >> $WORKSPACE/.env
                  echo "RESOURCE_GROUP=$RESOURCE_GROUP" >> $WORKSPACE/.env
                  echo "ACR_NAME=$ACR_NAME" >> $WORKSPACE/.env
                '''
            }
        }

        stage('Configurar ACR credentials para Container App') {
            steps {
                sh '''
                  set -a && source $WORKSPACE/.env && set +a

                  echo "Configurando credenciales del ACR para la Container App..."
                  ACR_SERVER=$(az acr show --name $ACR_NAME --resource-group $RESOURCE_GROUP --query loginServer --output tsv)
                  echo "Servidor ACR: $ACR_SERVER"
                  
                  az containerapp registry set \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --server $ACR_SERVER \
                    --identity system
                '''
            }
        }

        stage('Deploy a Azure Container App') {
            steps {
                sh '''
                  set -a && source $WORKSPACE/.env && set +a

                  echo "Updating Azure Container App $APP_NAME to image $IMAGE"
                  az containerapp update \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --image $IMAGE \
                    --set-env-vars ENV=${ENV} API_PROVIDER_URL=${API_PROVIDER_URL}
                '''
            }
        }

        stage('Imprimir endpoint del Container App') {
            steps {
                sh '''
                  set -a && source $WORKSPACE/.env && set +a

                  ENDPOINT=$(az containerapp show --name $APP_NAME --resource-group $RESOURCE_GROUP --query properties.configuration.ingress.fqdn -o tsv)
                  echo "Endpoint del Container App: https://$ENDPOINT"
                '''
            }
        }
    }
}
