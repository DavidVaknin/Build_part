
pipeline { 
    agent any 

    parameters
    {
        text defaultValue: 'git@github.com:DavidVaknin/Build_part.git', description: '', name: 'RepositoryUrl'
    }   
    stages  
     {
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
                            branches: [[name: 'master']],
                            extensions: [[$class: 'CleanBeforeCheckout']]]
                        )
                
                        // run cmake generate and build
                        cmakeBuild buildDir: 'build', installation: 'InSearchPath', steps: [[args: '--target install', withCmake: true]]

                        echo '----- CMake project was build successfully -----'
                    }
                }
            }
                post { 
                    always { 
                     step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'david.vaknin@devalore.com', sendToIndividuals: true])
                    }
                }
        }
         

        //stage('CodeAnlaysis')
        //{
           // steps 
            //{
               // sh 'make check'
               // junit 'reports/**/*.xml' 
           // }
       // }
        stage('Test')
        {
            steps 
            {
               sh  'cppcheck --enable=all --inconclusive --xml-version=2 --force --library=windows,posix,gnu libbar/ 2> result.xml && cppcheck --enable=all --inconclusive --xml-version=2 --force --library=windows,posix,gnu libfoo/ 2> result.xml && cppcheck --enable=all --inconclusive --xml-version=2 --force --library=windows,posix,gnu main/ 2> result.xml && cppcheck --enable=all --inconclusive --xml-version=2 --force --library=windows,posix,gnu test/ 2> test/result.xml'
               // junit 'reports/**/*.xml' 
            }
        }
       // stage('Deploy') 
        //{
           // steps 
           // {
               // sh 'make publish'
          //  }
       // }
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

