pipeline {
    agent any

    stages {
        stage('Create Folder Tree with Files') {
            steps {
                script {
                    // Create a nested file tree with mixed files
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

        stage('Upload Files to Nexus (10-Day Range)') {
            steps {
                script {
                    def uploadToDateFolders = { String localDir, String repoUrl, String repoName, String credsId, String startDateStr, int numDays ->
                        def sdf = new java.text.SimpleDateFormat("dd-MM-yyyy")
                        def startDate = sdf.parse(startDateStr)

                        dir(localDir) {
                            def files = sh(returnStdout: true, script: "find . -type f").trim().split('\n')

                            (0..<numDays).each { offset ->
                                def date = startDate + offset
                                def dateStr = sdf.format(date)

                                echo "📂 Uploading into Nexus folder: ${dateStr}"

                                files.each { filePath ->
                                    def fileName = filePath.tokenize('/').last()
                                    def uploadUrl = "${repoUrl}/${repoName}/${dateStr}/${fileName}"

                                    echo "→ Uploading ${filePath} → ${uploadUrl}"

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
                    }

                    // Call: upload from 05-06-2025 to 15-06-2025
                    uploadToDateFolders(
                        'first_pipeline',
                        'http://localhost:8081/repository',
                        'iranRepo',
                        'nexus3',
                        '05-06-2025',  // start date
                        11             // number of days (inclusive range)
                    )
                }
            }
        }
    }
}
