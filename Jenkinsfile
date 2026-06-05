// ============================================================================
// Jenkinsfile — mcp-context-forge 部署到 腾讯云 TKE
// ============================================================================
// 使用官方镜像 ghcr.io/ibm/mcp-context-forge，kubectl apply 部署。
//
// Coding DevOps: 凭据管理中添加 TKE 集群凭据，凭据 ID 填入环境变量即可
// ============================================================================

pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: '镜像 tag')
        booleanParam(name: 'DRY_RUN', defaultValue: false, description: '仅校验，不实际部署')
    }

    stages {
        stage('拉取代码') {
            steps {
                checkout scm
            }
        }

        stage('部署到 TKE') {
            steps {
                withCredentials([file(credentialsId: env.TKE_CLUSTER_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                    sh '''
                        # 确保 namespace 存在
                        kubectl get namespace mcp-gateway 2>/dev/null || kubectl create namespace mcp-gateway

                        # 部署
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
