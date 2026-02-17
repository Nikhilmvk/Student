pipeline {
    agent {
        // Use your Ubuntu VM as the Jenkins agent
        label 'ubuntu-vm'
    }

    environment {
        // Project paths
        PROJECT_FOLDER = "StudentPortal.Web-master/StudentPortal.Web"
        CSPROJ = "StudentPortal.Web.csproj"
        PUBLISH_FOLDER = "${WORKSPACE}/publish"

        // SQL Server connection (from Windows)
        SQL_SERVER_IP = "192.168.17.1"  // Replace with your SQL Server IP

        // GitHub SSH credentials ID in Jenkins
        GIT_CREDENTIALS = "github-ssh"
    }

    stages {

        stage('Clone Repository') {
            steps {
                sshagent([env.GIT_CREDENTIALS]) {
                    sh 'git clone git@github.com:Nikhilmvk/Student.git'
                }
            }
        }

        stage('Check Workspace') {
            steps {
                sh 'ls -lR "${WORKSPACE}"'
            }
        }

        stage('Publish .NET Project') {
            steps {
                sh """
                dotnet publish "${WORKSPACE}/${PROJECT_FOLDER}/${CSPROJ}" -c Release -o "${PUBLISH_FOLDER}"
                """
            }
        }

        stage('Update Connection String') {
            steps {
                sh """
                sed -i "s/localhost/${SQL_SERVER_IP}/g" "${PUBLISH_FOLDER}/appsettings.json"
                """
            }
        }

        stage('Build & Run Docker') {
            steps {
                sh """
                docker build -t studentportal:latest ${PUBLISH_FOLDER} &&
                docker stop studentportal || true &&
                docker rm studentportal || true &&
                docker run -d -p 80:80 --name studentportal studentportal:latest
                """
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
