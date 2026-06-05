pipeline {
    agent any

    environment {
        FULL_IMAGE = "${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_VERSION}"
    }

    stages {
        stage('构建并推送镜像') {
            steps {
                script {
                    docker.withRegistry("https://${CODING_DOCKER_REG_HOST}", "${TCR_PUSH_CREDENTIAL_ID}") {
                        def img = docker.build("${FULL_IMAGE}", "-f Containerfile ${DOCKER_BUILD_CONTEXT}")
                        img.push()
                    }
                }
            }
        }

        stage('部署到 TKE') {
            steps {
                script {
                    withKubeConfig([credentialsId: "${TKE_CLUSTER_CREDENTIAL_ID}"]) {
                        sh '''
                            sed -i "s|image:.*|image: ${FULL_IMAGE}|" k8s/deployment.yaml
                            kubectl apply -f k8s/
                            kubectl rollout status deployment/mcp-context-forge-mcpgateway \
                                --namespace mcp-gateway --timeout=5m
                        '''
                    }
                }
            }
        }
    }
}
