pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

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
                dir('bin/publish') {
                    deleteDir()
                }
                bat 'dotnet publish Target.csproj -c Release --no-build -o bin/publish'
                archiveArtifacts artifacts: 'bin/publish/**'
            }
        }

        stage('Deploy to Azure') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([file(
                    credentialsId: 'azure-target',
                    variable: 'AZURE_CREDENTIALS_FILE'
                )]) {
                    withEnv(["AZURE_CONFIG_DIR=${pwd(tmp: true)}/azure"]) {
                        powershell '''
                            $ErrorActionPreference = 'Stop'
                            $az = Join-Path $env:ProgramFiles 'Microsoft SDKs/Azure/CLI2/wbin/az.cmd'
                            $credential = Get-Content -LiteralPath $env:AZURE_CREDENTIALS_FILE -Raw | ConvertFrom-Json

                            Compress-Archive -Path 'bin/publish/*' -DestinationPath 'bin/target.zip' -Force

                            try {
                                & $az login --service-principal --username $credential.appId "--password=$($credential.password)" --tenant $credential.tenant --output none
                                if ($LASTEXITCODE -ne 0) {
                                    throw 'Azure login failed.'
                                }

                                & $az webapp deploy --subscription 'abf11666-596d-46fc-b18a-7ccd68a0fa20' --resource-group 'TargetProject' --name 'Target' --src-path 'bin/target.zip' --type zip --output none
                                if ($LASTEXITCODE -ne 0) {
                                    throw 'Azure deployment failed.'
                                }
                            }
                            finally {
                                & $az logout --output none
                            }
                        '''
                    }
                }
            }
        }
    }
}
