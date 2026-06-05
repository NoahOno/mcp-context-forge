pipeline {
    agent any

    stages {
        stage('拉取代码') {
            steps {
                checkout scm
            }
        }

        stage('构建镜像') {
            steps {
                script {
                    docker.withRegistry("https://${env.CODING_DOCKER_REG_HOST}", "${env.TCR_PUSH_CREDENTIAL_ID}") {
                        docker.build("${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_VERSION}", "-f Containerfile ${env.DOCKER_BUILD_CONTEXT}")
                    }
                }
            }
        }

        stage('推送制品仓库') {
            steps {
                script {
                    docker.withRegistry("https://${env.CODING_DOCKER_REG_HOST}", "${env.TCR_PUSH_CREDENTIAL_ID}") {
                        docker.image("${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_VERSION}").push()
                    }
                }
            }
        }

        stage('部署到 TKE') {
            steps {
                script {
                    withKubeConfig([credentialsId: "${env.TKE_CLUSTER_CREDENTIAL_ID}"]) {
                        sh """
                            sed -i 's#\\(.*\\)${TCR_REPOSITORY_NAME}:.*#\\1${TCR_REPOSITORY_NAME}:${DOCKER_IMAGE_VERSION}#' k8s/deployment.yaml
                            kubectl apply -f k8s/
                            kubectl rollout status deployment/mcp-context-forge-mcpgateway \\
                                --namespace mcp-gateway --timeout=5m
                        """
                    }
                }
            }
        }
    }
}
