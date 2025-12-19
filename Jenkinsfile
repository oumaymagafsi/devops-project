pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'oumaymagafsi/devops-project'
        DOCKER_TAG   = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '====== Checking out code from GitHub ======'
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/oumaymagafsi/devops-project.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo '====== Building application with Maven ======'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '====== Running SonarQube code analysis ======'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=devops-project \
                        -Dsonar.projectName="DevOps Project" \
                        -Dsonar.host.url=http://172.17.0.1:9000 \
                        -Dsonar.token=\$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '====== Building Docker image ======'
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                echo '====== Pushing Docker image to DockerHub ======'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '====== Deploying to Kubernetes ======'
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl set image deployment/devops-project \
                        devops-project=${DOCKER_IMAGE}:${DOCKER_TAG} -n devops
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo '====== Verifying deployment ======'
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    sh 'kubectl get pods -n devops'
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
            echo "🚀 Image deployed: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo '❌ Pipeline failed. Check Jenkins logs.'
        }
        always {
            echo '🧹 Cleaning workspace...'
            cleanWs()
        }
    }
}

