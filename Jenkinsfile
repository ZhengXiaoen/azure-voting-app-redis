pipeline {
    agent any
    environment {
        APP_VERSION = "1.0."
        ACR_NAME = "zxetest"
        REPOSITORY = "vote-app"
        TENANT_CREDENTIALS_ID = "b2d2659f-b2c7-4914-b189-8e5f7aac8273"
        CLIENT_CREDENTIALS_ID = "b9120790-8654-497e-bf5a-a0159ccab9b6"
    }
    stages {
        stage('getCode') {
            steps {
                git 'https://github.com/ZhengXiaoen/azure-voting-app-redis.git/'
            }
        }
        stage('buildAndPush') {
            steps {
                script{
                    withCredentials([
                        string(credentialsId: "$TENANT_CREDENTIALS_ID", variable: 'TENANT_ID'),
                        usernamePassword(credentialsId: "$CLIENT_CREDENTIALS_ID", passwordVariable: 'SP_CLIENT_SECRET', usernameVariable: 'SP_CLIENT_ID')
                    ]) {
                        sh '''
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
