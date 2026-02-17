pipeline {
    agent any

    environment {
        PROJECT_NAME = "StudentPortal.Web"
        GIT_REPO = "https://github.com/Nikhilmvk/Student.git"
        VM_IP = "192.168.17.134"
        VM_USER = "nikhil"
        APP_FOLDER = "/home/nikhil/StudentPortalApp"
        SQL_SERVER_IP = "192.168.17.1"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git url: "$GIT_REPO", branch: 'main'
            }
        }

        stage('Publish .NET Project') {
            steps {
                bat """
                dotnet publish "%WORKSPACE%\\$PROJECT_NAME\\$PROJECT_NAME.csproj" -c Release -o "%WORKSPACE%\\publish"
                """
            }
        }

        stage('Update Connection String') {
            steps {
                bat """
                powershell -Command "(Get-Content %WORKSPACE%\\publish\\appsettings.json) -replace 'localhost', '$SQL_SERVER_IP' | Set-Content %WORKSPACE%\\publish\\appsettings.json"
                """
            }
        }

        stage('Copy to Ubuntu VM') {
            steps {
                sshagent(['ubuntu-vm-ssh']) {
                    bat """
                    pscp -r "%WORKSPACE%\\publish\\*" %VM_USER%@%VM_IP%:%APP_FOLDER%\\
                    """
                }
            }
        }

        stage('Build & Run Docker on Ubuntu') {
            steps {
                sshagent(['ubuntu-vm-ssh']) {
                    bat """
                    ssh %VM_USER%@%VM_IP% ^
                        docker build -t studentportal:latest %APP_FOLDER% ^&^&
                        docker stop studentportal ^|^| exit 0 ^&^&
                        docker rm studentportal ^|^| exit 0 ^&^&
                        docker run -d -p 80:80 --name studentportal studentportal:latest
                    """
                }
            }
        }
    }
}
