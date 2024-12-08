pipeline {
    agent any
    parameters {
       string(defaultValue: 'platform', description: 'Latest value is master', name: 'MATRIX_BRANCH_OR_TAG')
       string(defaultValue: 'random', description: 'Only use in rare cases', name: 'MATRIX_CUSTOM_IMAGE_TAG')
    }
    options {
        // Timeout counter starts AFTER agent is allocated
        timeout(time: 100, unit: 'MINUTES')
      
        }
    stages {
        stage('Build') {
            steps {
              cleanWs()
              git branch: "${params.MATRIX_BRANCH_OR_TAG}", credentialsId: 'git_matrix_id', url: 'https://bitbucket-eng-rtp1.cisco.com/bitbucket/scm/cm/matrix4.git'
              withCredentials([
                    usernamePassword(credentialsId: 'docker_matrix_id', passwordVariable: 'DOCKER_PWD', usernameVariable: 'DOCKER_USER')
                    ]) {
                        timestamps {
                            dir("${env.WORKSPACE}/") {
                                    sh """
                                        # cleaning previous docker images save reclaim space
                                        docker images -a
                                        if [ `docker images -a -q|wc -l` -gt 0 ]; then
                                            docker rmi -f `docker images -a -q`
                                        fi
                                        #checking free space
                                        df -h
                                        set +e
                                        env |grep -i DOCKER
                                        echo ${params.MATRIX_BRANCH_OR_TAG}
                                        if [ -z ${params.MATRIX_CUSTOM_IMAGE_TAG} ]; then
                                            bash buildMatrix4BaseImage.sh -u upload 
                                        else
                                            bash buildMatrix4BaseImage.sh -u upload  -c ${params.MATRIX_CUSTOM_IMAGE_TAG}
                                        fi
                                        """
                            }
                        }
                    }
           }
        }
        stage (clean){
            steps {
                cleanWs cleanWhenFailure: true, cleanWhenNotBuilt: true
            }
        }

    }
    post {
         success {  
             emailext body: "${env.BUILD_URL}", recipientProviders: [requestor()], subject: "Matrix4 Image Build completed in ${currentBuild.durationString.replace(' and counting', '')}"
         }  
         failure {  
            emailext body: "${env.BUILD_URL}", recipientProviders: [requestor()], subject: "Matrix4 Image Build failed"   
         }  
        always { 
            sh '''
                docker logout dockerhub.cisco.com
            '''
        }
    }
}
