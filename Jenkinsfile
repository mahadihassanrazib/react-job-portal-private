pipeline {
    agent any

    environment {

        IMAGE_TAG = "${env.BUILD_NUMBER}"
        PREVIOUS_TAG = "${env.BUILD_NUMBER.toInteger() - 1}"
        PROJECT_NAME = "react-job-portal-fe"
        PORT = "5173"
        ALERT_EMAIL = 'mahadihassanrazib@gmail.com'   // change to your recipient
    }

    stages {
        stage('Clone Repository') {
            steps {

                echo "Triggered by: ${env.ref}"

                echo "Clone React Job Portal Frontend from Main Branch"

                git branch: 'main',
                    credentialsId: 'REACT-JOB-PORT-REPO-CLONE',
                    url: 'https://github.com/mahadihassanrazib/react-job-portal-private.git'

                sh 'ls -lah'
            }
        }
        stage('prepare configuration file') {
            steps {
                dir('frontend') {
                    sh 'ls -lah'

                    echo "📝 Creating Project Configuration dynamically..."

                    writeFile file: 'Dockerfile', text: """
                        FROM node:24-slim AS builder

                        WORKDIR /bjit

                        COPY package*.json ./

                        RUN npm install

                        FROM node:24-alpine AS production

                        WORKDIR /bjit

                        COPY --from=builder /bjit/node_modules ./node_modules
                        COPY --from=builder /bjit/package*.json ./

                        # COPY src ./src
                        # COPY public ./public
                        # COPY index.html ./
                        # COPY vite.config.js ./
                        # COPY .env ./

                        COPY . .

                        ADD https://github.com/bjithassanrazib/three-tier-application/blob/main/README.md /bjit/KICKME.md

                        RUN adduser --disabled-password --gecos "" bjit

                        RUN chown -R bjit:bjit /bjit

                        EXPOSE ${env.PORT}

                        USER bjit

                        HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
                        CMD node -e "require('http').get('http://localhost:5000', (res) => { process.exit(res.statusCode === 200 ? 0 : 1); }).on('error', () => { process.exit(1); });"

                        CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]

                    """
                }
            }
        }
        stage('Build Docker Image...') {
            steps {
                dir('frontend') {
                    script {
                        echo "🔨 Building Docker image with Buildx..."

                        sh """
                            # Ensure buildx is available
                            docker --version

                            docker build -t ${env.PROJECT_NAME}:v${env.IMAGE_TAG} .

                            echo "✅ Docker image built successfully"

                            docker images -a
                        """
                    }
                }
            }
        }
        stage('Run our Frontend') {
            steps {
                echo "Deleting previous running container -- ${env.PROJECT_NAME}-v${env.PREVIOUS_TAG}"
                sh "docker rm -f ${env.PROJECT_NAME}-v${env.PREVIOUS_TAG}"
                sh "docker run -d -p 8081:${env.PORT} --name ${env.PROJECT_NAME}-v${env.IMAGE_TAG} ${env.PROJECT_NAME}:v${env.IMAGE_TAG}"
                sleep(10)
            }
        }
        stage('Check our Frontend Container running status log') {
            steps {
                sh "docker logs ${env.PROJECT_NAME}-v${env.IMAGE_TAG}"
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment successful! I am from success block"
        }

        failure {
            echo "❌ Deployment failed! I am from failure block"

            emailext(
                to: "${ALERT_EMAIL}",
                subject: "Jenkins Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """
                    <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> failed.</p>
                    <p>Console: <a href="${env.BUILD_URL}console">${env.BUILD_URL}console</a></p>
                    <p>The full console log is attached.</p>
                """,
                attachLog: true,     // attaches build.log
                compressLog: false   // plain text, not .gz
            )
        }

        always {
            echo "🎉 Hey! I am always block"
            // sh "rm -f ${APP_NAME}-${IMAGE_TAG}.tar || true"
            cleanWs()
        }
    }
}
