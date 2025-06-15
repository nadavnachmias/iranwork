pipeline {
    agent any

    stages {
        stage('Delete Old Date-Based Folders') {
            steps {
                script {
                    def cleanupOldNexusFolders = { String repoUrl, String repoPath, String credentialsId ->
                        def threshold = new Date() - 7

                        withCredentials([usernamePassword(credentialsId: credentialsId, passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            // List folders using curl and extract dd-MM-yyyy formatted folder names
                            def listCmd = """
                                curl -k -s -u "$USERNAME:$PASSWORD" "${repoUrl}/${repoPath}/" | \
                                grep -oE 'href="[0-9]{2}-[0-9]{2}-[0-9]{4}/"' | \
                                cut -d '"' -f2 | sed 's#/##'
                            """

                            def result = sh(returnStdout: true, script: listCmd).trim()
                            if (!result) {
                                echo "⚠️ No folders found at ${repoUrl}/${repoPath}/"
                                return
                            }

                            def folders = result.split("\\n")

                            folders.each { folder ->
                                try {
                                    def folderDate = Date.parse('dd-MM-yyyy', folder)
                                    if (folderDate.before(threshold)) {
                                        def deleteUrl = "${repoUrl}/${repoPath}/${folder}/"
                                        echo "🗑️ Deleting folder: ${deleteUrl}"

                                        sh """
                                            curl -k -u "$USERNAME:$PASSWORD" -X DELETE "${deleteUrl}"
                                        """
                                    } else {
                                        echo "✅ Keeping folder: ${folder}"
                                    }
                                } catch (Exception e) {
                                    echo "⚠️ Skipping invalid folder: ${folder}"
                                }
                            }
                        }
                    }

                    // Call the function
                    cleanupOldNexusFolders(
                        'http://localhost:8081/repository',  // base URL
                        'iranRepo',                          // repo name
                        'nexus3'                             // Jenkins credentials ID
                    )
                }
            }
        }
    }
}
