pipeline {
    agent { label 'windows-agent' } // Must run on a Windows agent

    environment {
        // Define your global variables
        SOLUTION_FILE = 'HeartRhythmTherapeuticSite.sln'
        PROJECT_FOLDER = 'HeartRhythmTherapeuticSite' // Name of the folder containing the .csproj
        AZURE_CRED_ID = 'azure-sp-credentials'
        TENANT_ID     = '369c2ef3-f556-48ad-8e21-0d526103d8bd'
        RESOURCE_GROUP= 'your-resource-group-name'
        WEB_APP_NAME  = 'your-azure-app-service-name'
    }

    stages {
        stage('Restore Dependencies') {
            steps {
                bat 'nuget restore "%SOLUTION_FILE%"'
            }
        }

        stage('Build & Package App') {
            steps {
                script {
                    // Find and invoke the configured MSBuild tool
                    def msbuildPath = tool name: 'MSBuild_VS2022', type: 'hudson.plugins.msbuild.MSBuildInstallation'
                    
                    // Build and package into a ready-to-deploy zip folder
                    bat "\"${msbuildPath}\" \"%SOLUTION_FILE%\" /p:Configuration=Release /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation=\"%WORKSPACE%\\build-output\\deploy.zip\""
                }
            }
        }

       
    }
    
    post {
        always {
            // Clean up workspace workspace files after completion
            cleanWs()
        }
    }
}
