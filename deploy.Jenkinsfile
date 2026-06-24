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
        stage('Publish') {
            steps {
                bat """
                ${MSBUILD} HeartRhythmTherapeuticSite\\HeartRhythmTherapeuticSite.csproj /p:Configuration=Release /p:DeployOnBuild=true /p:WebPublishMethod=FileSystem /p:PublishUrl=publish /p:DeleteExistingFiles=True
                """
            }
        }
        stage('Zip') {
            steps {
                bat """
                    powershell Compress-Archive -Path publish\\* -DestinationPath deploy.zip -Force
                """
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
                      
                        az webapp deploy --resource-group %AZURE_RG% --name %AZURE_WEBAPP% --src-path deploy.zip --type zip
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployed successfully to Azure App Service: ${env.AZURE_WEBAPP} from ${env.ARTIFACT_ZIP}"
        }
        failure {
            echo 'Deployment failed'
        }
    }
}
