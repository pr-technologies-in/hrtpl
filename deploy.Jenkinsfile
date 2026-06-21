pipeline {
    agent any

    environment {
        SOLUTION = 'HeartRhythmTherapeuticSite.sln'
        PUBLISH_DIR = 'F:\\publish'
        ZIP_FILE = 'HeartRhythmTherapeuticSite.zip'
        AZURE_WEBAPP = 'prtechnologies-a5abbmaxagbpg3br.centralindia-01'
        AZURE_RG = 'PayAsYouGo-RG'
        MSBUILD = '"D:\\Program Files\\Microsoft Visual Studio\\2022\\Enterprise\\MSBuild\\Current\\Bin\\MSBuild.exe"'
        NUGET = '"C:\\Nuget\\nuget.exe"'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat """
                    ${MSBUILD} %SOLUTION% /p:Configuration=Release /p:DeployOnBuild=true /p:PublishUrl=%PUBLISH_DIR% /p:WebPublishMethod=FileSystem /p:DeleteExistingFiles=True /t:Rebuild /m
                """
            }
        }

        stage('Package') {
            steps {
                bat """
                    powershell Compress-Archive -Path %PUBLISH_DIR%\\* -DestinationPath %ZIP_FILE% -Force
                """
                archiveArtifacts artifacts: ZIP_FILE, fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: 'azure-service-principal',
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                    clientIdVariable: 'AZURE_CLIENT_ID',
                    clientSecretVariable: 'AZURE_CLIENT_SECRET',
                    tenantIdVariable: 'AZURE_TENANT_ID'
                )]) {
                    bat """
                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        az account set --subscription %AZURE_SUBSCRIPTION_ID%

                        az webapp deploy ^
                          --resource-group %AZURE_RG% ^
                          --name %AZURE_WEBAPP% ^
                          --src-path %ZIP_FILE% ^
                          --type zip
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployed successfully to Azure App Service: ${env.AZURE_WEBAPP}"
        }
        failure {
            echo 'Deployment failed'
        }
    }
}
