pipeline
 { 
    agent any 
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
        stage('Test')
        {
            steps 
            {
               // sh 'make check'
               // junit 'reports/**/*.xml' 
            }
        }
        stage('Deploy') 
        {
            steps 
            {
               // sh 'make publish'
            }
        }   
    }
}
def printJobParameter()
{
    def introduction = """
----- Build CMake project -----
git@github.com:DavidVaknin/Build_part.git = ${params.RepositoryUrl}
/home/matt/Documents/DuduV/Build_part/Build_part = ${params.CheckoutDirectory}
BuildSlaveONE1 = ${params.BuildSlaveTag}
AdditionalGenerateArguments = ${params.AdditionalGenerateArguments}
AdditionalBuildArguments = ${params.AdditionalBuildArguments}
-------------------------------
"""
    
    echo introduction
}