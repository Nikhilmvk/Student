pipeline {
    agent any

    environment {
        // Paths and names
        PROJECT_FOLDER = "StudentPortal.Web-master\\StudentPortal.Web"
        CSPROJ = "StudentPortal.Web.csproj"
        PUBLISH_FOLDER = "${env.WORKSPACE}\\publish"

        // Ubuntu VM
        VM_IP = "192.168.17.134"
        VM_USER = "nikhil"
        APP_FOLDER = "/home/nikhil/StudentPortalApp"

        // SQL Server connection
        SQL_SERVER_IP = "192.168.17.1"  // Replace with your Windows SQL Server IP

        // GitHub SSH credentials ID in Jenkins
        GIT_CREDENTIALS = "github-ssh"
    }

    stages {

        stage('Clone Repository') {
            steps {
                // Use SSH key added to Jenkins credentials
                sshagent([env.GIT_CREDENTIALS]) {
                    bat 'git clone git@github.com:Nikhilmvk/Student.git'
                }
            }
        }

        stage('Check Workspace') {
            steps {
                bat 'dir "${env.WORKSPACE}" /s'
            }
        }

        stage('Publish .NET Project') {
            steps {
                bat """
                dotnet publish "${env.WORKSPACE}\\${PROJECT_FOLDER}\\${CSPROJ}" -c Release -o "${PUBLISH_FOLDER}"
                """
            }
        }

        stage('Update Connection String') {
            steps {
                // Replace "localhost" in appsettings.json with your SQL Server IP
                bat """
                powershell -Command "(Get-Content '${PUBLISH_FOLDER}\\appsettings.json') -replace 'localhost', '${SQL_SERVER_IP}' | Set-Content '${PUBLISH_FOLDER}\\appsettings.json'"
                """
            }
        }

        stage('Copy to Ubuntu VM') {
            steps {
                // Use SSH key added to Jenkins credentials
                sshagent([env.GIT_CREDENTIALS]) {
                    bat """
                    pscp -r "${PUBLISH_FOLDER}\\*" ${VM_USER}@${VM_IP}:${APP_FOLDER}\\
                    """
                }
            }
        }

        stage('Build & Run Docker on Ubuntu') {
            steps {
                sshagent([env.GIT_CREDENTIALS]) {
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

    post {
        always {
            echo 'Pipeline finished!'
        }
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
