pipeline {
    agent any
    
    environment {
        FAILED_STAGE = ''
        IMAGE_NAME = 'pmcbackend'
        REGISTRY = 'harbor-reg.zetabox.tn'
        BE_VERSION = "${env.BUILD_NUMBER}-${env.GIT_BRANCH}"
        TEAMS_WEBHOOK_URL = "https://prod-110.westeurope.logic.azure.com:443/workflows/daccb247bd0e4c74afaa60b111fbeb78/triggers/manual/paths/invoke?api-version=2016-06-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=4hTMPW0J90sS5w5T25VeakKjEt1rRYERYUytUCqS1yw"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }

    stages {
        stage('say hi') {
            steps {
                sh "pwd"
                FAILED_STAGE= "${env.STAGE_NAME}"
            }
        }

        stage('Push Docker Image to registry') {
            when {
                anyOf {
                    branch 'DevelopFlutter'
                    branch 'RafikBranch'
                    branch 'OmranBranch'
                    branch 'main'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_registry', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh "docker login ${REGISTRY} -u $username -p $password"
                    sh "docker tag ${IMAGE_NAME} ${REGISTRY}/pmc/${IMAGE_NAME}:${BE_VERSION}"
                    sh "docker push ${REGISTRY}/pmc/${IMAGE_NAME}:${BE_VERSION}"
                    FAILED_STAGE= "${env.STAGE_NAME}"
                }
            }
        }

        stage('Update Docker image In VPS') {
            when {
                anyOf {
                    branch 'DevelopFlutter'
                    branch 'RafikBranch'
                    branch 'OmranBranch'
                    branch 'OthmenBranch'
                }
            }
            environment {
                PORT = '8010'
            }
            steps {
                echo "Switching Docker context to the remote engine!"
                sh "docker context use pmc-remote-engine"
                sh """
                if [ \$(docker ps -a --format 'table {{.Names}}' | grep ${env.GIT_BRANCH}) ]; then
                    docker stop ${env.GIT_BRANCH} && docker rm ${env.GIT_BRANCH}
                fi
                """
                sh "docker run -d --name ${env.GIT_BRANCH} --restart always --env-file ./.env -v /etc/localtime:/etc/localtime:ro -v /home/pmc/log/DevelopFlutter/:/tmp -p ${PORT}:8000 ${REGISTRY}/pmc/${IMAGE_NAME}:${BE_VERSION}"
                FAILED_STAGE= "${env.STAGE_NAME}"
            }
        }
    }
    
    post {
        always {
            script {
                
                    env.GIT_AUTHOR = sh(script: 'git show -s --pretty=%an', returnStdout: true).trim()
                    // Send notification
                    msteamsNotification(currentBuild.currentResult)
                
                   
               
             }
        }
    }
}

def msteamsNotification(buildStatus) {
    def appName = 'PMC Backend' // Set your app name here
    def workflowUrl = TEAMS_WEBHOOK_URL
    def buildStartTime = new Date(currentBuild.startTimeInMillis + currentBuild.duration).format("yyyy-MM-dd HH:mm:ss")
    def imageUrl = "https://www.jenkins.io/images/logos/jenkins/jenkins.png"
    def bldStatus = "Jenkins Build SUCCESS"
    def bldStatusColor = "Good"
    def stageStatus = "All Stages Passed"

    // Customize the notification based on build result
    if (buildStatus == 'FAILURE') {
        imageUrl = "https://www.jenkins.io/images/logos/fire/fire.png"
        bldStatus = "Jenkins Build FAILED"
        bldStatusColor = "Warning"
        stageStatus = "Failed in stage: ${env.failedStage}"
    }

    def payload = """
    {
        "type": "message",
        "attachments": [
            {
                "contentType": "application/vnd.microsoft.card.adaptive",
                "content": {
                    "\$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
                    "type": "AdaptiveCard",
                    "version": "1.6",
                    "body": [
                        {
                            "type": "TextBlock",
                            "size": "Medium",
                            "weight": "Bolder",
                            "text": "Build Notification"
                        },
                        {
                            "type": "ColumnSet",
                            "columns": [
                                {
                                    "type": "Column",
                                    "items": [
                                        {
                                            "type": "Image",
                                            "url": "${imageUrl}",
                                            "altText": "Jenkins Logo",
                                            "size": "Medium",
                                            "height": "50px",
                                            "width": "48px"
                                        }
                                    ],
                                    "width": "auto"
                                },
                                {
                                    "type": "Column",
                                    "items": [
                                        {
                                            "type": "TextBlock",
                                            "weight": "Bolder",
                                            "text": "${bldStatus}",
                                            "color": "${bldStatusColor}",
                                            "wrap": true,
                                            "style": "heading",
                                            "size": "ExtraLarge",
                                            "isSubtle": true
                                        },
                                        {
                                            "type": "TextBlock",
                                            "spacing": "None",
                                            "text": "Build end time: ${buildStartTime}",
                                            "wrap": true,
                                            "isSubtle": true
                                        }
                                    ],
                                    "width": "auto",
                                    "verticalContentAlignment": "Center"
                                }
                            ]
                        },
                        {
                            "type": "FactSet",
                            "separator": true,
                            "facts": [
                                {
                                    "title": "App Name:",
                                    "value": "${appName}"
                                },
                                {
                                    "title": "Stage:",
                                    "value": "${stageStatus}"
                                },
                                {
                                    "title": "Job Name:",
                                    "value": "${env.JOB_BASE_NAME}"
                                },
                                {
                                    "title": "Build Number:",
                                    "value": "${env.BUILD_NUMBER}"
                                },
                                {
                                    "title": "Commit Author:",
                                    "value": "${env.GIT_AUTHOR}"
                                }
                            ],
                            "spacing": "Medium",
                            "separator": true
                        }
                    ],
                    "actions": [
                        {
                            "type": "Action.OpenUrl",
                            "title": "View ${appName} Build",
                            "url": "${env.BUILD_URL}",
                            "iconUrl": "https://i.ibb.co/Ks2JKfG/cloudbees-logo-icon-168396.png"
                        }
                    ]
                }
            }
        ]
    }
    """
    
    // Send notification to MS Teams
    httpRequest(
        httpMode: 'POST',
        acceptType: 'APPLICATION_JSON',
        contentType: 'APPLICATION_JSON',
        url: workflowUrl,
        requestBody: payload
    )
}
