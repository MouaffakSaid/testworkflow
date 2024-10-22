pipeline {
    agent any
    environment {
        IMAGE_NAME = 'pmcbackend'
        REGISTRY = 'harbor-reg.zetabox.tn'
        BE_VERSION = "${env.BUILD_NUMBER}"+"-"+"${env.GIT_BRANCH}"
        TEAMS_WEBHOOK_URL = "your-webhook-url-here"
        failedStage = '' // Initialize empty value for failedStage
    }

    stages {
        stage('say hi'){
            steps{
                sh "pwd"
            }
        }
        
        stage('Push Docker Image to registry'){
            when {
                anyOf {
                    branch 'DevelopFlutter'
                    branch 'RafikBranch'
                    branch 'OmranBranch'
                    branch 'main'
                }
            }
            steps {
                script {
                    try {
                        // Code for pushing the Docker image
                    } catch (e) {
                        env.failedStage = 'Push Docker Image to registry' // Assign failed stage name
                        throw e // Rethrow the exception to mark the stage as failed
                    }
                }
            }
        }

        stage('Update Docker image In VPS'){
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
                script {
                    try {
                        // Code for updating the Docker image
                    } catch (e) {
                        env.failedStage = 'Update Docker image In VPS' // Assign failed stage name
                        throw e // Rethrow the exception to mark the stage as failed
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                try {
                    // Capture the Git author name
                    env.GIT_AUTHOR = sh(script: 'git show -s --pretty=%an', returnStdout: true).trim()
                    // Send notification
                    msteamsNotification(currentBuild.currentResult)
                } catch (Exception e) {
                    // Log the error if the notification fails
                    echo "Error in sending Teams notification: ${e}"
                }
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
