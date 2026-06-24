pipeline {
    agent any

    environment {
        SOLUTION = 'HeartRhythmTherapeuticSite.sln'
        PUBLISH_DIR = 'F:\\publish'
        ZIP_FILE = 'E:\\Downloads\\Songs\\HRTPL_latest_Package_22nd_June.zip'
        AZURE_WEBAPP = 'prtechnologies'
        AZURE_RG = 'PayAsYouGo-RG'
        MSBUILD = '"D:\\Program Files\\Microsoft Visual Studio\\2022\\Enterprise\\MSBuild\\Current\\Bin\\MSBuild.exe"'
        NUGET = '"C:\\Nuget\\nuget.exe"'
        ARTIFACT_ZIP  = "build-${BUILD_NUMBER}.zip"
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
                    ${MSBUILD} %SOLUTION% /p:Configuration=Release /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:PackageLocation="%WORKSPACE%\\%ARTIFACT_ZIP%"
                """
            }
        }

        stage('Package') {
            steps {
            echo "packaging from %WORKSPACE%\\ ${env.ARTIFACT_ZIP}"
                
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
                  echo "Deploying from : ${env.ZIP_FILE}"
                    bat """
                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        az account set --subscription %AZURE_SUBSCRIPTION_ID%
                      
                        az webapp deploy --resource-group %AZURE_RG% --name %AZURE_WEBAPP% --src-path %ZIP_FILE%
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
