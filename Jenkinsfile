def FINAL_APP_ENV = "dev"
def FINAL_APP_NAME = "nginx-app"



pipeline {

   environment {

        IMAGE_VER           = "${params.APP_VERSION}"
        SERVICE_NAME        = "nginx-app" 
   }
	
	
   agent {  docker {

	    image 'siticom/terraform-ansible'
	    label 'cdnode'
	    args "-u root:root --entrypoint='' --network host"

            }    } 

   
	
    stages {      

         stage ('Choose a tag to deploy') {

            steps {
                script {
                    FINAL_APP_VERSION = input message: 'Please Choose Your Tag',
                            ok: 'Next',
                            parameters: [
                                    choice(name: 'APP_VERSION',
                                            choices: getImageTags(FINAL_APP_NAME, FINAL_APP_ENV),
                                            description: 'List of docker tags')
                            ]
                }
            }
        }

	    
stage('Initialization'){
      steps {
       
        script{
          sh "touch pass_file"
          sh "chmod 640 pass_file"
          vault_file="vault_file.yaml"
     			
          withCredentials([string(credentialsId: 'AnsibleVault', variable: 'password')]) {
              sh "echo ${password} > ${vault_file}"
              sh "cat vault_file.yaml"
          }
                      
        }
      }
    }

   }

}


def getImageTags(imageName, env = null) {
    
	def filterRegExp = '.*'

    	def tags = null
	
    	withCredentials([usernamePassword(credentialsId: 'hub-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {

        def authApiUrl = 'https://auth.docker.io/'
        def dockerApiUrl = 'https://registry-1.docker.io/v2'
        def reposName = "nimrodops/${imageName}"

        def getTokenUrl = "${authApiUrl}token?service=registry.docker.io&scope=repository:${reposName}:pull"
        def getTokenCommand = 'set +x && curl -s GET \'' + getTokenUrl + '\' -H "Authorization: Basic $(echo -n $USERNAME:$PASSWORD | base64)"'
        def getTokenResponse = sh(script: getTokenCommand, returnStdout: true)
        def token = readJSON(text: getTokenResponse).token

        def getTagsUrl = "${dockerApiUrl}/${reposName}/tags/list"
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

