pipeline { 

    agent {
        node {
            label 'build'
        }
    }

    parameters {
        string(defaultValue: 'master', description: 'Relevant branch to test.', name: 'Branch')
        string(defaultValue: 'david.vaknin@devalore.com', description: 'write mailRecipients.', name: 'MailRecipients')
        choice(name: 'BuildType', choices:"Debug\nRelease", description: "Select build type")       
    }
    
    stages {
        stage('Static Code Analysis') {   
            steps { 
                script {
                    runCommand('cppcheck --enable=all --inconclusive --xml-version=2 --force --library=windows,posix,gnu libbar/ 2> Cppcheck_result.xml')

                    runCommand('cppcheck-htmlreport  --file=Cppcheck_result.xml --title=LibreOffice --report-dir=cppcheck_reports --source-dir=')                   
                    publishHTML (target: [
                        allowMissing: false,    
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'cppcheck_reports',
                        reportFiles: 'index.html',
                        reportName: "Cppcheck Report"
                        ])                    
                }
            }
        }

        stage('Build') {
            steps {    

                // debug info
                // printJobParameter()

                script {

                    // run cmake generate and build
                    cmakeBuild buildDir: 'build', buildType: params.BuildType , installation: 'InSearchPath', steps: [[args: '--target install', withCmake: true]]

                    runCommand('./build/test/testfoo/testfoo --gtest_output="xml:testresults.xml"')
                    junit 'testresults.xml'
                }        
            }                       
            
            post { 
                failure { 
                    step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject: "${currentBuild.currentResult}: ${env.JOB_NAME} - build ${currentBuild.number} = 'FAILURE'", recipients: params.MailRecipients, sendToIndividuals: true])
                }
            }
        }
        
        // stage('Report') 
        // {    
        //     steps 
        //     {
        //         script{
        //             if(params.Report){

        //                 runCommand('cppcheck-htmlreport  --file=Cppcheck_result.xml --title=LibreOffice --report-dir=cppcheck_reports --source-dir=')
                        
        //                 // try{
        //                 //     step([$class: 'RobotPublisher',
        //                 //         disableArchiveOutput: false,
        //                 //         logFileName: 'log.html',
        //                 //         otherFiles: '',
        //                 //         outputFileName: 'output.xml',
        //                 //         outputPath: '.',
        //                 //         passThreshold: 100,
        //                 //         reportFileName: 'report.html',
        //                 //         unstableThreshold: 0]);
        //                 // }catch(exc){
        //                 //     echo 'Something failed in Robot publisher'
        //                 // }

        //                 /* ...HTML report... */

        //                 // Archive the built artifacts
        //                 archive (includes: 'pkg/*.gem')
        //                 //archiveArtifacts "xml"
                        

        //                 /*-----publish html-----*/
        //                 // snippet generator doesn't include "target:"
        //                 publishHTML (target: [
        //                     allowMissing: false,    
        //                     alwaysLinkToLastBuild: false,
        //                     keepAll: true,
        //                     reportDir: 'cppcheck_reports',
        //                     reportFiles: 'index.html',
        //                     reportName: "Cppcheck Report"
        //                     ])

        //             }else echo "Report step skipped"   
        //         }
        //     }
        //     post { 
        //         failure { 
        //             step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject: "${currentBuild.currentResult}: ${env.JOB_NAME} - Report ${currentBuild.number} = 'FAILURE'", recipients: params.MailRecipients, sendToIndividuals: true])
        //         }
        //     }
        // }
        
        // stage('Functional Testing') {

        // }

        stage('Send Email') {
            steps {
                script {

                    emailext (body: '''${SCRIPT, template="buildlog.template"}''',
                    mimeType: 'text/html',
                    subject: "[Jenkins] Buildlog",
                    to: params.MailRecipients,
                    replyTo: params.MailRecipients,
                    recipientProviders: [[$class: 'CulpritsRecipientProvider']])
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
robotrun = ${ROBOTRUN} ${params.RobotTestDirectory}

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
