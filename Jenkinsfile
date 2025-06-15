pipeline {
    agent any

    stages {
        stage('Create Dummy JSON Files') {
            steps {
                script {
                    // Create folder and JSON files under Jenkins workspace
                    sh '''
                        mkdir -p first_pipeline
                        echo '{}' > first_pipeline/a.json
                        echo '{}' > first_pipeline/b.json
                        echo '{}' > first_pipeline/c.json
                    '''
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    // Define the upload function
                    def uploadFilesToNexus = { String localDir, String repoUrl, String repoName, String credsId ->
                        // Get today's date in format dd-MM-yyyy
                        def today = new Date().format('dd-MM-yyyy')

                        // Move into the local directory with the files
                        dir(localDir) {
                            // List all files in this directory (non-recursive)
                            def files = sh(returnStdout: true, script: 'find . -maxdepth 1 -type f').trim().split('\n')

                            // Loop through each file and upload
                            files.each { filePath ->
                                // Remove './' from filename (e.g., './a.json' → 'a.json')
                                def fileName = filePath.replaceFirst("^\\./", "")
                                
                                // Upload URL: http://localhost:8081/repository/iranRepo/<date>/<filename>
                                def uploadUrl = "${repoUrl}/${repoName}/${today}/${fileName}"

                                // Print what’s being uploaded
                                echo "Uploading ${fileName} to ${uploadUrl}"

                                // Use Jenkins credentials and upload via curl
                                withCredentials([usernamePassword(credentialsId: credsId, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                                    sh """#!/bin/bash
                                        curl -s -o /dev/null -w "%{http_code}" \
                                             -u "$USERNAME:$PASSWORD" \
                                             --upload-file '${fileName}' \
                                             '${uploadUrl}'
                                    """
                                }
                            }
                        }
                    }

                    // Call the function with your parameters
                    uploadFilesToNexus(
                        'first_pipeline',                          // Folder in Jenkins workspace
                        'http://localhost:8081/repository',        // Nexus base URL
                        'iranRepo',                                // Nexus repo name
                        'nexus3'                                   // Jenkins credentials ID
                    )
                }
            }
        }
    }
}
