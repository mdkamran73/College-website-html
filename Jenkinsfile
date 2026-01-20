pipeline {
    agent any

    parameters {
        string(
            name: 'ENVIRONMENT',
            defaultValue: 'dev',
            description: 'Target environment: dev | qa | prod'
        )
    }

    environment {
        DEPLOY_DIR = "/var/www/html"
        APP_REPO   = "https://github.com/mdkamran73/College-website-html.git"
        SLACK_CHANNEL = "#cricket"
    }

    stages {

        stage('Validate Environment') {
            steps {
                script {
                    if (!(params.ENVIRONMENT in ['dev', 'qa', 'prod'])) {
                        error "Invalid ENVIRONMENT value. Use dev, qa, or prod."
                    }

                    if (params.ENVIRONMENT == 'dev') {
                        env.TARGET_SERVER = "34.236.237.89"
                    } else if (params.ENVIRONMENT == 'qa') {
                        env.TARGET_SERVER = "98.86.150.192"
                    } else {
                        env.TARGET_SERVER = "13.219.233.88"
                    }

                    echo "Deploying to ${params.ENVIRONMENT.toUpperCase()} -> ${env.TARGET_SERVER}"
                }
            }
        }

        stage('Notify Deployment Started') {
            steps {
                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "#439FE0",
                    message: "🚀 *Deployment Started*\nEnvironment: *${params.ENVIRONMENT.toUpperCase()}*\nJob: ${env.JOB_NAME}\nBuild: #${env.BUILD_NUMBER}"
                )
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${APP_REPO}"
            }
        }

        stage('Approval for PROD') {
            when {
                expression { params.ENVIRONMENT == 'prod' }
            }
            steps {
                input message: 'Approve PROD deployment?'
            }
        }

        stage('Deploy Application') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh """
                    chmod 600 \$SSH_KEY
                    ssh -o StrictHostKeyChecking=no -i \$SSH_KEY \$SSH_USER@${TARGET_SERVER} 'sudo rm -rf ${DEPLOY_DIR}/*'
                    scp -o StrictHostKeyChecking=no -i \$SSH_KEY -r * \$SSH_USER@${TARGET_SERVER}:/tmp/
                    ssh -o StrictHostKeyChecking=no -i \$SSH_KEY \$SSH_USER@${TARGET_SERVER} 'sudo cp -r /tmp/* ${DEPLOY_DIR}/ && sudo systemctl restart apache2'
                    """
                }
            }
        }
    }

    post {

        success {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: "good",
                message: "✅ *Deployment Successful*\nEnvironment: *${params.ENVIRONMENT.toUpperCase()}*\nURL: http://${TARGET_SERVER}\nBuild: #${env.BUILD_NUMBER}"
            )
        }

        failure {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: "danger",
                message: "❌ *Deployment Failed*\nEnvironment: *${params.ENVIRONMENT.toUpperCase()}*\nJob: ${env.JOB_NAME}\nBuild: #${env.BUILD_NUMBER}"
            )
        }

        aborted {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: "warning",
                message: "⚠️ *Deployment Aborted*\nEnvironment: *${params.ENVIRONMENT.toUpperCase()}*\nBuild: #${env.BUILD_NUMBER}"
            )
        }
    }
}
