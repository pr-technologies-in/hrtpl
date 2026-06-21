pipeline {
    agent { label 'windows' }

    environment {
        SOLUTION = 'HeartRhythmTherapeuticSite.sln'
        CONFIGURATION = 'Release'
        PLATFORM = 'Any CPU'
        MSBUILD = '"D:\\Program Files\\Microsoft Visual Studio\\2022\\Enterprise\\MSBuild\\Current\\Bin\\MSBuild.exe"'
        NUGET = '"C:\\Nuget\\nuget.exe"'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore NuGet Packages') {
            steps {
                bat """
                    ${NUGET} restore ${SOLUTION}
                """
            }
        }

        stage('Build') {
            steps {
                bat """
                    ${MSBUILD} ${SOLUTION} ^
                        /p:Configuration=${CONFIGURATION} ^
                        /p:Platform="${PLATFORM}" ^
                        /t:Rebuild ^
                        /m
                """
            }
        }
    }
}
