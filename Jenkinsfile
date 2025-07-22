stage('buildAndPush') {
    steps {
        script{
            withCredentials([
                string(credentialsId: "$TENANT_CREDENTIALS_ID", variable: 'TENANT_ID'),
                usernamePassword(
                    credentialsId: "$CLIENT_CREDENTIALS_ID", 
                    passwordVariable: 'SP_CLIENT_SECRET', 
                    usernameVariable: 'SP_CLIENT_ID'
                )
            ]) {
                sh '''
                    # 1. 登录Azure CLI（服务主体）
                    az login --service-principal \
                      -u $SP_CLIENT_ID \
                      -p $SP_CLIENT_SECRET \
                      --tenant $TENANT_ID
                    
                    # 2. 关键：让Docker登录到ACR（自动获取临时凭证）
                    az acr login --name $ACR_NAME
                    
                    # 3. 构建镜像
                    docker build -t $REPOSITORY:$APP_VERSION$BUILD_ID ./azure-vote/
                    
                    # 4. 标记镜像（ACR镜像格式必须是：<acr-name>.azurecr.io/<仓库名>:<标签>）
                    docker tag $REPOSITORY:$APP_VERSION$BUILD_ID $ACR_NAME.azurecr.io/$REPOSITORY:$APP_VERSION$BUILD_ID
                    
                    # 5. 推送镜像到ACR（此时Docker已通过az acr login认证）
                    docker push $ACR_NAME.azurecr.io/$REPOSITORY:$APP_VERSION$BUILD_ID
                    
                    # 6. 清理登录状态
                    docker logout $ACR_NAME.azurecr.io
                    az logout
                '''
            }
        }
    }
}
