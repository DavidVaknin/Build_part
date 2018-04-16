
pipeline { 
    agent any 

     parameters
    {
        string(defaultValue: 'git@github.com:DavidVaknin/Build_part.git', description: 'The url of the git repository the contains the projects CMakeLists.txt file in the root directory.  ', name: 'RepositoryUrl')
        string(defaultValue: '/home/matt/Documents/DuduV/Build_part/Build_part', description: 'A workspace directory on the master and build-slave into which the code is checked-out and which is used for the build.  ', name: 'CheckoutDirectory')
        string(defaultValue: '', description: 'The tag for the build-slave on which the project is build.', name: 'BuildSlaveTag')
        string(defaultValue: 'master', description: 'Relevant branch to test.', name: 'Branch')
        string(defaultValue: 'david.vaknin@devalore.com', description: 'write mailRecipients.', name: 'MailRecipients')
          choice(
                name: 'BuildType',
                choices:"Debug\nRelease",
                description: "Select build type")
    }   
    
    stages  
     {  
          stage('Analysis test')
        {
            steps 
            {
               sh  'cppcheck --enable=all --inconclusive --xml-version=2 --force --library=windows,posix,gnu libbar/ 2> Cppcheck_result.xml'
               sh 'ls -l'
               // Cppcheck Dosnt Support for now
               //   junit 'result.xml' 
            }
        }
        stage('Build and Test') 
        {

            steps
            {   
                
                node(params.BuildSlaveTag)
                {
                    // acquiering an extra workspace seems to be necessary to prevent interaction between
                    // the parallel run nodes, although node() should already create an own workspace.
                    ws(params.CheckoutDirectory)   
                    {   
                        // debug info
                        printJobParameter()
                    
                        // checkout sources
                        checkout([$class: 'GitSCM',
                            userRemoteConfigs: [[url: params.RepositoryUrl]],
                            branches: [[name: params.Branch]],
                            extensions: [[$class: 'CleanBeforeCheckout']]]
                        )
                   
                        script{
                            try {
                                notifyStarted()
                                // run cmake generate and buildmkdir Release
                                cmakeBuild buildDir: 'build', buildType: params.BuildType , installation: 'InSearchPath', steps: [[args: '--target install', withCmake: true]]    
                            
                                echo '----- CMake project was build successfully -----'
                                notifySuccessful()
                            } catch (e) {
                                currentBuild.result = "FAILED"
                                notifyFailed()
                                throw e
                            }
                        }
                    }
                }                       
            }
             
                post { 
                    failure { 
                     step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject: "Pipeline build fail: ${currentBuild.fullDisplayName}", recipients: params.MailRecipients, sendToIndividuals: true])
                    }
                }

        }
        stage('Report') 
        {
             steps 
            {
                sh 'cppcheck-htmlreport  --file=Cppcheck_result.xml --title=LibreOffice --report-dir=cppcheck_reports --source-dir='
                
                //sh 'chmod -R 777 Cppcheck_reports/index.html'
                //sh 'Cppcheck_reports/index.html'
                //sh './testfoo --gtest_output=xml'
                sh 'ls -l'

                 /* ...HTML report... */

                 // Archive the built artifactsa
                archive (includes: 'pkg/*.gem')

                // publish html
                 // snippet generator doesn't include "target:"
                publishHTML (target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: 'cppcheck_reports',
                    reportFiles: 'index.html',
                    reportName: "Cppcheck Report"
                    ])

                
                /*Email report*/ 
                emailext (attachLog: true, body: '''${SCRIPT, template="buildlog.template"}''', compressLog: true, mimeType: 'text/html', subject: 'Build logs', to: params.MailRecipients, replyTo: params.MailRecipients,recipientProviders: [[$class: 'DevelopersRecipientProvider']])
                
            }
            post { 
                failure { 
                    step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject:"pipeline SUCCESS", recipients: params.MailRecipients, sendToIndividuals: true])
                }
            }
        }
    }
}


def printJobParameter()
{
    def introduction = """
----- Build CMake project -----
RepositoryUrl = ${params.RepositoryUrl}
CheckoutDirectory = ${params.CheckoutDirectory}
BuildSlaveTag = ${params.BuildSlaveTag}
-------------------------------
"""
    
    echo introduction
}
def notifyStarted() {
  // send to Slack
  slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
 
  // send to HipChat
  hipchatSend (color: 'YELLOW', notify: true,
      message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
    )
 
  // send to email
  emailext (
      subject: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
    )
}

def notifySuccessful() {
  slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
 
  hipchatSend (color: 'GREEN', notify: true,
      message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
    )
 
  emailext (
      subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
    )
}

def notifyFailed() {
  slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
 
  hipchatSend (color: 'RED', notify: true,
      message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
    )
 
  emailext (
      subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
    )
}