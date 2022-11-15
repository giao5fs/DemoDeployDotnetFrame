pipeline {
    agent { label 'master' }
    
    environment {
        BRANCH = 'staging'
        SOLUTION = 'Removethebackground.API.sln'
        PROJECT_PATH = 'Removethebackground.API.sln'
        APP_NAME = 'soapapi'
        ZIP_PACKAGE = "${WORKSPACE}/RTB.WebAPI/obj/Debug/Package/RTB.WebAPI.zip"
        // TARGETGROUPARN = "arn:aws:elasticloadbalancing:us-east-1:781748326470:targetgroup/https-all-pixelz/efc8f621a5c7aceb"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "${BRANCH}"]], extensions: [[$class: 'CleanBeforeCheckout', deleteUntrackedNestedRepositories: true]], userRemoteConfigs: [[credentialsId: 'thanglq_jenkins_application_password_readonly', url: 'https://bitbucket.org/pixelzdev/internal-service/src/NBE-0001-init-folder/']]])
            }
        }
        stage('Build') {
            steps {
                Build('Debug')
            }
        }
        stage('Deploy') {
            steps {
                script {
    		        def INSTANCEID = "i-0278e01402a889574"
                    def DEPLOY_SERVER = '52.221.37.194'

                    withCredentials([usernamePassword(credentialsId: 'RTB-DEV05-WEB', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        DeployToRemoteIIS("https://${DEPLOY_SERVER}:8172/msdeploy.axd", USERNAME, PASSWORD, APP_NAME, ZIP_PACKAGE)
                    }
                }
            }
        }
        stage('SendNotify') {
            steps {
                SendNotifyEmail(currentBuild.currentResult)
            }
        }
    }
}

void SendNotifyEmail(result) {
    emailext (
        body: """<p>${result}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                 <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
        recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], 
        subject: "[Jenkins] ${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - ${result}"                
    )    
}

void Build(buildMode) {
    bat "nuget restore \"${SOLUTION}\""
                                                  
    bat returnStatus: true,
        script: """
            \"${tool 'MsBuild15'}\" \"${PROJECT_PATH}\" ^
                /p:Configuration=${buildMode} ^
                /p:Platform=\"Any CPU\" ^
                /p:AutoParameterizationWebConfigConnectionStrings=False ^
                /p:DeployOnBuild=True ^
                /p:GenerateSerializationAssemblies=Off ^
                /p:WebPublishMethod=Package ^
                /p:PackageAsSingleFile=true ^
                /verbosity:minimal ^
                /nologo ^
        """
}

void DeployToRemoteIIS(computerName, username, password, webApplicationName, zip_package){
    try {
        bat """
            \"${tool 'MsDeploy'}\" ^
                -source:package="${zip_package}" ^
                -dest:auto,computerName=\"${computerName}\",username=\"${username}\",password=\"${password}\",authtype=\"Basic\",includeAcls=\"False\" ^
                -verb:sync ^
                -disableLink:AppPoolExtension ^
                -disableLink:ContentExtension ^
                -disableLink:CertificateExtension ^
                -setParam:name=\"IIS Web Application Name\",value=\"${webApplicationName}\" ^
                -enableRule:AppOffline ^
                -allowUntrusted ^
        """
    } catch (e) {
        bat """
            \"${tool 'MsDeploy'}\" ^
                -verb:delete ^
                -allowUntrusted ^
                -dest:contentPath=${webApplicationName}/App_Offline.htm,computerName=\"${computerName}",username=\"${username}\",password=\"${password}\",authtype=\"Basic\" ^
        """
        throw e
    }
}
