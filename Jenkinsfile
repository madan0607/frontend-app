pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'   // Change to your region
        S3_BUCKET = 'suhask18'   // Your target S3 bucket
        CLOUDFRONT_ID = 'ERYIS4L6QYVO4' // CloudFront distribution ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Fetching source code..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building project..."
                // Example for React/Vue/Angular frontend
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Deploy to S3') {
            steps {
                echo "Deploying to S3..."
                // Sync build output to S3 bucket
                sh '''
                aws s3 sync build/ s3://$S3_BUCKET \
                    --delete \
                    --acl public-read
                '''
            }
        }

        stage('Invalidate CloudFront Cache') {
            steps {
                echo "Invalidating CloudFront cache..."
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
            echo "❌ Deployment failed. Please check the logs."
        }
    }
}
