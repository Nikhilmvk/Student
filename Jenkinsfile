pipeline {
    agent any

    environment {
        PROJECT_NAME = "StudentPortal.Web"
        VM_IP = "192.168.17.134"
        VM_USER = "nikhil"
        APP_FOLDER = "/home/nikhil/StudentPortalApp"
        SQL_SERVER_IP = "192.168.17.1"  // Your Windows SQL Server IP
    }

    stages {

        stage('Publish .NET Project') {
            steps {
                bat """
                dotnet publish "${env.WORKSPACE}\\${PROJECT_NAME}\\StudentPortal.Web.csproj" -c Release -o "${env.WORKSPACE}\\publish"
                """
            }
        }

        stage('Update Connection String') {
            steps {
                bat """
                powershell -Command "(Get-Content '${env.WORKSPACE}\\publish\\appsettings.json') -replace 'localhost', '${SQL_SERVER_IP}' | Set-Content '${env.WORKSPACE}\\publish\\appsettings.json'"
                """
            }
        }

        stage('Copy to Ubuntu VM') {
            steps {
                sshagent(['ubuntu-vm-ssh']) {
                    bat """
                    pscp -r "${env.WORKSPACE}\\publish\\*" ${VM_USER}@${VM_IP}:${APP_FOLDER}\\
                    """
                }
            }
        }

        stage('Build & Run Docker on Ubuntu') {
            steps {
                sshagent(['ubuntu-vm-ssh']) {
                    bat """
                    ssh ${VM_USER}@${VM_IP} ^
                        docker build -t studentportal:latest ${APP_FOLDER} ^&^&
                        docker stop studentportal ^|^| exit 0 ^&^&
                        docker rm studentportal ^|^| exit 0 ^&^&
                        docker run -d -p 80:80 --name studentportal studentportal:latest
                    """
                }
            }
        }
    }
}
