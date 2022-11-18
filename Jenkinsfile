pipeline {
    agent { label 'windows' }
    
    environment {
        BRANCH = 'master-dev05'
        SOLUTION = 'DemoDeployDotnetFrame.sln'
        PROJECT_PATH = 'DemoDeployDotnetFrame.sln'
        APP_NAME = 'DemoDeployDotnetFrame'
        ZIP_PACKAGE = "${WORKSPACE}/DemoDeployDotnetFrame/obj/Release/Package/DemoDeployDotnetFrame.zip"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "${BRANCH}"]], extensions: [[$class: 'CleanBeforeCheckout', deleteUntrackedNestedRepositories: true]], 
                userRemoteConfigs: [[credentialsId: 'Yugi1973', 
                url: 'https://github.com/giao5fs/DemoDeployDotnetFrame.git']]])
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
                    withCredentials([usernamePassword(credentialsId: 'MSDeploy_Credential', passwordVariable: "PASSWORD", usernameVariable: "USERNAME")]) {
                        DeployToRemoteIIS("https://35.171.162.27:8172/MSDeploy.axd", USERNAME, PASSWORD, APP_NAME, ZIP_PACKAGE)
                    }
                }
            }
        }
    }
}

void Build(buildMode) {
    bat "nuget restore \"${SOLUTION}\""
                                                  
    bat returnStatus: true,
        script: """
            \"${tool 'MSBuild2019'}\" \"${PROJECT_PATH}\" ^
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
            \"${tool 'MSDeploy'}\" ^
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
            \"${tool 'MSDeploy'}\" ^
                -verb:delete ^
                -allowUntrusted ^
                -dest:contentPath=${webApplicationName}/App_Offline.htm,computerName=\"${computerName}",username=\"${username}\",password=\"${password}\",authtype=\"Basic\" ^
        """
        throw e
    }
}
