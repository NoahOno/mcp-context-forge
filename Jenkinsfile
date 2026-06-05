// ============================================================================
// Jenkinsfile — mcp-context-forge 部署到 腾讯云 TKE
// ============================================================================
// 使用官方镜像 ghcr.io/ibm/mcp-context-forge，通过 Helm 部署到已有 TKE 集群。
// PostgreSQL、Redis、PgBouncer 由 Helm chart 内置部署。
//
// Jenkins 需要配置 Credential:
//   tke-kubeconfig : Secret file 类型 → TKE 集群的 kubeconfig 文件
// ============================================================================

pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: '镜像 tag')
        string(name: 'HELM_RELEASE', defaultValue: 'mcp-context-forge', description: 'Helm release 名称')
        string(name: 'TKE_NAMESPACE', defaultValue: 'mcp-gateway', description: '部署的目标 namespace')
        booleanParam(name: 'DRY_RUN', defaultValue: false, description: '仅校验，不实际部署')
    }

    environment {
        CHART_PATH = 'charts/mcp-stack'
        IMAGE_REPO = 'ghcr.io/ibm/mcp-context-forge'
    }

    stages {
        stage('拉取代码') {
            steps {
                checkout scm
            }
        }

        stage('部署到 TKE') {
            steps {
                withKubeCredentials([file(credentialsId: 'tke-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        # 确保 namespace 存在
                        kubectl get namespace ${TKE_NAMESPACE} 2>/dev/null || kubectl create namespace ${TKE_NAMESPACE}

                        # 判断是首次安装还是升级
                        if helm list -n ${TKE_NAMESPACE} -q | grep -q "^${HELM_RELEASE}$"; then
                            ACTION=upgrade
                        else
                            ACTION=install
                        fi

                        # dry-run 模式
                        DRY_FLAG=""
                        if [ "${DRY_RUN}" = "true" ]; then
                            DRY_FLAG="--dry-run --debug"
                        fi

                        # 执行 Helm 部署
                        helm $ACTION ${HELM_RELEASE} ${CHART_PATH} \
                            --namespace ${TKE_NAMESPACE} \
                            --values values-tke.yaml \
                            --set mcpContextForge.image.repository=${IMAGE_REPO} \
                            --set mcpContextForge.image.tag=${IMAGE_TAG} \
                            --timeout 10m \
                            --wait \
                            --atomic \
                            $DRY_FLAG

                        # 等待 rollout 完成并检查状态
                        if [ "${DRY_RUN}" != "true" ]; then
                            kubectl rollout status deployment/${HELM_RELEASE}-mcp-stack-mcpgateway \
                                --namespace ${TKE_NAMESPACE} --timeout=5m
                        fi
                    '''
                }
            }
        }
    }
}
