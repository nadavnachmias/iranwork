pipeline {
    agent any

    stages {
        stage('Create Folder Tree and Test Files') {
            steps {
                script {
                    // Create a nested folder tree with various files
                    sh '''
                        mkdir -p first_pipeline/level1/level2
                        echo '{}' > first_pipeline/level1/level2/a.json
                        echo 'log content' > first_pipeline/level1/level2/b.log
                        echo 'some text' > first_pipeline/level1/level2/c.txt
                        echo 'data' > first_pipeline/level1/level2/data.csv
                    '''
                }
            }
        }

        stage('Upload All Files to Nexus (Flat, with -k)') {
            steps {
                script {
                    def uploadAllFilesFlat = { String localDir, String repoUrl, String repoName, String credsId ->
                        def today = new Date().format('dd-MM-yyyy')

                        dir(localDir) {
                            // Find all files recursively
                            def files = sh(returnStdout: true, script: "find . -type f").trim().split('\n')

                            files.each { filePath ->
                                def fileName = filePath.tokenize('/').last()  // just the filename
                                def uploadUrl = "${repoUrl}/${repoName}/${today}/${fileName}"

                                echo "Uploading ${filePath} → ${uploadUrl}"

                                withCredentials([usernamePassword(credentialsId: credsId, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                                    sh """
                                        curl -k -u "$USERNAME:$PASSWORD" \
                                             --upload-file '${filePath}' \
                                             '${uploadUrl}'
                                    """
                                }
                            }
                        }
                    }

                    // Call the uploader function
                    uploadAllFilesFlat(
                        'first_pipeline',                          // Jenkins workspace folder
                        'http://localhost:8081/repository',        // Nexus base URL
                        'iranRepo',                                // Nexus repo name
                        'nexus3'                                   // Jenkins credentials ID
                    )
                }
            }
        }
    }
}
