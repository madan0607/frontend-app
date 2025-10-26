pipeline {
    agent any

    tools {
        nodejs 'node16' // Optional: if you configured NodeJS tool in Jenkins
    }

    triggers {
        // Poll SCM every 1 minute
        pollSCM('* * * * *')
    }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'     // Update to your AWS region
        S3_BUCKET = 'suhask18'               // Target S3 bucket
        CLOUDFRONT_ID = 'ERYIS4L6QYVO4'     // CloudFront distribution ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "⚙️ Building the project..."
                sh '''
                    npm install
                    npm run build
                '''
            }
        }

        stage('Deploy to S3') {
            steps {
                echo "☁️ Deploying to S3..."
                withAWS(credentials: 'aws-creds', region: "${AWS_DEFAULT_REGION}") {
                    sh '''
                        aws s3 sync build/ s3://$S3_BUCKET \
                            --delete \
                            --acl public-read
                    '''
                }
            }
        }

        stage('Invalidate CloudFront Cache') {
            steps {
                echo "🚀 Invalidating CloudFront cache..."
                withAWS(credentials: 'aws-creds', region: "${AWS_DEFAULT_REGION}") {
                    sh '''
                        aws cloudfront create-invalidation \
                            --distribution-id $CLOUDFRONT_ID \
                            --paths "/*"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Check build logs for details."
        }
    }
}
