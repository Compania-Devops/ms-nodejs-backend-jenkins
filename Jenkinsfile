pipeline {
    agent {
        docker {
            image 'devops-agent:latest'
        }
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
    }
}
