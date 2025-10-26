pipeline {
    agent any

    triggers {
        // Poll SCM every 1 minute
        pollSCM('* * * * *')
    }

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'      // Update to your region
        S3_BUCKET = 'suhask18'                // Your S3 bucket
        CLOUDFRONT_ID = 'ERYIS4L6QYVO4'      // CloudFront distribution ID
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
            environment {
                // Load AWS credentials from Jenkins credential store
                AWS_ACCESS_KEY_ID = credentials('AKIA6AGKFTIA25H7A22S')
                AWS_SECRET_ACCESS_KEY = credentials('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')
            }
            steps {
                echo "☁️ Deploying build files to S3..."
                sh '''
                    aws s3 sync build/ s3://$S3_BUCKET \
                        --delete \
                        --acl public-read
                '''
            }
        }

        stage('Invalidate CloudFront Cache') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('AKIA6AGKFTIA25H7A22S')
                AWS_SECRET_ACCESS_KEY = credentials('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')
            }
            steps {
                echo "🚀 Invalidating CloudFront cache..."
                sh '''
                    aws cloudfront create-invalidation \
                        --distribution-id $CLOUDFRONT_ID \
                        --paths "/*"
                '''
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
