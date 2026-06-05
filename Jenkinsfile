// ============================================================================
// Jenkinsfile — mcp-context-forge 部署到 腾讯云 TKE
// ============================================================================
// 使用官方镜像 ghcr.io/ibm/mcp-context-forge，kubectl apply 部署。
//
// Coding DevOps 凭据管理 → 新增凭据:
//   凭据ID: tke-kubeconfig
//   类型:   上传文件
//   内容:   TKE 集群的 kubeconfig 文件
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
                withCredentials([file(credentialsId: 'tke-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        # 确保 namespace 存在
                        kubectl get namespace mcp-gateway 2>/dev/null || kubectl create namespace mcp-gateway

                        # 更新镜像 tag
                        cd k8s
                        sed -i "s|image: ghcr.io/ibm/mcp-context-forge:.*|image: ghcr.io/ibm/mcp-context-forge:${IMAGE_TAG}|" deployment.yaml

                        # 部署
                        if [ "${DRY_RUN}" = "true" ]; then
                            kubectl apply -f . --dry-run=client
                        else
                            kubectl apply -f .
                            kubectl rollout status deployment/mcp-context-forge-mcpgateway \
                                --namespace mcp-gateway --timeout=5m
                        fi
                    '''
                }
            }
        }
    }
}
