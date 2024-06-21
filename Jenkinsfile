
pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_REGION = 'us-east-1'
        EB_APPLICATION_NAME = 'Angular-todo'
        EB_ENVIRONMENT_NAME = 'Angular-todo-env'
        S3_BUCKET = 'elasticbeanstalk-us-east-1-451474896501'
        NODEJS_VERSION = '22.3.0' // Change to the desired Node.js version
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    // Install Node.js and npm
                    sh 'curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -'
                    sh 'sudo apt-get install -y nodejs'
                    // Install project dependencies
                    sh 'npm install'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    // Build the Angular project
                    sh 'npm run build --prod'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                script {
                    // Archive the build artifacts
                    sh 'tar -czf angular-app.tar.gz -C dist .'
                    archiveArtifacts artifacts: 'angular-app.tar.gz', allowEmptyArchive: true
                }
            }
        }

        stage('Deploy to AWS Elastic Beanstalk') {
            steps {
                script {
                    // Install AWS CLI
                    sh 'sudo apt-get install -y awscli'
                    // Configure AWS CLI
                    sh 'aws configure set aws_access_key_id ${AWS_ACCESS_KEY_ID}'
                    sh 'aws configure set aws_secret_access_key ${AWS_SECRET_ACCESS_KEY}'
                    sh 'aws configure set region ${AWS_REGION}'
                    // Upload artifact to S3
                    sh 'aws s3 cp angular-app.tar.gz s3://${S3_BUCKET}/angular-app-${BUILD_ID}.tar.gz'
                    // Create new Elastic Beanstalk application version
                    sh """
                    aws elasticbeanstalk create-application-version \
                        --application-name ${EB_APPLICATION_NAME} \
                        --version-label ${BUILD_ID} \
                        --source-bundle S3Bucket=${S3_BUCKET},S3Key=angular-app-${BUILD_ID}.tar.gz
                    """
                    // Update Elastic Beanstalk environment to new version
                    sh """
                    aws elasticbeanstalk update-environment \
                        --environment-name ${EB_ENVIRONMENT_NAME} \
                        --version-label ${BUILD_ID}
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
