pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: '镜像 tag')
        booleanParam(name: 'DRY_RUN', defaultValue: false, description: '仅校验，不实际部署')
    }

    stages {
        stage('检查文件') {
            steps {
                sh '''
                    echo "=== 当前目录 ==="
                    pwd
                    echo ""
                    echo "=== 根目录文件 ==="
                    ls -la
                    echo ""
                    echo "=== k8s 目录 ==="
                    ls -la k8s/ 2>/dev/null || echo "k8s/ 目录不存在!"
                '''
            }
        }

        stage('部署到 TKE') {
            steps {
                withCredentials([file(credentialsId: env.TKE_CLUSTER_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl get namespace mcp-gateway 2>/dev/null || kubectl create namespace mcp-gateway

                        if [ "${DRY_RUN}" = "true" ]; then
                            kubectl apply -f k8s/ --dry-run=client
                        else
                            kubectl apply -f k8s/
                            kubectl set image deployment/mcp-context-forge-mcpgateway \
                                mcpgateway=ghcr.io/ibm/mcp-context-forge:${IMAGE_TAG} \
                                --namespace mcp-gateway
                            kubectl rollout status deployment/mcp-context-forge-mcpgateway \
                                --namespace mcp-gateway --timeout=5m
                        fi
                    '''
                }
            }
        }
    }
}
