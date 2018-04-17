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

                        //sh '''if [ -d /usr/share]
                         //   then
                         //        echo "Directoty exist"
                         //   else
                         //       mkdir /usr/share
                         //   fi
                         //   '''

                        // checkout sources
                        checkout([$class: 'GitSCM',
                            userRemoteConfigs: [[url: params.RepositoryUrl]],
                            branches: [[name: params.Branch]],
                            extensions: [[$class: 'CleanBeforeCheckout']]]
                        )
                   
                        
                        // run cmake generate and buildmkdir Release
                        cmakeBuild buildDir: 'build', buildType: params.BuildType , installation: 'InSearchPath', steps: [[args: '--target install', withCmake: true]]    
                    
                        echo '----- CMake project was build successfully -----'
                                
                            
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
                
                
                //sh './testfoo --gtest_output=xml'
                sh 'ls test/testfoo'

                 /* ...HTML report... */

                 // Archive the built artifacts
                archive (includes: 'pkg/*.gem')
                //archiveArtifacts "xml"
                

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
                    //emailext (attachLog: true, body: '''${SCRIPT, template="buildlog.template"}''', compressLog: true, mimeType: 'html', subject: 'Build logs', to: params.MailRecipients, replyTo: params.MailRecipients)

            }
            post { 
                failure { 
                    step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject:"pipeline SUCCESS", recipients: params.MailRecipients, sendToIndividuals: true])
                always {
                    junit 'build/reports/**/*.xml'
                    }
                }
            }
        }
        stage('Send email') {
            steps{
                emailext body: '''${SCRIPT, template="buildlog.template"}''',
                mimeType: 'text/html',
                subject: "[Jenkins] Buildlog",
                to: params.MailRecipients,
                replyTo: params.MailRecipients,
                recipientProviders: [[$class: 'CulpritsRecipientProvider']]
            }  
            post {
                always {
                 emailext (
                    to: params.MailRecipients,
                    subject: "${currentBuild.currentResult}: ${env.JOB_NAME} - build ${currentBuild.number}",
                    body: '${FILE, path="$WORKSPACE/result.xml"}')
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
