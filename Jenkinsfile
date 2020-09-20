def FINAL_APP_ENV = "dev"
def FINAL_APP_NAME = "nginx-app"
def FINAL_APP_ENV_TYPE = "dev"


pipeline {

    environment {

        IMAGE_VER           = "${params.APP_VERSION}"
        SERVICE_NAME        = "nginx-app" 
    }
   agent   {  

        docker {

	    image 'siticom/terraform-ansible'
	    label 'cdnode'
	    args "-u root:root --entrypoint='' --network host"

        }

    } 

    stages {
      

         stage ('Choose a tag to deploy') {
            when {
                expression { return params.APP_ENV == null}
            }
            agent none
            options { timeout(time: 5, unit: 'MINUTES')}
            steps {
                script {
                    FINAL_APP_VERSION = input message: 'Please Choose Your Tag',
                            ok: 'Next',
                            parameters: [
                                    choice(name: 'APP_VERSION',
                                            choices: getImageTags(FINAL_APP_NAME, FINAL_APP_ENV),
                                            description: 'Available Docker images belong to the application you have chosen')
                            ]
                }
            }
        }



   }

}


def getImageTags(imageName, env = null) {
    def filterRegExp

    if(env == 'ppd' || env == 'prd') {
        filterRegExp = '^\\d+\\.\\d+\\.\\d+$'
    } else {
        filterRegExp = '^\\d+\\.\\d+\\.\\d+.*'
    }

    def tags = null
    withCredentials([usernamePassword(credentialsId: 'hub-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {

        def portusApiUrl = 'https://auth.docker.io/v2'
        def dockerRegistryApiUrl = 'https://registry-1.docker.io/v2'
        def reposName = "nimrodops/${imageName}"

        def getTokenUrl = "${portusApiUrl}token?service=registry.docker.io&scope=repository:${reposName}:pull"
        def getTokenCommand = 'set +x && curl -s GET \'' + getTokenUrl + '\' -H "Authorization: Basic $(echo -n $USERNAME:$PASSWORD | base64)"'
        def getTokenResponse = sh(script: getTokenCommand, returnStdout: true)
        def token = readJSON(text: getTokenResponse).token

        def getTagsUrl = "${dockerRegistryApiUrl}/${reposName}/tags/list"
        def getTagsCommand = 'set +x && curl -s GET \'' + getTagsUrl + '\' -H \'Authorization: Bearer ' + token + '\''
        def getTagsResponse = sh(script: getTagsCommand, returnStdout: true)
        tags = readJSON(text: getTagsResponse).tags

        if(filterRegExp != null) {
            def filterPattern = ~"${filterRegExp}"
            tags = tags?.findAll{tag -> tag =~ filterPattern}
        }
    }

    return tags?.sort()
}

