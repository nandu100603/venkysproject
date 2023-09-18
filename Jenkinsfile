pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile and Test') {
            matrix {
                axes {
                    axis {
                        name 'LANGUAGE'
                        values 'cpp', 'java', 'python'
                    }
                }
                stages {
                    stage('Compile ${LANGUAGE}') {
                        when {
                            changeset "${LANGUAGE}/**"
                        }
                        steps {
                            script {
                                def languageDir = "${LANGUAGE}/"
                                def changedFiles = getChangedFiles(languageDir)

                                if (changedFiles) {
                                    echo "${LANGUAGE} code changes detected in the following files:"
                                    echo changedFiles.join('\n')

                                    // Compile and test steps for the specific language
                                    def compilationStatus = [:]

                                    if (LANGUAGE == 'cpp') {
                                        for (file in changedFiles) {
                                            def outputName = file.replaceAll('\\.cpp$', '.out')
                                            def compileCmd = "g++ ${file} -o ${outputName}"
                                            def result = bat(script: compileCmd, returnStatus: true)
                                            compilationStatus[file] = result
                                        }
                                    }

                                    if (LANGUAGE == 'java') {
                                        for (file in changedFiles) {
                                            def className = file.replaceAll('\\.java$', '')
                                            def compileCmd = "javac ${file}"
                                            def result = bat(script: compileCmd, returnStatus: true)
                                            compilationStatus[className] = result
                                        }
                                    }

                                    if (LANGUAGE == 'python') {
                                        for (file in changedFiles) {
                                            def result = bat(script: "python ${file}", returnStatus: true)
                                            compilationStatus[file] = result
                                        }
                                    }

                                    // Print compilation status
                                    echo "Compilation Status for ${LANGUAGE}:"
                                    compilationStatus.each { fileName, status ->
                                        if (status == 0) {
                                            echo "${fileName}: Compiled successfully"
                                        } else {
                                            echo "${fileName}: Compilation failed"
                                            currentBuild.result = 'FAILURE' // Mark the build as failed
                                        }
                                    }
                                } else {
                                    echo "${LANGUAGE} code changes detected, but no specific files found."
                                }
                            }
                        }
                    }
                }
            }
        }
    }
post {
        success {
            script {
                def authorEmail = bat(script: 'git log -1 --pretty=format:%ae', returnStdout: true).trim()
echo "Author's email address: ${authorEmail}"

                if (authorEmail) {

                    emailext body: '''
                        Hi ${GIT_AUTHOR_NAME},
                        Your Jenkins build ${BUILD_ID} for project ${JOB_NAME} has been completed successfully.
                        Thanks, The Jenkins team
                    ''',
                    subject: 'Jenkins build ${BUILD_ID} for project ${JOB_NAME} has been completed successfully',
                    to: authorEmail,
                    from: 'nandu100603@gmail.com'
                } else {
                    echo "No email address found for the commit author."
                }
            }
        }
    failure {
        script {
            def authorEmail = bat(script: 'git log -1 --pretty=format:%ae', returnStdout: true).trim()
            echo "Author's email address: ${authorEmail}"

            if (authorEmail) {
                def failureMessage = '''
                    Hi ${GIT_AUTHOR_NAME},
                    Your Jenkins build ${BUILD_ID} for project ${JOB_NAME} has failed.
                    Please check the Jenkins console for more details.
                    Thanks,
                    The Jenkins team
                '''

                emailext body: failureMessage,
                        subject: 'Jenkins build ${BUILD_ID} for project ${JOB_NAME} has failed',
                        to: authorEmail,
                        from: 'nandu100603@gmail.com'
            } else {
                echo "No email address found for the commit author."
            }
        }
    }
    }  
}


@NonCPS
List<String> getChangedFiles(String directory) {
    def changedFiles = []
    for (changeLogSet in currentBuild.changeSets) {
        for (entry in changeLogSet.getItems()) {
            for (path in entry.affectedPaths) {
                if (path.startsWith(directory)) {
                    changedFiles.add(path)
                }
            }
        }
    }
    return changedFiles
}
