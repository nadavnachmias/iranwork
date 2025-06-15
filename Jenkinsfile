pipeline {
    agent any

    stages {
        stage('Clean Old Nexus Folders') {
            steps {
                script {
                    def cleanupOldNexusFolders = { String repoUrl, String repoPath, String credentialsId ->
                        def threshold = new Date() - 7  // today minus 7 days

                        withCredentials([usernamePassword(credentialsId: credentialsId, passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            // List folders in the repo path and extract those matching dd-MM-yyyy format
                            def listCmd = """
                            curl -s -u "$USERNAME:$PASSWORD" "${repoUrl}/${repoPath}/" | \
                            grep -oP '(?<=href=")[0-9]{2}-[0-9]{2}-[0-9]{4}/(?=")' | sed 's#/##'
                            """

                            def folderListRaw = sh(returnStdout: true, script: listCmd).trim()

                            if (!folderListRaw) {
                                echo "No date-based folders found in Nexus path: ${repoPath}"
                                return
                            }

                            def folderList = folderListRaw.split("\\n")

                            folderList.each { folder ->
                                try {
                                    def folderDate = Date.parse('dd-MM-yyyy', folder)

                                    if (folderDate.before(threshold)) {
                                        def deleteUrl = "${repoUrl}/${repoPath}/${folder}/"
                                        echo "🗑️ Deleting folder: ${folder} → ${deleteUrl}"

                                        sh """
                                            curl -k -u "$USERNAME:$PASSWORD" -X DELETE "${deleteUrl}"
                                        """
                                    } else {
                                        echo "✅ Keeping folder: ${folder}"
                                    }
                                } catch (Exception e) {
                                    echo "⚠️ Skipping non-date folder: ${folder}"
                                }
                            }
                        }
                    }

                    // Call cleanup function with same repo + credentials
                    cleanupOldNexusFolders(
                        'http://localhost:8081/repository',  // Nexus base URL
                        'iranRepo',                          // Nexus repo path
                        'nexus3'                             // Jenkins credentials ID
                    )
                }
            }
        }
    }
}
