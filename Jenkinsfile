
pipeline {
    agent any
    environment {
        IMAGE_NAME = 'pmcbackend'
        REGISTRY = 'harbor-reg.zetabox.tn'
        // BE_VERSION = readFile(file: 'PMCBackEnd/projectVersion.txt').trim()
        // FINAL_VERSION = "${BE_VERSION}"+"-"+"${env.GIT_BRANCH}"
        BE_VERSION = "${env.BUILD_NUMBER}"+"-"+"${env.GIT_BRANCH}"
        TEAMS_WEBHOOK_URL = "https://prod-110.westeurope.logic.azure.com:443/workflows/daccb247bd0e4c74afaa60b111fbeb78/triggers/manual/paths/invoke?api-version=2016-06-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=4hTMPW0J90sS5w5T25VeakKjEt1rRYERYUytUCqS1yw"
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }

    stages {
        
        stage('say hi'){
          steps{
              
                sh "pwd"
                
              }


        }
        

        stage('Push Docker Image to registry'){
          when{
               anyOf{	 
               branch 'DevelopFlutter'
               branch 'RafikBranch'
               branch 'OmranBranch'
               branch 'OthmenBranch'
			}
           }
          steps{
              withCredentials([usernamePassword(credentialsId: 'docker_registry', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh "docker login ${REGISTRY} -u $username -p $password"
                sh "docker tag ${IMAGE_NAME} ${REGISTRY}/pmc/${IMAGE_NAME}:${BE_VERSION}"
                sh "docker push ${REGISTRY}/pmc/${IMAGE_NAME}:${BE_VERSION}"
              }
          }
        }

        stage ('Update Docker image In VPS'){
           when{
		    anyOf{			
                      branch 'DevelopFlutter'
                      branch 'RafikBranch'
                      branch 'OmranBranch'
                      branch 'OthmenBranch'
			}
           }
           environment {
               PORT        = '8010'
           }
           steps{
               echo "switch docker context to the remote engine !"
               sh "docker context use pmc-remote-engine" 
               sh """if [ \$(docker ps -a --format 'table {{.Names}}' | grep ${env.GIT_BRANCH}) ]
                   then
                      docker stop ${env.GIT_BRANCH} && docker rm ${env.GIT_BRANCH}
                   fi"""
		   
                   sh "docker run -d --name ${env.GIT_BRANCH} --restart always --env-file ./.env -v /etc/localtime:/etc/localtime:ro -v /home/pmc/log/DevelopFlutter/:/tmp -p ${PORT}:8000 ${REGISTRY}/pmc/${IMAGE_NAME}:${BE_VERSION}"
           }
        }
    }
    post{
	    success {
            office365ConnectorSend color: '#86BC25', 
		                           status: "${currentBuild.currentResult}", 
		                           webhookUrl: "${TEAMS_WEBHOOK_URL}",
		                           message: "Build Success: ${JOB_NAME} - ${currentBuild.displayName}<br>Pipeline duration: ${currentBuild.durationString.replace(' and counting', '')}"
         }
        failure {
            office365ConnectorSend color: '#ff0000',
		                           status: "${currentBuild.currentResult}",
		                           webhookUrl: "${TEAMS_WEBHOOK_URL}",
                                    message: "Build Failed: ${JOB_NAME} - ${currentBuild.displayName}<br>Pipeline duration: ${currentBuild.durationString.replace(' and counting', '')}"
         }
        always {
            cleanWs(deleteDirs: true)
            echo "switch docker context to default !"
            sh "docker context use default"
        }
    }
}
