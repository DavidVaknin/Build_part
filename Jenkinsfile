
pipeline { 
    agent any 

    parameters {
       string defaultValue: 'git@github.com:DavidVaknin/Build_part.git', description: '', name: 'RepositoryUrl', trim: false, string defaultValue: 'home/matt/Documents/DuduV/Build_part/Build_part', description: '', name: 'CheckoutDirectory', trim: false, string defaultValue: 'TAG1', description: '', name: 'BuildSlaveTag', trim: false
    }

    stages  
     {
        stage('Build')  
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
        }
        //stage('Test')
        //{
           // steps 
            //{
               // sh 'make check'
               // junit 'reports/**/*.xml' 
           // }
       // }
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

