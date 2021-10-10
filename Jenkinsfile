slack_channel = 'android-ci-cd'

pipeline
{ 
    agent{
        docker{
            image 'mindbowser/android-30-sdk:1.0'
            args '-u root:root'
        }
    }
    stages{
        stage('Init')
        {
            //check git commit message contains "skip ci" if found don't run the pipeline
             
            steps {
                script {

                    lastCommitInfo = sh(script: "git log -1", returnStdout: true).trim()
                    
                    commitContainsSkip = sh(script: "git log -1 | grep 'skip ci'", returnStatus: true)
                    
                    slackMessage = "*${env.JOB_NAME}* *${env.BRANCH_NAME}* received a new commit. \nHere is commmit info: ${lastCommitInfo}\n*Console Output*: <${BUILD_URL}/console | (Open)>"
                    
                    slack_send(slackMessage)

                    if(commitContainsSkip == 0) {
                        skippingText = " Skipping Build for *${env.BRANCH_NAME}* branch."
                        currentBuild.result = 'ABORTED'
                        slack_send(skippingText,"warning")
                        error('BUILD SKIPPED') 
                    }
                }
            }
        }
        stage('build')
        {
            // call fastlane lane for generate apk and uploading to testflight
                 steps{

                    sh "chmod +x gradlew"
                    
                    sh "chmod +x Gemfile"
                    
                    sh "fastlane build --env ${env.BRANCH_NAME}"    //eg. fastlane build --env development
                }
        }

    }
    post {
        always {
            // delete the workspace

            sh "chmod -R 777 ."
            
            deleteDir() 
        }
        success{
            
             slack_send("Jenkins job  for *${env.BRANCH_NAME}* completed successfully. ","#0066ff")
        }
        aborted{
            slack_send("Jenkins job  for *${env.BRANCH_NAME}* Skipped/Aborted.","warning")
        }
        failure {
            
          slack_send("*${env.BRANCH_NAME}* Something went wrong.Build failed. Check here: Console Output*: <${BUILD_URL}/console | (Open)>","danger")
        }
    }

}

def slack_send(slackMessage,messageColor="good")
{
    slackSend channel: slack_channel , color: messageColor, message: slackMessage
}