pipeline {
    agent any

    stages {
        stage('Upload Files to Nexus') {
            steps {
                script {
                    def uploadAllFilesToNexus = { String baseDir, String repoUrl, String repoPath, String credentialsId ->
                        def currentDate = new Date().format('dd-MM-yyyy')

                        dir(baseDir) {
                            def files = sh(returnStdout: true, script: "find . -type f").trim().split('\n')

                            files.each { filePath ->
                                def relativePath = filePath.replaceFirst("^\\./", "")
                                def nexusUrl = "${repoUrl}/${repoPath}/${currentDate}/feature_test/${relativePath}"

                                echo "Uploading: ${filePath} to ${nexusUrl}"

                                withCredentials([usernamePassword(credentialsId: credentialsId, passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                                    sh """
                                        curl -v -u "$USERNAME:$PASSWORD" \
                                             --upload-file "${filePath}" \
                                             "${nexusUrl}"
                                    """
                                }
                            }
                        }
                    }

                    uploadAllFilesToNexus(
                        'first_pipeline',                          // folder inside workspace
                        'http://localhost:8081/repository',        // Nexus repo base URL
                        'iranRepo',                                // Nexus repo name
                        'nexus3'                                   // Jenkins credentials ID
                    )
                }
            }
        }
    }
}
