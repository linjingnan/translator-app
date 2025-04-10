pipeline {
    agent any

    tools {
        nodejs 'Node.js'
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-southeast-2'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Install Project Dependencies and Build') {
            steps {
                    sh '''
                    npm cache clean --force
                    rm -rf node_modules
                    npm install
                    npm run build 
                    '''
                    }
        }


        stage('Check and Install AWS CLI') {
            steps {
                script {
                    def awsCliExists = sh(script: 'which aws', returnStatus: true) == 0
                    if (!awsCliExists) {
                        echo 'AWS CLI is not installed. Installing AWS CLI...'
                        sh '''
                            sudo apt-get update
                            sudo apt-get install -y awscli
                        '''
                    }
                }
            }
        }

        stage('Configure AWS CLI') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ifa-frontend-aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_DEFAULT_REGION
                        aws configure set output json
                    '''
                }
            }
        }

        stage('Deploy to S3') {
            steps {
                dir('out') {
                    sh '''
                        echo "Uploading files from out/ directory to S3..."
                        aws s3 cp . s3://ifa-frontend/ --recursive
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'aws s3 ls s3://ifa-frontend/ --recursive'
            }
        }

        
    }
}
