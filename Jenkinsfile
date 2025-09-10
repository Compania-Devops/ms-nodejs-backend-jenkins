pipeline {
    agent {
        docker {
            image 'devops-agent:latest'
        }
    }

    environment {
        // Variables globales
        ENV = "dev" 
        API_PROVIDER_URL = "http://api.com"
        APELLIDO = "baraujo"
        SHORT_SHA = "${env.GIT_COMMIT[0..6]}"
        IMAGE_NAME = "acr${APELLIDO}.azurecr.io/my-nodejs-app"
        TAG = "${SHORT_SHA}"
        IMAGE = "${IMAGE_NAME}:${TAG}"
        RESOURCE_GROUP = "rg-cicd-terraform-app-${APELLIDO}"
        ACR_NAME = "acr${APELLIDO}"
        APP_NAME = "aca-ms-${APELLIDO}-${ENV}"
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
                        --tenant $AZ_TENANT_ID

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
                sh '''
                  ls -l
                  echo "Hello desde Alpine con Node.js 20 + Azure CLI + credenciales seguras en Jenkins!"
                '''
            }
        }

        stage('Docker Login & Build') {
            steps {
                sh '''
                  echo ">>> Haciendo login en ACR"
                  az acr login --name $ACR_NAME

                  echo ">>> Construyendo imagen $IMAGE"
                  docker build -t $IMAGE .
                  docker push $IMAGE
                '''
            }
        }

        stage('Configurar Container App & Deploy') {
            steps {
                sh '''
                  echo "Configurando credenciales del ACR para la Container App..."
                  ACR_SERVER=$(az acr show --name $ACR_NAME --resource-group $RESOURCE_GROUP --query loginServer --output tsv)
                  echo "Servidor ACR: $ACR_SERVER"

                  az containerapp registry set \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --server $ACR_SERVER \
                    --identity system

                  echo "Updating Azure Container App $APP_NAME to image $IMAGE"
                  az containerapp update \
                    --name $APP_NAME \
                    --resource-group $RESOURCE_GROUP \
                    --image $IMAGE \
                    --set-env-vars ENV=${ENV} API_PROVIDER_URL=${API_PROVIDER_URL}

                  ENDPOINT=$(az containerapp show --name $APP_NAME --resource-group $RESOURCE_GROUP --query properties.configuration.ingress.fqdn -o tsv)
                  echo "Endpoint del Container App: https://$ENDPOINT"
                '''
            }
        }
    }
}
