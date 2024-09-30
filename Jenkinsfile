pipeline {
    agent any

    environment {
        // Existing environment variables
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_ACCOUNT_ID = credentials('ACCOUNT_ID')
        AWS_ECR_REPO_NAME = credentials('ECR_REPO_WEBAPP')
        AWS_DEFAULT_REGION = 'us-east-1'
        ORGANIZATION_NAME = "fleetman-k8s-ci"
        SERVICE_NAME = "fleetman-webapp"
            
        REPOSITORY_TAG = "${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
        
        // Add Maven home environment variable
        M2_HOME = tool 'Maven'
    }

    stages {
        stage('Preparation') {
            steps {
                cleanWs()
                git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
            }
        }

        stage('Build') {
            steps {
                script {
                    // Use the full path to Maven executable
                    def mvnHome = tool 'Maven'
                    if (isUnix()) {
                        sh "'${mvnHome}/bin/mvn' clean package"
                    } else {
                        bat(/"${mvnHome}\bin\mvn" clean package/)
                    }
                }
            }
        }

        // Other stages remain the same

        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "fleetman-webapp"
                GIT_ORG_NAME = "darshantech5"
            }
            steps {
              dir('manifests') {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "darshansathish5@gmail.com"
                        git config user.name "DarshanS007"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        echo $BUILD_NUMBER
                        imageTag=$(grep -oP '(?<=fleetman-webapp:)[^ ]+' deploy.yaml)
                        echo $imageTag
                        sed -i "s/${AWS_ECR_REPO_NAME}:${imageTag}/${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}/" deploy.yaml
                        git add deploy.yaml
                        git commit -m "Update deployment Image to version ${BUILD_NUMBER}"
                        
                        git pull origin master --rebase
                        
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_ORG_NAME}/${GIT_REPO_NAME} HEAD:master --force
                    '''
                }
              }
            }
        }
    }
}
