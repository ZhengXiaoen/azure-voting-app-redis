pipeline {
    agent any
    environment {
        // 环境变量配置（同上）
        APP_VERSION = "1.0."
        ACR_NAME = "zxetest"
        REPOSITORY = "vote-app"
        TENANT_CREDENTIALS_ID = "b2d2659f-b2c7-4914-b189-8e5f7aac8273"
        CLIENT_CREDENTIALS_ID = "b9120790-8654-497e-bf5a-a0159ccab9b6"
    }
    triggers {
        githubPush()
    }
    stages {
        stage('Check Branch') {
            steps {
                script {
                    // 获取当前分支（GitHub推送的分支信息在env.BRANCH_NAME中）
                    def currentBranch = env.BRANCH_NAME ?: "unknown"
                    echo "当前分支: ${currentBranch}"
                    
                    // 非master分支直接终止流水线
                    if (currentBranch != 'master') {
                        error("跳过非master分支: ${currentBranch}")
                    }
                }
            }
        }
        // 后续阶段（仅master分支会执行）
        stage('getCode') {
            steps {
                git 'https://github.com/ZhengXiaoen/azure-voting-app-redis.git/'
            }
        }
        stage('buildAndPush') {
            steps {
                // 构建推送逻辑（同上）
                script{
                    withCredentials([
                        string(credentialsId: "$TENANT_CREDENTIALS_ID", variable: 'TENANT_ID'),
                        usernamePassword(credentialsId: "$CLIENT_CREDENTIALS_ID", passwordVariable: 'SP_CLIENT_SECRET', usernameVariable: 'SP_CLIENT_ID')
                    ]) {
                        sh '''
                            # 同之前的shell脚本
                            az login --service-principal -u $SP_CLIENT_ID -p $SP_CLIENT_SECRET --tenant $TENANT_ID
                            docker build -t $REPOSITORY:$APP_VERSION$BUILD_ID ./azure-vote/
                            docker tag $REPOSITORY:$APP_VERSION$BUILD_ID $ACR_NAME.azurecr.io/$REPOSITORY:$APP_VERSION$BUILD_ID
                            docker push $ACR_NAME.azurecr.io/$REPOSITORY:$APP_VERSION$BUILD_ID
                            docker logout $ACR_NAME.azurecr.io
                            az logout
                        '''
                    }
                }
            }
        }
    }
}
