pipeline {
    agent any

    stages {
        stage('Restore') {
            steps {
                bat 'dotnet restore Target.csproj'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build Target.csproj -c Release --no-restore'
            }
        }

        stage('Publish') {
            steps {
                bat 'dotnet publish Target.csproj -c Release --no-build -o bin/publish'
                archiveArtifacts artifacts: 'bin/publish/**'
            }
        }
    }
}
