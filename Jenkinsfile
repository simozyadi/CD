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

  stage('Initialization'){
      steps {

        script{

         

          withCredentials([string(credentialsId: 'AnsibleVault', variable: 'vaultPass')]) {
              sh "echo 'vaultpass: ${vaultPass}' > pass.yaml}"
              sh "cat pass.yaml"
          }

        }
      }
    }
  stage('Update Inventory'){
      steps {
        script{
            withCredentials([usernamePassword(credentialsId: 'simogit', passwordVariable: 'GIT_PWD', usernameVariable: 'GIT_LOGIN')]) {
              sh """
                 
                 ansible-playbook ansible/main.yml -i ansible/hosts -e workdir=${WORKSPACE} -t inventory --extra-vars 'BUILD_NUMBER=${env.BUILD_NUMBER}' --extra-vars 'BRANCH_NAME=${env.BRANCH_NAME}'  --extra-vars 'GIT_LOGIN=${GIT_LOGIN}' --extra-vars 'GIT_PWD=${GIT_PWD}' --extra-vars 'SERVICE_NAME=${SERVICE_NAME}' --extra-vars 'IMAGE_VER=${FINAL_APP_VERSION}'


                """
              }

        }
      }
    }

    stage('Deploy Application'){
      steps {
        script{
          withCredentials([usernamePassword(credentialsId: 'GitCredentialsYass', passwordVariable: 'GIT_PWD', usernameVariable: 'GIT_LOGIN')]) {

            sh """

              pwd && ls
              ansible-playbook ansible/main.yml -i ansible/hosts -e workdir=${WORKSPACE} -t helm --extra-vars '@pass.yaml' --extra-vars 'GIT_LOGIN=${GIT_LOGIN}' --extra-vars 'GIT_PWD=${GIT_PWD}'
            """
          }
        }
      }
    }


   }

}


def getImageTags(imageName, env = null) {
    def filterRegExp

    if(env == 'ppd' || env == 'prd') {
        filterRegExp = '.*'
    } else {
        filterRegExp = '.*'
    }

    def tags = null
    withCredentials([usernamePassword(credentialsId: 'hub-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {

        def portusApiUrl = 'https://auth.docker.io/'
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
