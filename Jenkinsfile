pipeline {
    agent any

    triggers {
        githubPullRequest {
            
            cron('H/5 * * * *') // ejecuta cada 5 minutos
            triggerPhrase('Jenkins test this please') // opcional: frase para iniciar build
            onlyTriggerPhrase(false)
            useGitHubHooks(true)
        }
    }

    tools {
        'hudson.plugins.cmake.CmakeTool' 'cmake' // Name of the CMake installation configured in Jenkins
    }

    stages {
        
        stage('Checkout') {
            steps {
                // Checkout code from version control including submodules using the scm variable
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: "${GIT_BRANCH}"]],
                    extensions: [[$class: 'SubmoduleOption', recursiveSubmodules: true]], 
                    userRemoteConfigs: scm.userRemoteConfigs
                ])
            }
        }
        

        stage('Build') {
            steps {
                script {
                    // Use the CMake installation configured in Jenkins
                    withEnv(["PATH+CM=${tool name: 'cmake'}/bin"]) {
                        // Create a build directory
                        sh 'rm -rf build'
                        sh 'mkdir -p build'
                        dir('build') {
                            // Run CMake to configure the build system
                            sh 'cmake ..'
                            // Build the project
                            sh 'cmake --build .'
                        }
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    dir('build') {
                        // Run tests
                        sh 'ctest --verbose'
                    }
                }
            }
        }
    }

    post {
        always {
            // Archive the build artifacts
            archiveArtifacts artifacts: 'build/**/*', allowEmptyArchive: true
            
            // Clean up build directory after the build
            deleteDir()
        }
    }
}
