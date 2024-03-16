pipeline {
    agent { label "J-Agent" }
    environment {
        APP_NAME = "register-app-pipeline"
        IMAGE_TAG = "latest" // Define IMAGE_TAG or ensure it's passed from a previous step or trigger
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/leketech/gitops-register-app',
                        credentialsId: 'github'
                    ]]
                ])
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                script {
                    sh """
                       cat deployment.yaml
                       sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                       cat deployment.yaml
                    """
                }
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                           git config --global user.name "leketech"
                           git config --global user.email "lewis.leke@icloud.com"
                           git add deployment.yaml
                           git commit -m "Updated Deployment Manifest"
                           // Ensure you are on the correct branch and it exists
                           git fetch origin
                           git checkout main
                           git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/leketech/gitops-register-app main
                        """
                    }
                }
            }
        }
    }
}
