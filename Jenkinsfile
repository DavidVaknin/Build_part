pipeline { 
    agent any 
   
     parameters
    {
        string(defaultValue: 'git@github.com:DavidVaknin/Build_part.git', description: 'The url of the git repository the contains the projects CMakeLists.txt file in the root directory.  ', name: 'RepositoryUrl')
        string(defaultValue: '/home/matt/Documents/DuduV/Build_part/', description: 'A workspace directory on the master and build-slave into which the code is checked-out and which is used for the build.  ', name: 'CheckoutDirectory')
        string(defaultValue: '', description: 'The tag for the build-slave on which the project is build.', name: 'BuildSlaveTag')
        string(defaultValue: 'master', description: 'Relevant branch to test.', name: 'Branch')
        string(defaultValue: 'david.vaknin@devalore.com', description: 'write mailRecipients.', name: 'MailRecipients')
        string(defaultValue: 'pybot /home/matt/Documents/DuduV/Build_part/robot3_test/test1.robot', description: 'Directory to robot test ', name: 'RobotTestDirectory')
        choice(name: 'BuildType', choices:"Debug\nRelease", description: "Select build type")       
        booleanParam(defaultValue: true, description: 'Unchek for skip on this step', name: 'Analysis_test')
        booleanParam(defaultValue: true, description: 'Unchek for skip on this step', name: 'Build')
        booleanParam(defaultValue: true, description: 'Unchek for skip on this step', name: 'Report')
        booleanParam(defaultValue: true, description: 'Unchek for skip on this step', name: 'Send_mail')
    }  
    environment{

        // 'This value is exported to all commands in this stage'
        ROBOTRUN = 'pybot'
    }    
    
    
    stages  
    {  
          stage('Analysis test')
        {   
           
            steps 
            { 
                script{
                    if(params.Analysis_test){
                    runCommand('cppcheck --enable=all --inconclusive --xml-version=2 --force --library=windows,posix,gnu libbar/ 2> Cppcheck_result.xml')
                    // Cppcheck Dosnt Support for now
                    //   junit 'result.xml' 
                    } else echo "Analysis test step skipped"
                }
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

                        //sh '''if [ -d /usr/share]
                         //   then
                         //        echo "Directoty exist"
                         //   else
                         //       mkdir /usr/share
                         //   fi
                         //   '''


                        // Clean workspace before build
                        //cleanWs()
                        // checkout sources
                        checkout([$class: 'GitSCM',
                            userRemoteConfigs: [[url: params.RepositoryUrl]],
                            branches: [[name: params.Branch]],
                            extensions: [[$class: 'CleanBeforeCheckout']]]
                        )
                   
                        script{
                            if(params.Build){
                                // run cmake generate and build
                                cmakeBuild buildDir: 'build', buildType: params.BuildType , installation: 'InSearchPath', steps: [[args: '--target install', withCmake: true]]    
                             }else echo "build step skipped"
                        }        
                    }
                }
            }                       
            
            post { 
                failure { 
                    step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject: "${currentBuild.currentResult}: ${env.JOB_NAME} - build ${currentBuild.number} = 'FAILURE'", recipients: params.MailRecipients, sendToIndividuals: true])
                }
            }
        }
           
        stage('Report') 
            {
            steps 
            {
                script{
                    if(params.Report){
                        runCommand('cppcheck-htmlreport  --file=Cppcheck_result.xml --title=LibreOffice --report-dir=cppcheck_reports --source-dir=')
                        
                        
                        //sh './testfoo --gtest_output=xml'
                     
                        runCommand('ls ../Build_part')

                            /*-------Robot FrameWork------*/
                            
                        runCommand( ${ROBOTRUN} params.RobotTestDirectory)

                        try{
                            step([$class: 'RobotPublisher',
                                disableArchiveOutput: false,
                                logFileName: 'log.html',
                                otherFiles: '',
                                outputFileName: 'output.xml',
                                outputPath: '.',
                                passThreshold: 100,
                                reportFileName: 'report.html',
                                unstableThreshold: 0]);
                        }catch(exc){
                            echo 'Something failed in Robot publisher'
                        }
 
                        /* ...HTML report... */

                        // Archive the built artifacts
                        archive (includes: 'pkg/*.gem')
                        //archiveArtifacts "xml"
                        

                        /*-----publish html-----*/
                        // snippet generator doesn't include "target:"
                        publishHTML (target: [
                            allowMissing: false,    
                            alwaysLinkToLastBuild: false,
                            keepAll: true,
                            reportDir: 'cppcheck_reports',
                            reportFiles: 'index.html',
                            reportName: "Cppcheck Report"
                            ])

                    }else echo "Report step skipped"   
                }
            }
            post { 
                failure { 
                    step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject: "${currentBuild.currentResult}: ${env.JOB_NAME} - Report ${currentBuild.number} = 'FAILURE'", recipients: params.MailRecipients, sendToIndividuals: true])
                }
            }
        }
        
        stage('Send email') {
            steps{
                script{
                    if(params.Send_mail){
                        emailext (body: '''${SCRIPT, template="buildlog.template"}''',
                        mimeType: 'text/html',
                        subject: "[Jenkins] Buildlog",
                        to: params.MailRecipients,
                        replyTo: params.MailRecipients,
                        recipientProviders: [[$class: 'CulpritsRecipientProvider']])

                    }else echo "Send mail skipped"
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

def runCommand( command )
{
    if(isUnix())
    {
        sh command
    }
    else
    {
        bat command
    }
}
