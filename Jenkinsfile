// ============================================================================
// Jenkinsfile — mcp-context-forge 部署到 腾讯云 TKE
// ============================================================================
// 使用官方镜像 ghcr.io/ibm/mcp-context-forge，kubectl apply 部署。
//
// Coding 平台内置变量（自动注入，无需配置）:
//   GIT_LOCAL_BRANCH, GIT_COMMIT, DOCKER_IMAGE_VERSION
// Coding 需手动配置:
//   环境变量 TKE_CLUSTER_CREDENTIAL_ID = <TKE凭据ID>
//   凭据管理 上传 TKE kubeconfig
// ============================================================================

pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE_VERSION', defaultValue: 'latest', description: '官方镜像 tag（latest / v1.0.0 等）')
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
                                mcpgateway=ghcr.io/ibm/mcp-context-forge:${DOCKER_IMAGE_VERSION} \
                                --namespace mcp-gateway
                            kubectl annotate deployment/mcp-context-forge-mcpgateway \
                                --namespace mcp-gateway \
                                coding.git-branch=${GIT_LOCAL_BRANCH} \
                                coding.git-commit=${GIT_COMMIT} \
                                --overwrite
                            kubectl rollout status deployment/mcp-context-forge-mcpgateway \
                                --namespace mcp-gateway --timeout=5m
                        fi
                    '''
                }
            }
        }
    }
}
