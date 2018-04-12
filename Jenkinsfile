
pipeline { 
    agent any 

     parameters
    {
        string(defaultValue: 'git@github.com:DavidVaknin/Build_part.git', description: 'The url of the git repository the contains the projects CMakeLists.txt file in the root directory.  ', name: 'RepositoryUrl')
        string(defaultValue: '/home/matt/Documents/DuduV/Build_part/Build_part', description: 'A workspace directory on the master and build-slave into which the code is checked-out and which is used for the build.  ', name: 'CheckoutDirectory')
        string(defaultValue: '', description: 'The tag for the build-slave on which the project is build.', name: 'BuildSlaveTag')
        string(defaultValue: 'master', description: 'Relevant branch to test.', name: 'Branch')
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
                
                        // run cmake generate and build
                        cmakeBuild buildDir: 'build', installation: 'InSearchPath', steps: [[args: '--target install', withCmake: true]]

                        echo '----- CMake project was build successfully -----'
                    }
                }
            }
             
                post { 
                    failure { 
                     step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject: "Pipeline Success: ${currentBuild.fullDisplayName}", recipients: 'david.vaknin@devalore.com', sendToIndividuals: true])
                    }
                }

        }
         stage('Report') 
        {
             steps 
            {
                sh 'cd test/testfoo/'
                sh 'cppcheck-htmlreport  --file=Cppcheck_result.xml --title=LibreOffice --report-dir=Cppcheck_reports --source-dir='
                
                //sh 'ls -l'
                //sh 'chmod -R 777 Cppcheck_reports/index.html'
                //sh 'Cppcheck_reports/index.html'
                //sh './testfoo --gtest_output=xml'
                sh 'ls -l'
            /* ...unchanged... */

            // Archive the built artifacts
            archive (includes: 'pkg/*.gem')

            // publish html
            // snippet generator doesn't include "target:"
            publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: '',
                reportFiles: 'index.html',
                reportName: "RCov Report"
                ])

            }
            post { 
                    failure { 
                     step([$class: 'Mailer', notifyEveryUnstableBuild: true,subject:"pipeline SUCCESS", recipients: 'david.vaknin@devalore.com', sendToIndividuals: true])
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

