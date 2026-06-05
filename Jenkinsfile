// ============================================================================
// Jenkinsfile — mcp-context-forge 部署到 腾讯云 TKE
// ============================================================================
// Coding DevOps 需配置:
//   环境变量:
//     TCR_NAMESPACE_NAME  = cyouops
//     TCR_REPOSITORY_NAME = mcp-context-forge
//     TKE_CLUSTER_CREDENTIAL_ID = <TKE凭据ID>
//   凭据管理:
//     tcr-credentials  : 用户名+密码 → TCR 访问凭证
//     tke-kubeconfig   : 上传文件    → TKE 集群 kubeconfig
// ============================================================================

pipeline {
    agent any

    parameters {
        booleanParam(name: 'DRY_RUN', defaultValue: false, description: '仅校验，不实际部署')
    }

    environment {
        TCR_REGISTRY = 'ccr.ccs.tencentyun.com'
        IMAGE = "${TCR_REGISTRY}/${TCR_NAMESPACE_NAME}/${TCR_REPOSITORY_NAME}:${TCR_IMAGE_NAME}"
    }

    stages {
        stage('构建并推送镜像') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'tcr-credentials',
                    usernameVariable: 'TCR_USER',
                    passwordVariable: 'TCR_PASS'
                )]) {
                    sh '''
                        echo "构建: ${IMAGE}"
                        docker build -f Containerfile -t ${IMAGE} .

                        echo "${TCR_PASS}" | docker login ${TCR_REGISTRY} -u ${TCR_USER} --password-stdin
                        docker push ${IMAGE}
                        docker logout ${TCR_REGISTRY}
                    '''
                }
            }
        }

        stage('部署到 TKE') {
            when { expression { !params.DRY_RUN } }
            steps {
                withCredentials([file(credentialsId: env.TKE_CLUSTER_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl get namespace mcp-gateway 2>/dev/null || kubectl create namespace mcp-gateway
                        kubectl apply -f k8s/
                        kubectl set image deployment/mcp-context-forge-mcpgateway \
                            mcpgateway=${IMAGE} \
                            --namespace mcp-gateway
                        kubectl rollout status deployment/mcp-context-forge-mcpgateway \
                            --namespace mcp-gateway --timeout=5m
                    '''
                }
            }
        }
    }
}
